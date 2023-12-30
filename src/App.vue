<script setup lang="ts">
import { computed, ref } from 'vue'

const words = [
  ['ship', 'rug', 'fan', 'cheek', 'haze', 'dice', 'both', 'well', 'jot', 'move'],
  ['fish', 'duck', 'gap', 'cheese', 'rail', 'hive', 'bone', 'wedge', 'moss', 'tooth'],
  ['c1', 'c2', 'c3', 'c4', 'c5', 'c6', 'c7', 'c8', 'c9', 'c10'],
  ['d1', 'd2', 'd3', 'd4', 'd5', 'd6', 'd7', 'd8', 'd9', 'd10'],
  ['fib', 'thatch', 'sum', 'heel', 'wide', 'rake', 'goes', 'shop', 'vet', 'june'],
  ['fill', 'catch', 'thumb', 'heap', 'wise', 'rave', 'goat', 'shone', 'bed', 'juice'],
  ['g1', 'g2', 'g3', 'g4', 'g5', 'g6', 'g7', 'g8', 'g9', 'g10'],
  ['h1', 'h2', 'h3', 'h4', 'h5', 'h6', 'h7', 'h8', 'h9', 'h10'],
  ['i1', 'i2', 'i3', 'i4', 'i5', 'i6', 'i7', 'i8', 'i9', 'i10'],
  ['j1', 'j2', 'j3', 'j4', 'j5', 'j6', 'j7', 'j8', 'j9', 'j10'],
  ['k1', 'k2', 'k3', 'k4', 'k5', 'k6', 'k7', 'k8', 'k9', 'k10'],
  ['l1', 'l2', 'l3', 'l4', 'l5', 'l6', 'l7', 'l8', 'l9', 'l10'],
]

const audioSources = ref<HTMLAudioElement[]>([])
const calibrationTone = ref<HTMLAudioElement | null>(null)
const scores = ref<number[]>(Array(12).fill(0))
const playingCalibration = ref(false)
const runningTest = ref(false)
const currentTest = ref(1)
const currentAudio = computed(() => audioSources.value[currentTest.value - 1])
const currentWordlist = computed(() => words[currentTest.value - 1])
const score = computed(() => scores.value.reduce((a, b) => a + b))

function startTest() {
  if (runningTest.value || !currentAudio.value)
    return;
  runningTest.value = true
  currentAudio.value.play()
  scores.value.fill(0)
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

function calibrate() {
  if (playingCalibration.value || !calibrationTone.value)
    return
  playingCalibration.value = true
  calibrationTone.value.play()
}
function stopCalibration() {
  if (!playingCalibration.value || !calibrationTone.value)
    return
  playingCalibration.value = false
  calibrationTone.value.pause()
  calibrationTone.value.currentTime = 0
}
</script>

<template>
  <div class="status-info">
    <p>
      Current Test
      <span>{{ currentTest >= 9 ? currentTest + 2 : currentTest }}</span>
    </p>

    <p>
      Score
      <span>{{ score }}</span>
    </p>
  </div>

  <div class="test-info">
    <div class="test-controls">
      <button @click.prevent="currentTest--">Previous</button>
      <button @click.prevent="currentTest = randomTest()">Random</button>
      <button @click.prevent="currentTest++">Next</button>
    </div>
  </div>

  <div class="controls">
    <button v-if="playingCalibration" @click.prevent="stopCalibration">
      End Calibration
    </button>
    <button v-else @click.prevent="calibrate">Calibrate</button>

    <button v-if="runningTest" @click.prevent="stopTest">Cancel Test</button>
    <button v-else @click.prevent="startTest">Begin Test</button>
  </div>

  <template v-for="i in 12">
    <audio ref="audioSources" @ended="endTest">
      <source :src="`/audio/list-${i.toString().padStart(2, '0')}.wav`" type="audio/wav" />
    </audio>
  </template>

  <ul>
    <template v-for="i in currentWordlist.length">
      <li class="word">
        <p>{{ currentWordlist[i - 1] }}</p>

        <div>
          <input type="radio" :value="3" :id="`${currentWordlist[i - 1]}-3`" :name="currentWordlist[i - 1]"
            v-model="scores[i - 1]" :v-bind:value="3" />
          <label :for="`${currentWordlist[i - 1]}-3`">3</label>

          <input type="radio" :value="2" :id="`${currentWordlist[i - 1]}-2`" :name="currentWordlist[i - 1]"
            v-model="scores[i - 1]" :v-bind:value="2" />
          <label :for="`${currentWordlist[i - 1]}-2`">2</label>

          <input type="radio" :value="1" :id="`${currentWordlist[i - 1]}-1`" :name="currentWordlist[i - 1]"
            v-model="scores[i - 1]" :v-bind:value="1" />
          <label :for="`${currentWordlist[i - 1]}-1`">1</label>

          <input type="radio" :value="0" :id="`${currentWordlist[i - 1]}-0`" :name="currentWordlist[i - 1]"
            v-model="scores[i - 1]" :v-bind:value="0" />
          <label :for="`${currentWordlist[i - 1]}-0`">0</label>
        </div>
      </li>
    </template>
  </ul>

  <audio ref="calibrationTone" @ended="stopCalibration">
    <source src="/audio/calibration.wav" type="audio/wav" />
  </audio>
</template>

<style scoped>
.test-info {
  margin-bottom: 1rem;
}

.test-controls {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
  gap: 2rem;

  font-size: 1.5rem;
}

.controls {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
  gap: 2rem;

  margin-bottom: 2rem;
  font-size: 1.5rem;
}

.status-info {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  align-items: center;
  justify-content: space-evenly;

  font-size: 1.25rem;
}

.status-info span {
  margin-left: 0.5rem;
  font-size: 1.75rem;
  font-weight: 700;
}

.word {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  align-items: center;
  justify-content: space-between;

  font-size: 1.25rem;
}

.word p {
  margin: 0;
}

.word input {
  display: none;
}

.word label {
  margin-right: 1rem;
  font-weight: 300;
}

.word input:checked+label {
  font-weight: 700;
}
</style>
