<template>
    <div class="logo">NIKꞰIN</div>
    <div class="line"></div>
    <div class="player">
        <div class="player__track"
             @click="playSound"
        >
            <img class="logo"
                 src="/play.svg"
                 alt="play"
            />
            <div class="track__name track__text">
                Lorem ipsum dolor sit amet consectetur adipisicing elit.
            </div>
            <div class="track__time track__text">
                1:42:45
            </div>
        </div>
    </div>
    <canvas class="canvas" id="canvas"></canvas>
    <audio class="audio" id="audio"></audio>
</template>

<script setup>
import {onMounted, ref} from "vue";
import getRndInt from "./helpers/getRndInt.js";

const height = ref()
const width = ref()
const ctx = ref(null)

let isPlay = false
let isDraw = false

let audioCtx
let audioElement
let analyser
let bufferLength
let audioEle
let dataArray
let posX
let points = []
let barWidth = 0
let analyser_fftSize = 256

const draw = () => {
    if (!isDraw) return
    analyser.getByteFrequencyData(dataArray)
    ctx.value.clearRect(0, 0, width.value, height.value)

    posX = 0
    for (let i = 0; i < dataArray.length; i++) {
        points[i] = {
            x: posX,
            y: 256 - dataArray[dataArray.length - 1 - i]
        }
        posX += barWidth
    }

    points[points.length - 1].y = points[points.length - 2].y

    for (let i = 0; i < dataArray.length; i++) {
        points[i+dataArray.length] = {
            x: posX,
            y: 256 - dataArray[i]
        }
        posX += barWidth
    }

    for (let i = points.length - 1; i > 1; i--) {
        points[i].y = points[i - 1].y
    }

    for (let i = 0; i < 8; i++) {
        drawLine(ctx.value, points.map((item) => {
            item.y += 4 * i
            return item
        }), `hsla(154, 100%, 69%, ${0.1+i/10})` )
    }
    // drawLine(ctx.value, points)
    window.requestAnimationFrame(draw);
}

const drawLine = (ctx, points, color) => {
    ctx.beginPath();
    ctx.moveTo(points[0].x, points[0].y);
    ctx.strokeStyle = color;

    for (let i = 0; i < points.length; i++) {

        ctx.lineTo(points[i].x, points[i].y)
    }
    ctx.stroke()
    ctx.save()
}
const playSound = () => {
    if (isPlay) {
        audioEle.pause()
        isPlay = false
        isDraw = false
        return
    }
    const src = `./music/01_Electro_mix.mp3`
    audioEle = new Audio()
    audioEle.src = src
    audioCtx = new AudioContext()
    const audioSourceNode = audioCtx.createMediaElementSource(audioEle)

    //Create analyser node
    analyser = audioCtx.createAnalyser();
    analyser.fftSize = analyser_fftSize ;
    bufferLength = analyser.frequencyBinCount;
    dataArray = new Uint8Array(bufferLength);

    //Set up audio node network
    audioSourceNode.connect(analyser);
    analyser.connect(audioCtx.destination);

    isPlay = true
    isDraw = true
    audioEle.play()

    draw();
}

onMounted(() => {
    height.value = document.body.clientHeight
    width.value = document.body.clientWidth

    const elem = document.getElementById('canvas')
    ctx.value = elem.getContext('2d')

    elem.width = width.value
    elem.height = 400

    barWidth = width.value / 256
    posX = 0

    for (let i = 0; i < 256; i++) {
        points.push({
            x: posX,
            y: 256
        })
        posX += barWidth
    }
})
</script>