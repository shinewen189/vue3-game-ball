<template>
    <div class="imgBox" ref="imgbox">
        <slot></slot>
        <div class="mask"
             v-show="state.isShow"
             :style="{
                 left:state.maskX + 'px',
             top :state.maskY +'px'}">

        </div>
    </div>
    <transition name="fade">
        <div class="zoomBox"
             v-show="state.isShow"
             :style="{left :state.boxX +'px',top: state.boxY +'px',}">
            <img :src="state.imgSrc"
                 :style="{
                width:state.zoomImgWidth + 'px',
                height: state.zoomImgHeight + 'px',
                marginLeft :-state.bImgX + 'px',
                marginTop : -state.bImgY + 'px'
               }">
        </div>
    </transition>
</template>

<script setup>
    import {defineProps, onMounted, onUnmounted, reactive, ref} from 'vue'

    const maskWidth = 50, maskHeight = 50
    const imgbox = ref(null)
    const props = defineProps({mode: String})
    const state = reactive({
        zoomImgWidth: 0,
        zoomImgHeight: 0,
        isShow: false,
        boxX: 0,
        boxY: 0,
        maskX: 0,
        maskY: 0,
        bImgX: 0,
        bImgY: 0,
        imgSrc: ''
    })
    const zommIn = (ev) => {
        const img = ev.currentTarget.getElementsByTagName('img')[0]
        const {offsetWidth, offsetTop, offsetHeight} = imgbox.value
        state.zoomImgWidth = offsetWidth * 3
        state.zoomImgHeight = offsetHeight * 3
        state.imgSrc = img.src
        state.boxX = offsetWidth
        state.boxY = offsetTop
        state.isShow = true
    }
    const zoomMove = (ev) => {
        const {clientX, clientY} = ev
        const {offsetTop, offsetLeft} = ev.currentTarget
        const {offsetWidth, offsetHeight} = imgbox.value
        let mx = clientX - offsetLeft - (maskWidth / 2),
            my = clientY - offsetTop - (maskHeight / 2)
        mx = mx < 0 ? 0 : mx
        my = my < 0 ? 0 : my
        if (mx > offsetWidth - maskWidth / 2) {
            mx = offsetWidth - maskWidth / 2
        }
        if (my > offsetHeight - maskHeight / 2) {
            my = offsetHeight - maskHeight / 2
        }
        state.maskX = mx
        state.maskY = my

        state.bImgX = mx * (state.zoomImgWidth - offsetWidth) / (offsetWidth - maskWidth)
        state.bImgY = my * (state.zoomImgWidth - offsetWidth) / (offsetWidth - maskWidth)
    }
    const zommOut = () => {
        state.isShow = false
    }
    //绑定方法
    onMounted(() => {
        imgbox.value.addEventListener('mouseover', zommIn)
        imgbox.value.addEventListener('mousemove', zoomMove)
        imgbox.value.addEventListener('mouseout', zommOut)
    })
    onUnmounted(() => {
        imgbox.value.removeEventListener('mouseover', () => {
        })
        imgbox.value.removeEventListener('mousemove', () => {
        })
        imgbox.value.removeEventListener('mouseout', () => {
        })
    })
</script>

<style scoped>
    .mask {
        width: 50px;
        height: 50px;
        background-color: #fff;
        opacity: .6;
        cursor: crosshair;
        position: absolute;
    }

    .imgBox {
        position: relative;
        overflow: hidden;
        width: 200px;
        height: 200px;
    }

    .zoomBox {
        width: 200px;
        height: 200px;
        position: absolute;
        border: 1px solid #eee;
        background: #fff;
        overflow: hidden;
    }

    .fade-enter-active, .fade-leave-active {
        transition: opacity .5s
    }

    .fade-enter, .fade-leave-active {
        opacity: 0
    }
</style>
