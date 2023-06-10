<template>
    <q-page>
        <div>
            <img src="heist_icons/SmallContract.png" @click="toggleIP" width="50">
            <input v-model="ip_addrs" :hidden="!input_ip" style="vertical-align: top;">
            <span style="vertical-align: top;" :hidden="input_ip">{{ ip_addrs }}</span>
        </div>
        <run-tracker :hidden="new_grab" :currentRun="currentRun" @trackerEvent="trackerCatcher"></run-tracker>
        <select-price @pricerEvent="trackerCatcher" :image_data="image_data" :prices="prices"></select-price>
    </q-page>
</template>

<script lang="ts">
import RunTracker from 'components/RunTracker.vue';
import SelectPrice from 'components/SelectPrice.vue';
import { RunStatus, HeistressRequest, PriceInfo } from 'src/resources/remoteinfo';
import { defineComponent } from 'vue';

export default defineComponent({
    name: 'TestPage',
    data() {
        return {
            currentRun : new RunStatus(),
            new_grab : false,
            image_data : new Uint8ClampedArray(),
            input_ip: true,
            ip_addrs: '',
            command_sock: null as null | WebSocket,
            update_sock: null as null | WebSocket,
            image_sock: null as null | WebSocket,
            price_sock: null as null | WebSocket,
            prices : [] as Array<string>
        }
    },
    components: { RunTracker, SelectPrice },
    setup() {
        return {}
    },
    methods : {
        toggleIP() {
            if (this.input_ip) {
                this.input_ip = false
                this.command_sock = new WebSocket('wss://' + this.ip_addrs +':10635')
                this.update_sock = new WebSocket('wss://' + this.ip_addrs + ':10636')
                this.image_sock = new WebSocket('wss://' + this.ip_addrs + ':10637')
                this.price_sock = new WebSocket('wss://' + this.ip_addrs + ':10638')
                this.update_sock.addEventListener('message', (message) => {
                    const data = JSON.parse(message.data.toString())
                    this.currentRun.dejson(data)
                })
                this.image_sock.addEventListener('message', (message) => {
                    const data = message.data as string
                    const adata = data.split(',').map(Number)
                    const cdata = new Uint8ClampedArray(adata)
                    this.image_data = cdata
                })
                this.price_sock.addEventListener('message', (message) => {
                    const data = JSON.parse(message.data.toString())
                    const pinfo = new PriceInfo().dejson(data)
                    this.prices = pinfo.prices
                })
            } else {
                this.input_ip = true
            }
        },
        trackerCatcher(arg1 : string, arg2 : string) {
            if (arg1 == '_new_grab') {
                this.new_grab = true
            }
            if (arg1 == '_old_grab') {
                this.new_grab = false
            }
            if (arg1 == 'toggle_blueprint' && this.command_sock != null)
            {
                const request = new HeistressRequest('toggle_blueprint')
                this.command_sock.send(JSON.stringify(request.rejson()))
            }
            if (arg1 == 'toggle_job' && this.command_sock != null) {
                const request = new HeistressRequest('toggle_job', arg2)
                this.command_sock.send(JSON.stringify(request.rejson()))
            }
            if (arg1 == 'toggle_rogue' && this.command_sock != null) {
                const request = new HeistressRequest('toggle_rogue', arg2)
                this.command_sock.send(JSON.stringify(request.rejson()))
            }
            if (arg1 == 'add_reward' && this.command_sock != null) {
                const request = new HeistressRequest('add_reward', arg2)
                this.command_sock.send(JSON.stringify(request.rejson()))
            }
            if (arg1 == 'remove_reward' && this.command_sock != null) {
                const request = new HeistressRequest('remove_reward', arg2)
                this.command_sock.send(JSON.stringify(request.rejson()))
            }
            if (arg1 == 'capture_curio' && this.command_sock != null) {
                const request = new HeistressRequest('capture_curio')
                this.command_sock.send(JSON.stringify(request.rejson()))
            }
            if (arg1 == 'curio_select' && this.command_sock != null) {
                const request = new HeistressRequest('curio_select', arg2)
                this.command_sock.send(JSON.stringify(request.rejson()))
            }
        }
    }
});
</script>