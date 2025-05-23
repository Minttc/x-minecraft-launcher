<template>
  <div
    v-if="currentUrl"
    class="non-moveable flex flex-grow-0"
  >
    <span class="max-w-40  overflow-hidden overflow-ellipsis whitespace-nowrap">
      <span class="bar_content">
        {{ basename(currentUrl, '/') }}
      </span>
    </span>
    <v-btn
      icon
      x-small
      @click="prev"
    >
      <v-icon class="mr-0">
        skip_previous
      </v-icon>
    </v-btn>
    <v-btn
      icon
      x-small
      @click="play"
    >
      <v-icon class="mr-0">
        {{ playing ? 'pause' : 'play_arrow' }}
      </v-icon>
    </v-btn>
    <v-btn
      icon
      x-small
      @click="next"
    >
      <v-icon class="mr-0">
        skip_next
      </v-icon>
    </v-btn>
    <v-btn
      icon
      x-small
      @click="shuffle = !shuffle"
    >
      <v-icon class="mr-0">
        {{ shuffle ? 'shuffle' : 'repeat' }}
      </v-icon>
    </v-btn>
    <!-- <v-slider
      class="h-5 w-5"
      hide-details
      dense
      max="100"
      min="0"
    /> -->
    <v-menu
      offset-y
    >
      <template #activator="{ on, attrs }">
        <v-btn
          icon
          x-small
          v-bind="attrs"
          v-on="on"
          @wheel="onWheel"
        >
          <v-icon class="mr-0">
            volume_up
          </v-icon>
        </v-btn>
      </template>
      <v-sheet
        class="w-5 overflow-hidden rounded"
        @wheel="onWheel"
      >
        <v-slider
          v-model="volume"
          vertical
          hide-details
          :step="0.01"
          :max="1"
          :min="0"
          dense
        />
      </v-sheet>
    </v-menu>
    <audio
      ref="audio"
      :src="currentUrl"
      :type="currentMineType"
    />
  </div>
</template>
<script lang="ts" setup>
import { injection } from '@/util/inject'
import { basename } from '@/util/basename'
import { useLocalStorage, useMediaControls } from '@vueuse/core'
import { kTheme } from '@/composables/theme'
import { kInstanceLaunch } from '@/composables/instanceLaunch'

defineProps<{ }>()

const { currentTheme } = injection(kTheme)
const index = ref(0)
const currentBackgroundMusics = computed(() => currentTheme.value?.backgroundMusic || [])
const currentUrl = computed(() => currentBackgroundMusics.value[index.value]?.url)
const currentMineType = computed(() => currentBackgroundMusics.value[index.value]?.mimeType)

const audio = ref<HTMLAudioElement | null>(null)
const { playing, volume, ended } = useMediaControls(audio)

const shuffle = useLocalStorage('audioShuffle', false)

onMounted(() => {
  playing.value = localStorage.getItem('audioPlaying') === 'true'
})

watch(playing, (v) => {
  localStorage.setItem('audioPlaying', v.toString())
})

const { gameProcesses } = injection(kInstanceLaunch)

let lastState = undefined as boolean | undefined
watch(computed(() => gameProcesses.value.length), (cur, last) => {
  if (cur > 0 && last === 0) {
    lastState = playing.value
    audio.value?.pause()
  } else if (cur === 0 && last > 0) {
    if (lastState) {
      audio.value?.play()
    }
  }
})

onMounted(() => {
  const v = localStorage.getItem('audioVolume')
  if (v) {
    volume.value = parseFloat(v)
  }
})

watch(volume, (v) => {
  localStorage.setItem('audioVolume', v.toString())
})

const play = () => {
  playing.value = !playing.value
}
const next = async () => {
  audio.value?.pause()
  // play next
  if (!shuffle.value) {
    index.value = (index.value + 1) % currentBackgroundMusics.value.length
  } else {
    // should not play the same music
    let nextIndex = index.value
    while (nextIndex === index.value) {
      nextIndex = Math.floor(Math.random() * currentBackgroundMusics.value.length)
    }
    index.value = nextIndex
  }
  await nextTick()
  audio.value?.play()
}
const prev = async () => {
  audio.value?.pause()
  // play prev
  index.value = (index.value - 1 + currentBackgroundMusics.value.length) % currentBackgroundMusics.value.length
  await nextTick()
  audio.value?.play()
}

watch(ended, (isEnded) => {
  if (isEnded) {
    if (currentBackgroundMusics.value.length > 1) {
      next()
    } else {
      audio.value?.play()
    }
  }
})

const onWheel = (e: WheelEvent) => {
  volume.value = Math.max(0, Math.min(1, volume.value - e.deltaY / 1000))
}
</script>
<style scoped>

.bar_content {
  display: block;
  width: fit-content;

  transform: translateX(160px); /* Animation start out of the screen */

  /* Add the animation */
  animation: move 20s linear infinite /* infinite make reapeat the animation indefinitely */;
}
/* Create the animation */
@keyframes move {
  to { transform: translateX(-100%); }
}
</style>
