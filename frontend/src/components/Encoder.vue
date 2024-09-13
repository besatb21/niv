<template>
    <input @input="handleInput" type="file" accept="audio/*" />
    <div v-if="audio">
        <button @click="transcode">Convert</button>
        {{ message }}
    </div>
    {{ audio ? audio.name : "Hello" }}
    <audio v-if="audio" :src="audio" controls></audio>

</template>

<script setup lang="ts">
import { FFmpeg } from '@ffmpeg/ffmpeg'
import type { LogEvent } from '@ffmpeg/ffmpeg/dist/esm/types'
import { fetchFile, toBlobURL } from '@ffmpeg/util'
import { defineComponent, ref } from 'vue'

const baseURL = 'https://unpkg.com/@ffmpeg/core-mt@0.12.6/dist/esm'
const videoURL = 'https://github.com/ffmpegwasm/testdata/blob/c81125391a0ada7599edc6bff2da51c1a3ed38d0/audio-15s.wav'


const ffmpeg = new FFmpeg()
const message = ref('Click Start to Transcode')
let audio = ref()



async function handleInput(el) {
    audio.value = el.target.files[0]
}


async function transcode() {
    message.value = 'Loading ffmpeg-core.js'
    ffmpeg.on('progress', ({ message: msg }: LogEvent) => {
        message.value = msg
    })

    await ffmpeg.load({
        coreURL: await toBlobURL(`${baseURL}/ffmpeg-core.js`, 'text/javascript'),
        wasmURL: await toBlobURL(`${baseURL}/ffmpeg-core.wasm`, 'application/wasm'),
        workerURL: await toBlobURL(`${baseURL}/ffmpeg-core.worker.js`, 'text/javascript')
    })


    const arrayBuffer = await audio.value.arrayBuffer();
    const fileData = new Uint8Array(arrayBuffer);

    await ffmpeg.writeFile(audio.value.name, fileData)
    await ffmpeg.exec(['-i', audio.value.name, '-ar', '24000', 'test.wav'])

    const data = await ffmpeg.readFile('test.wav')

    audio.value = URL.createObjectURL(new Blob([(data as Uint8Array).buffer], { type: 'audio/wav' }))
    console.log(audio.value);

}

</script>

<style>
#app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
}
</style>