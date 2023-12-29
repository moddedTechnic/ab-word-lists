<script setup lang="ts">
import { computed, ref } from 'vue'

const audioSource = ref<HTMLAudioElement | null>(null)
const runningTest = ref(false)
const currentTest = ref(1)

const testLabel = computed(() => currentTest.value.toString().padStart(2, '0'))

function startTest() {
  if (runningTest.value || !audioSource.value)
    return;
  runningTest.value = true
  audioSource.value.play()
}

function stopTest() {
  if (!runningTest.value || !audioSource.value)
    return;
  runningTest.value = false
  audioSource.value.pause()
  audioSource.value.currentTime = 0
}
</script>

<template>
  <div class="test-controls">
    <p class="currentTest">
      Current Test
      <span>{{ currentTest }}</span>
    </p>
  </div>

  <div class="controls">
    <button>Calibrate</button>

    <button v-if="runningTest" @click.prevent="stopTest">Cancel Test</button>
    <button v-else @click.prevent="startTest">Begin Test</button>
  </div>
  <audio ref="audioSource">
    <source :src="`/audio/list-${testLabel}.wav`" type="audio/wav" />
  </audio>
</template>

<style scoped>
.controls {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-evenly;
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
