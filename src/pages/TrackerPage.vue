<template>
    <q-page>
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
            command_sock: new WebSocket('ws://192.168.1.164:10635'),
            update_sock: new WebSocket('ws://192.168.1.164:10636'),
            image_sock: new WebSocket('ws://192.168.1.164:10637'),
            price_sock: new WebSocket('ws://192.168.1.164:10638'),
            prices : [] as Array<string>
        }
    },
    components: { RunTracker, SelectPrice },
    setup() {
        return {}
    },
    mounted() {
        this.update_sock.addEventListener('message', (message) => {
            //console.log(message)
            const data = JSON.parse(message.data)
            //const test = new RunStatus()
            //test.dejson(message.data)
            //console.log(test)
            this.currentRun.dejson(data)
        })
        this.image_sock.addEventListener('message', (message) => {
            const data = message.data as string
            console.log(data)
            const adata = data.split(',').map(Number)
            console.log(adata)
            const cdata = new Uint8ClampedArray(adata)
            console.log(cdata)
            this.image_data = cdata
        })
        this.price_sock.addEventListener('message', (message) => {
            const data = JSON.parse(message.data)
            const pinfo = new PriceInfo().dejson(data)
            this.prices = pinfo.prices
        })
    },
    methods : {
        trackerCatcher(arg1 : string, arg2 : string) {
            console.log('Event1' + arg1 + ' ' + arg2)
            if (arg1 == '_new_grab') {
                this.new_grab = true
            }
            if (arg1 == '_old_grab') {
                this.new_grab = false
            }
            if (arg1 == 'toggle_blueprint')
            {
                const request = new HeistressRequest('toggle_blueprint')
                this.command_sock.send(JSON.stringify(request.rejson()))
            }
            if (arg1 == 'toggle_job') {
                const request = new HeistressRequest('toggle_job', arg2)
                this.command_sock.send(JSON.stringify(request.rejson()))
            }
            if (arg1 == 'toggle_rogue') {
                const request = new HeistressRequest('toggle_rogue', arg2)
                this.command_sock.send(JSON.stringify(request.rejson()))
                console.log(this.command_sock)
            }
            if (arg1 == 'add_reward') {
                const request = new HeistressRequest('add_reward', arg2)
                this.command_sock.send(JSON.stringify(request.rejson()))
            }
            if (arg1 == 'remove_reward') {
                const request = new HeistressRequest('remove_reward', arg2)
                this.command_sock.send(JSON.stringify(request.rejson()))
            }
            if (arg1 == 'capture_curio') {
                const request = new HeistressRequest('capture_curio')
                this.command_sock.send(JSON.stringify(request.rejson()))
            }
            if (arg1 == 'curio_select') {
                const request = new HeistressRequest('curio_select', arg2)
                this.command_sock.send(JSON.stringify(request.rejson()))
            }
        }
    }
});
</script>