<script lang="ts">

import { defineComponent } from 'vue';
import Icons from '../resources/icons.json'
import { RunStatus } from '../resources/remoteinfo'

function getTimeString(start : Date, now : Date) : string {
    const dmilli = now.getMilliseconds() - start.getMilliseconds();
    const dsecs = now.getSeconds() - start.getSeconds();
    const dmin = now.getMinutes() - start.getMinutes();
    const dhour = now.getHours() - start.getHours();
    const ddays = now.getDay() - start.getDay();
    var delta = dmilli + 1000 * (dsecs + 60 * (dmin + 60 * (dhour + 24 * ddays)));
    if (delta < 0)
    {
        delta = 0;
    }
    const mins = Math.floor(delta / 1000 / 60);
    delta = delta - mins * 1000 * 60;
    const secs = Math.floor(delta / 1000);
    delta = delta - secs * 1000;
    const subsecs = Math.floor(delta/100);
    return ('0' + mins).slice(-2) + ':' + ('0' + secs).slice(-2) + '.' + subsecs
}

export default defineComponent({
    props : {
        currentRun : RunStatus
    },
    data() {
        return {
            undo_button : false
        }
    },
    methods: {
        toggleBlueprint() {
            this.$emit('trackerEvent', 'toggle_blueprint')
        },
        toggleJob(job : string) {
            this.$emit('trackerEvent', 'toggle_job', job)
        },
        toggleRogue(rogue : string) {
            this.$emit('trackerEvent', 'toggle_rogue', rogue)
        },
        addReward(reward : string) {
            if (this.undo_button) {
                this.$emit('trackerEvent', 'remove_reward', reward)
                this.undo_button = false
            } else { 
                this.$emit('trackerEvent', 'add_reward', reward)
            }

        },
        removeReward(reward : string) {
            this.$emit('trackerEvent', 'remove_reward', reward)
            console.log('Remove reward' + reward)
        },
        getDmgIcon(icon_name : string) : string {
            const icon = icon_name as keyof typeof Icons.damage
            return Icons.damage[icon]
        },
        getJobIcon(icon_name : string) : string {
            const icon = icon_name as keyof typeof Icons.jobs
            return Icons.jobs[icon]
        },
        getRewardIcon(icon_name : string) : string {
            const icon = icon_name as keyof typeof Icons.rewards
            return Icons.rewards[icon]
        },
        getTime(start : Date, end : Date) {
            return getTimeString(start, end)
        },
        triggerCurioCapture() {
            this.$emit('trackerEvent', 'capture_curio')
        },
        undoButton() {
            this.undo_button = true
        }
    }
})
</script>

<template>
    <div class="tracker-div" id="tracker">
        <div class="flex-span" v-if="currentRun.name">
            <span v-if="currentRun.blueprint">{{ $t("common.blueprint") }}</span>
            <span v-if="!currentRun.blueprint">{{ $t("common.contract") }}</span> : {{ $t("instances." + currentRun.name) }}
            <div class="damage-types"></div>
            <div class="damage-types" v-for="dmg in currentRun.damage.currentDamage()" :key="dmg">
                <img :src="'heist_icons/' + getDmgIcon(dmg)" />
            </div>
        </div>
        <div class="flex-span" v-else>
            {{ $t("common.notrunning") }}
        </div>
        <div class="flex-span">
            <div class="main-tracking">{{ getTime(currentRun.timer_start, currentRun.timer_current) }}</div>
            <div class="sub-tracking">
                <span>{{ $t("common.grab") }}</span>
                <span>{{ getTime(currentRun.timer_start, currentRun.timer_grab) }}</span>
            </div>
            <div class="sub-tracking">
                <span>{{ $t("common.complete") }}</span>
                <span>{{ getTime(currentRun.timer_grab, currentRun.timer_current) }}</span>
            </div>
            <div class="sub-tracking">
                <span class="blueprint" @click="toggleBlueprint">{{ $t("common.blueprint") }}</span>
            </div>
        </div>
        <div class="rogue-span">
            <div class="rogue-text" v-for="rogue in currentRun.rogues.rogueMap()" :key="rogue[0]">
                <span @click="toggleRogue(rogue[0])" :style="{'color': rogue[1] ? '#a1212a' : 'black'}">{{ $t("rogues_short." + rogue[0]) }}</span>
            </div>
        </div>
        <div class="flex-span">
            <div class="job-span" v-for="job in currentRun.jobs.jobMap()" :key="job[0]">
                <img @click="toggleJob(job[0])" :style="{'filter': job[1] ? 'grayscale(0%)' : 'grayscale(100%)', 'opacity': currentRun.could_job.jobMap().get(job[0]) ? '1.0' : '0.3'}" :src="'heist_icons/' + getJobIcon(job[0])"/>
            </div>
            <div class="job-span"></div>
            <div class="job-span">
                <img @click="undoButton()" src="heist_icons/undo.png"/>
            </div>
        </div>
        <div id="rewards" class="reward-span">
            <div class="reward-button" v-for="reward in currentRun.reward_chests.rewardMap()" :key="reward[0]" :style="{'opacity': currentRun.can_rewards[reward[0] as reward_type] > 0 ? '1.0' : '0.3'}">
                <img @click.alt="removeReward(reward[0])" @click.exact="addReward(reward[0])"  :style="{'filter':  currentRun.has_rewards[reward[0] as reward_type] > 0 ? 'grayscale(0%)' : 'grayscale(100%)', 'opacity': currentRun.has_rewards[reward[0] as reward_type] > 0 ? '1.0' : '0.6'}" :src="'heist_icons/' + getRewardIcon(reward[0])" :alt="$t('rewards.' + reward[0])" />
                <div @click.alt="removeReward(reward[0])" @click.exact="addReward(reward[0])" class="reward-button-text">
                    <span>{{ reward[1] }}</span>
                </div>
            </div>
        </div>
    </div>
</template>

<style lang="css">
/*@import url('https://fonts.googleapis.com/css2?family=Caveat&display=swap');*/
.tracker-div {
    font-family: "Caveat", cursive;
    -webkit-user-select: none;
    user-select: none;
    -webkit-app-region: drag;
    background-size:cover;
    background-repeat: no-repeat;
    background-attachment: fixed;
}
.flex-span {
    display: flex;
    align-items: left;
    width: 100%;
    font-size: 3vw;
}
.damage-types {
    width: 5%;
}
.damage-types > img {
    width: 100%;
}
.main-tracking {
    width: 40%;
    font-size: 5vw;
}
.sub-tracking {
    width: 20%;
    font-size: 3vw;
}
.sub-tracking > span {
    display: block;
}
.blueprint {
    -webkit-app-region: no-drag;
}
.rogue-span {
    display: grid;
    grid-template-columns: auto auto auto auto auto;
}
.rogue-text {
    -webkit-app-region: no-drag;
    display: inline-grid;
    font-size: 3vw;
    width: 100%
}
.job-span {
    width: 7%;
}
.job-span > img {
    -webkit-app-region: no-drag;
    width: 100%;
}
.reward-span {
    display: grid;
    grid-template-columns: auto auto auto auto auto;
}
.reward-button {
    display: inline-grid;
    position: relative;
}
.reward-button > img {
    -webkit-app-region: no-drag;
    width:100%;
}
.reward-button-text {
    position: absolute;
    font-size: 3vw;
    bottom: 0.1vw;
    right: 0vw;
    text-shadow: -1px 0 white, 0 1px white, 1px 0 white, 0 -1px white;
    color: darkslategray;
}
</style>