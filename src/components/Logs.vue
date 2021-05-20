<template>
  <div
    class="text-white text-opacity-50 p-5 h-full text-sm overflow-y-auto"
    ref="logsContainer"
  >
    <p v-for="log of displayedLogs">{{ log.msg }}</p>
    <div ref="endAnchor"></div>
  </div>
</template>

<script lang="ts">
import {
  defineComponent,
  nextTick,
  PropType,
  ref,
  watch,
  watchEffect,
} from 'vue'

type Log = {
  hostname: string
  level: number
  msg: string
  name: string
  pid: number
  time: number
}

export default defineComponent({
  props: {
    file: { type: Object as PropType<File> },
    play: { type: Boolean },
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
      const time = beginTime + seekTime
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

    watch(
      () => props.play,
      (play) => {
        let nextLogTimeout: number | undefined
        if (play && logs.length) {
          const currentTime = logs[cursor].time
          cursor++
          console.log(logs[cursor])
          nextLogTimeout = setTimeout(() => {
            displayedLogs.value.push(logs[cursor])
            scrollLogs()
            // trigger next log timeout
          }, logs[cursor].time - currentTime)
        } else {
          if (nextLogTimeout) clearTimeout(nextLogTimeout)
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
            updateView(0)
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
