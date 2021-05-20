<template>
  <div class="flex h-full">
    <div class="flex-1 bg-gray-700 h-full relative">
      <vue-plyr :options="{}" ref="videoPlayer">
        <video controls playsinline :src="videoUrl"></video>
      </vue-plyr>
      <div class="w-full absolute top-0 left-0 text-center p-2">
        <FilePicker @change="pickedVideo($event)">Pick Video</FilePicker>
      </div>
    </div>
    <div class="bg-gray-900 h-full w-96 relative">
      <Logs :file="logFile" :play="playing"></Logs>
      <div class="w-full absolute top-0 left-0 text-center p-2">
        <FilePicker @change="pickedLogs($event)">Pick Logs</FilePicker>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, onMounted, ref } from 'vue'
import FilePicker from './components/FilePicker.vue'
import Logs from './components/Logs.vue'

export default defineComponent({
  name: 'App',
  components: { FilePicker, Logs },
  setup() {
    const videoSource = ref<HTMLSourceElement>()
    const videoPlayer = ref<any>()
    const videoUrl = ref('')
    const logFile = ref<File>()
    const playing = ref(false)

    const pickedVideo = (e: Event) => {
      const input = e.currentTarget as HTMLInputElement
      if (input.files && input.files.length > 0) {
        videoUrl.value = URL.createObjectURL(input.files[0])
        // load logs
      }
    }

    const pickedLogs = (e: Event) => {
      const input = e.currentTarget as HTMLInputElement
      if (input.files && input.files.length > 0) {
        logFile.value = input.files[0]
      }
    }

    onMounted(() => {
      if (videoPlayer.value) {
        videoPlayer.value.player.on('playing', () => (playing.value = true))
        videoPlayer.value.player.on('pause', () => (playing.value = false))
        videoPlayer.value.player.on('seeked', (e) =>
          console.log(e.detail.plyr.currentTime)
        )
      }
    })

    return {
      pickedVideo,
      pickedLogs,
      videoSource,
      videoPlayer,
      videoUrl,
      logFile,
      playing,
    }
  },
})
</script>

<style>
html,
body,
#app {
  @apply w-full h-full;
}

#app {
}
</style>
