<template>

    <h1>NIKꞰIN</h1>
    <div class="line"></div>
    <div class="player">
        <div class="player__track"
             @click="playSound"
        >
            <img class="logo"
                 src="/play.svg"
                 alt="play"
            />
            <div class="track__name">
                Lorem ipsum dolor sit amet consectetur adipisicing elit.
            </div>
            <div class="track__time">
                1:42:45
            </div>
        </div>
    </div>
    <canvas class="canvas" id="canvas"></canvas>
    <audio class="audio" id="audio"></audio>

</template>

<script setup>
import {onMounted, ref} from "vue";

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

const draw = () => {
    if (!isDraw) {
        return
    }
    analyser.getByteFrequencyData(dataArray)

    //Clear canvas
    ctx.value.clearRect(0, 0, width.value, height.value)

    const barWidth = (width.value/2) / bufferLength
    posX = 0;
    const points = []

    for (let i = dataArray.length - 1; i > 5; i--) {
        if(posX > width.value/2 - 160) continue

        points.push({
            x: posX,
            y: dataArray[i] + 120
        })
        posX += barWidth
    }

    points.push({
        x: width.value/2 - 160,
        y: 300
    })

    points.push({
        x: width.value/2 + 160,
        y: 300
    })

    posX =  width.value/2 + 160 + barWidth

    const tmpArr = [...points]

    for (let i = tmpArr.length - 3; i > 0; i--) {
        points.push({
            x: posX,
            y: points[i].y
        })
        posX += barWidth
    }

    points.push({
        x: width.value,
        y: points[points.length-1].y
    })

    for (let i = 0; i < 8; i++) {
        points.map((item) => {
            item.y += 16+i*4
        })
        drawLine(ctx.value, points)
    }
    function drawLine(ctx, points){
        ctx.beginPath();
        ctx.moveTo(points[0].x, points[0].y);
        ctx.strokeStyle = 'hsla(154, 100%, 69%, .7)';

        for (let i = 0; i < points.length; i++) {

            ctx.lineTo(points[i].x, points[i].y)
        }
        ctx.stroke()
        ctx.save()
    }
    window.requestAnimationFrame(draw);
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
    analyser.fftSize = 64;
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
    elem.height = height.value
})
</script>

<style scoped>
</style>
