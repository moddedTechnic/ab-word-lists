<script setup lang="ts">
import { computed, ref } from 'vue'

const words = [
  ['ship', 'rug', 'fan', 'cheek', 'haze', 'dice', 'both', 'well', 'jot', 'move'],
  ['fish', 'duck', 'gap', 'cheese', 'rail', 'hive', 'bone', 'wedge', 'moss', 'tooth'],
  ['thud', 'witch', 'wrap', 'jail', 'keys', 'vice', 'get', 'shown', 'hoof', 'bomb'],
  ['fun', 'will', 'that', 'shape', 'wreath', 'hide', 'guess', 'comb', 'choose', 'job'],
  ['fib', 'thatch', 'sum', 'heel', 'wide', 'rake', 'goes', 'shop', 'vet', 'june'],
  ['fill', 'catch', 'thumb', 'heap', 'wise', 'rave', 'goat', 'shone', 'bed', 'juice'],
  ['badge', 'hutch', 'kill', 'thighs', 'wave', 'reap', 'foam', 'goose', 'knot', 'shed'],
  ['bath', 'hum', 'dip', 'five', 'ways', 'reach', 'joke', 'noose', 'got', 'shell'],
  ['man', 'hip', 'thug', 'ride', 'siege', 'veil', 'chose', 'shoot', 'web', 'cough'],
  ['have', 'whizz', 'buff', 'mice', 'teeth', 'guage', 'poach', 'rule', 'den', 'cosh'],
  ['kiss', 'buzz', 'hash', 'thieve', 'gate', 'wife', 'pole', 'wretch', 'dodge', 'moon'],
  ['wish', 'dutch', 'jam', 'heath', 'laze', 'bike', 'rove', 'pet', 'fog', 'soon'],
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

function previousTest(): number {
  const test = currentTest.value - 1
  if (test < 1)
    return words.length
  return test
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
      <button @click.prevent="currentTest = previousTest()">Previous</button>
      <button @click.prevent="currentTest = randomTest()">Random</button>
      <button @click.prevent="currentTest = currentTest % words.length + 1">Next</button>
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

  max-width: 15rem;
  margin-inline: auto;
  margin-bottom: 0.25rem;
  padding-inline: 1rem 0.5rem;

  font-size: 1.25rem;
  background-color: #fff0;

  transition: background-color 175ms ease-in-out;
}

.word:hover {
  background-color: #fff3;
}

.word p {
  margin: 0;
}

.word input {
  display: none;
}

.word label {
  font-weight: 300;
  padding-inline: 0.5rem;
}

.word input:checked+label {
  font-weight: 700;
}
</style>
