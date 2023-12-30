<script setup lang="ts">
import { computed, ref } from 'vue'

const audioSources = ref<HTMLAudioElement[]>([])
const runningTest = ref(false)
const currentTest = ref(1)
const currentAudio = computed(() => audioSources.value[currentTest.value - 1])

function startTest() {
  console.log(currentAudio.value)
  if (runningTest.value || !currentAudio.value)
    return;
  runningTest.value = true
  currentAudio.value.play()
}

function stopTest() {
  if (!runningTest.value || !currentAudio.value)
    return;
  runningTest.value = false
  currentAudio.value.pause()
  currentAudio.value.currentTime = 0
}

function endTest() {
  stopTest()
  currentTest.value++
}

function randomTest(): number {
  let test
  while ((test = Math.floor(Math.random() * 12) + 1) === currentTest.value);
  return test
}
</script>

<template>
  <div class="test-info">
    <p class="currentTest">
      Current Test
      <span>{{ currentTest }}</span>
    </p>

    <div class="test-controls">
      <button @click.prevent="currentTest--">Previous</button>
      <button @click.prevent="currentTest = randomTest()">Random</button>
      <button @click.prevent="currentTest++">Next</button>
    </div>
  </div>

  <div class="controls">
    <button>Calibrate</button>

    <button v-if="runningTest" @click.prevent="stopTest">Cancel Test</button>
    <button v-else @click.prevent="startTest">Begin Test</button>
  </div>

  <template v-for="i in 12">
    <audio ref="audioSources" @ended="endTest">
      <source :src="`/audio/list-${i.toString().padStart(2, '0')}.wav`" type="audio/wav" />
    </audio>
  </template>
</template>

<style scoped>
.test-info {
  margin-bottom: 1rem;
}

.test-controls {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
  gap: 2rem;

  font-size: 1.5rem;
}

.controls {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
  gap: 2rem;

  font-size: 1.5rem;
}

.currentTest {
  font-size: 1.25rem;
}

.currentTest span {
  margin-left: 0.5rem;
  font-size: 1.75rem;
  font-weight: 700;
}
</style>
