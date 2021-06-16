<template>
  <div
    class="text-white text-opacity-50 p-5 h-full text-sm overflow-y-auto"
    ref="logsContainer"
  >
    <p v-for="log of displayedLogs">
      {{ log.msg }} <span v-if="log.micId">{{ log.micId }}</span>
    </p>
    <div ref="endAnchor"></div>
  </div>
</template>

<script lang="ts">
import { defineComponent, nextTick, PropType, ref, watch } from 'vue'

type Log = {
  hostname: string
  level: number
  msg: string
  name: string
  pid: number
  time: number
  micId?: string
}

type Params = {
  play: boolean
  seekTime: number
  currentTime: number
  speed: number
}

export default defineComponent({
  props: {
    file: { type: Object as PropType<File> },
    params: { type: Object as PropType<Params>, required: true },
  },
  setup(props) {
    let logs: Log[] = []
    let cursor: number = 0
    let beginTime: number
    const displayedLogs = ref<Log[]>([])
    const logsContainer = ref<HTMLElement>()
    const endAnchor = ref<HTMLElement>()

    const scrollLogs = () =>
      nextTick(() => endAnchor.value?.scrollIntoView({ behavior: 'smooth' }))

    const updateView = (seekTime: number) => {
      displayedLogs.value = []
      const time = beginTime + seekTime * 1000
      cursor = 0
      for (let i = logs.length - 1; i > 0; i--) {
        if (logs[i].time <= time) {
          if (cursor === 0) cursor = i
          displayedLogs.value.unshift(logs[i])
        }
        if (displayedLogs.value.length > 100) break
      }
      scrollLogs()
    }

    let nextLogTimeout: number | undefined
    const checkLog = (seekTime?: number) => {
      if (logs.hasOwnProperty(cursor) && logs.hasOwnProperty(cursor + 1)) {
        const currentTime = seekTime
          ? beginTime + seekTime * 1000
          : logs[cursor].time
        nextLogTimeout = setTimeout(() => {
          cursor++
          displayedLogs.value.push(logs[cursor])
          if (displayedLogs.value.length > 100) displayedLogs.value.shift()
          scrollLogs()
          checkLog()
        }, (logs[cursor + 1].time - currentTime) / props.params.speed)
      }
    }

    watch(
      () => props.params.play,
      (play) => {
        if (play) checkLog(props.params.currentTime)
        else clearTimeout(nextLogTimeout)
      }
    )

    watch(
      () => props.params.seekTime,
      (seekTime) => {
        if (props.params.play) clearTimeout(nextLogTimeout)
        updateView(seekTime)
        if (props.params.play) checkLog(seekTime)
      }
    )

    watch(
      () => props.params.speed,
      (speed) => {
        if (props.params.play) {
          clearTimeout(nextLogTimeout)
          checkLog(props.params.currentTime)
        }
      }
    )

    watch(
      () => props.file,
      (file) => {
        if (file) {
          const fileReader = new FileReader()
          fileReader.onload = () => {
            const text = fileReader.result as string
            const lines = text.split(/\r\n|\n/)
            logs = []
            lines.forEach((line) => {
              try {
                logs.push(JSON.parse(line))
              } catch (e) {}
            })

            const firstLog = logs.find((log) =>
              log.msg.includes('Recording started.')
            )
            if (!firstLog) {
              console.error('Recording beginning not found.')
              return
            }
            beginTime = firstLog.time
            updateView(props.params.currentTime)
          }
          fileReader.readAsText(file)
        }
      }
    )

    return { displayedLogs, logsContainer, endAnchor }
  },
})
</script>

<style></style>
