<script lang="ts">

import { defineComponent } from 'vue';


export default defineComponent({
    props : {
        new_grab : Boolean,
        image_data : Uint8ClampedArray,
        prices : Array<string>
    },
    data() {
        return {
            grabbing : true,
            isDown : false,
            overlay_canvas: null as (HTMLCanvasElement | null),
            img_img: null as (HTMLImageElement | null),
            select_start_x : 0,
            select_start_y : 0,
            select_width : 0,
            select_height : 0,
            previous_url : null as (string | null)
        }
    },
    mounted() {
        this.overlay_canvas = document.getElementById('overlay_canvas') as HTMLCanvasElement
        this.img_img = document.getElementById('img_img') as HTMLImageElement
        this.img_img.onload = () => {
            if (this.img_img != null && this.overlay_canvas != null) {
                this.overlay_canvas.width = this.img_img.width
                this.overlay_canvas.height = this.img_img.height
            }
        }
    },
    watch : {
        image_data: {
            handler(new_data) {
                this.grabbing = true
                this.$emit('pricerEvent', '_new_grab')
                console.log(new_data)
                console.log('new data')
                if (this.img_img == null) {
                    return
                }
                this.img_img.src = ''
                if (this.previous_url != null) {
                    URL.revokeObjectURL(this.previous_url)
                    this.previous_url = null
                }
                const img = new Blob([new_data], {type: 'image/png'})
                this.previous_url = URL.createObjectURL(img)
                this.img_img.src = this.previous_url
                if (this.overlay_canvas != null) {
                    this.overlay_canvas.width = this.img_img.width
                    this.overlay_canvas.height = this.img_img.height
                }
            }
        }
    },
    methods: {
        selectdown(e : MouseEvent) {
            this.select_start_x = e.offsetX
            this.select_start_y = e.offsetY
            this.isDown = true
        },
        touchdown(e : TouchEvent) {
            if (this.overlay_canvas == null)
            {
                return
            }
            const rect = this.overlay_canvas.getBoundingClientRect()
            if (e.touches.length > 1)
            {
                this.select_start_x = e.touches[0].clientX - rect.left;
                this.select_start_y = e.touches[0].clientY - rect.top;
                const tnum = e.touches.length - 1;
                this.drawmove(e.touches[tnum].clientX - rect.left, e.touches[tnum].clientY - rect.top)
                e.preventDefault()
            }
        },
        selectup() {
            this.isDown = false
        },
        selectout() {
            this.isDown = false
        },
        selectmove(e : MouseEvent) {
            if (!this.isDown){
                return
            }
            const mouseX = e.offsetX
            const mouseY = e.offsetY
            this.drawmove(mouseX, mouseY)
        },
        drawmove(mouseX : number, mouseY : number) {
            if (this.overlay_canvas == null)
            {
                return
            }
            const context = this.overlay_canvas.getContext('2d')
            if (context == null) {
                return
            }
            context.clearRect(0,0,this.overlay_canvas.width, this.overlay_canvas.height)
            this.select_width = mouseX - this.select_start_x;
            this.select_height = mouseY - this.select_start_y;
            context.strokeStyle = 'rgba(0, 150, 150, 1)'
            context.fillStyle = 'rgba(0, 150, 150, 0.3)'
            context.strokeRect(this.select_start_x, this.select_start_y, this.select_width, this.select_height)
            context.fillRect(this.select_start_x, this.select_start_y, this.select_width, this.select_height)
            this.grabbing = true
        },
        touchmove(e : TouchEvent) {
            if (this.overlay_canvas == null) {
                return
            }
            const rect = this.overlay_canvas.getBoundingClientRect()
            if (e.touches.length > 1)
            {
                this.select_start_x = e.touches[0].clientX - rect.left;
                this.select_start_y = e.touches[0].clientY - rect.top;
                const tnum = e.touches.length - 1;
                this.drawmove(e.touches[tnum].clientX - rect.left, e.touches[tnum].clientY - rect.top)
                e.preventDefault()
            }
            //this.drawmove(mouseX, mouseY)
        },
        triggerFinish() {
            const data = (this.select_start_x + ',' + this.select_start_y + ',' +
                          this.select_width + ',' + this.select_height)
            this.$emit('pricerEvent', 'curio_select', data)
            this.$emit('pricerEvent', '_old_grab')
            this.grabbing = false
        },
        triggerCurioCapture() {
            this.$emit('pricerEvent', 'capture_curio')
        }
    }
})

</script>

<template>
    <img width="120" src="heist_icons/WaxSeal.png" @click="triggerFinish">
    <img @click="event => triggerCurioCapture()" :src="'heist_icons/Curio.png'" />
    <div v-for="prc in prices" :key="prc">
        <span>{{ prc }}</span>
    </div>
    <div :hidden="! grabbing" class="wrapper">
        <img id="img_img">
        <canvas @mousedown="selectdown" @touchstart="touchdown"
                @mouseup="selectup"
                @mouseout="selectout"
                @mousemove="selectmove" @touchmove="touchmove"
                id="overlay_canvas"></canvas>
    </div>
</template>

<style lang="css">
.wrapper{
    font-weight: bold;
}
.wrapper > img {
    position: absolute;
}
.wrapper > canvas {
    position: absolute;
}
</style>