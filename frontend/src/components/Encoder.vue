<template>
  <div>
    <input @input="handleInput" type="file" accept="audio/*"/>
    <h1>{{ !audio ? "Hello" : "" }}</h1>
  </div>
  <div v-if="audio">
    <button @click="transcode">Convert</button>
    {{ message }}
  </div>

  <div v-if="convertedAudio">
    <audio :src="convertedAudio" controls></audio>
  </div>

</template>

<script setup lang="ts">
import {FFmpeg} from '@ffmpeg/ffmpeg'
import type {LogEvent} from '@ffmpeg/ffmpeg/dist/esm/types'
import {toBlobURL} from '@ffmpeg/util'
import {ref} from 'vue'

const baseURL = 'https://unpkg.com/@ffmpeg/core-mt@0.12.6/dist/esm'


const ffmpeg = new FFmpeg()
const message = ref('Click Start to Transcode')
const audio = ref()
const convertedAudio = ref()

async function handleInput(el) {
  audio.value = el.target.files[0]
}


async function transcode() {
  message.value = 'Loading ffmpeg-core.js'
  ffmpeg.on('progress', ({message: msg}: LogEvent) => {
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

  convertedAudio.value = URL.createObjectURL(new Blob([(data as Uint8Array).buffer], {type: 'audio/wav'}))
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

div:not(#app) {
  border: 2px dotted black;
  padding: 10px;
}
</style>