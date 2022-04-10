<template>
  <div
    ref="keyboard"
    class="keyboard"
  >
    <div
      v-for="i in 1"
      :key="i"
      class="keys"
    >
      <div
        v-for="key in keys"
        :key="key.note"
        class="key"
        :class="{ sharp: key.sharp, active: key.active }"
      >
        {{ key.displayedKeymap }}
      </div>
      <!-- <div class="key">A</div>
      <div class="key sharp">Q</div>
      <div class="key"></div>
      <div class="key sharp"></div>
      <div class="key"></div>
      <div class="key"></div>
      <div class="key sharp"></div>
      <div class="key"></div>
      <div class="key sharp"></div>
      <div class="key"></div>
      <div class="key sharp"></div>
      <div class="key"></div> -->
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'

type Key = {
  active: boolean,
  displayedKeymap: string,
  keymap: string,
  note: string,
  sharp: boolean
}

export default defineComponent({
  name: 'App',

  data () {
    return {
      audioContext: null as AudioContext | null,
      gainControl: null as GainNode | null,
      currentOctave: 5,
      keys: [
        { active: false, displayedKeymap: 'A', keymap: 'KeyA', note: 'A1', sharp: false },
        { active: false, displayedKeymap: 'W', keymap: 'KeyW', note: 'AS1', sharp: true },
        { active: false, displayedKeymap: 'S', keymap: 'KeyS', note: 'B1', sharp: false },
        { active: false, displayedKeymap: 'D', keymap: 'KeyD', note: 'C2', sharp: false },
        { active: false, displayedKeymap: 'R', keymap: 'KeyR', note: 'CS2', sharp: true },
        { active: false, displayedKeymap: 'F', keymap: 'KeyF', note: 'D2', sharp: false },
        { active: false, displayedKeymap: 'T', keymap: 'KeyT', note: 'DS2', sharp: true },
        { active: false, displayedKeymap: 'G', keymap: 'KeyG', note: 'E2', sharp: false },
        { active: false, displayedKeymap: 'H', keymap: 'KeyH', note: 'F2', sharp: false },
        { active: false, displayedKeymap: 'U', keymap: 'KeyU', note: 'FS2', sharp: true },
        { active: false, displayedKeymap: 'J', keymap: 'KeyJ', note: 'G2', sharp: false },
        { active: false, displayedKeymap: 'I', keymap: 'KeyI', note: 'GS2', sharp: true },
        { active: false, displayedKeymap: 'K', keymap: 'KeyK', note: 'A2', sharp: false },
        { active: false, displayedKeymap: 'O', keymap: 'KeyO', note: 'AS2', sharp: true },
        { active: false, displayedKeymap: 'L', keymap: 'KeyL', note: 'B2', sharp: false },
        { active: false, displayedKeymap: 'Ç', keymap: 'Semicolon', note: 'C3', sharp: false },
        { active: false, displayedKeymap: '´', keymap: 'BracketLeft', note: 'CS3', sharp: true },
        { active: false, displayedKeymap: '~', keymap: 'Quote', note: 'D3', sharp: false },
        { active: false, displayedKeymap: '[', keymap: 'BracketRight', note: 'DS3', sharp: true },
        { active: false, displayedKeymap: ']', keymap: 'Backslash', note: 'E3', sharp: false },
        { active: false, displayedKeymap: 'Enter', keymap: 'Enter', note: 'F3', sharp: false }
      ]
    }
  },

  created () {
    this.audioContext = new AudioContext()

    const audioBuffer = this.audioContext.createBuffer(1, 0.3 * this.audioContext.sampleRate, this.audioContext.sampleRate)

    const channelData = audioBuffer.getChannelData(0)

    for (let i = 0; i < audioBuffer.length; i++) {
      channelData[i] = Math.random() * 2 - 1
    }

    this.gainControl = this.audioContext.createGain()

    this.gainControl.gain.setValueAtTime(0.05, 0)
    this.gainControl.connect(this.audioContext.destination)

    window.addEventListener('keydown', (e) => {
      if (e.repeat) return

      const key = this.keys.find(key => key.keymap === e.code)

      if (key) {
        key.active = true
        this.play(key)
        // const audioSource = this.audioContext.createBufferSource()
        // audioSource.buffer = audioBuffer
        // audioSource.connect(gain)
        // audioSource.start()
      }
    })

    window.addEventListener('keyup', (e) => {
      const key = this.keys.find(key => key.keymap === e.code)

      if (key) {
        key.active = false
      }
    })
  },

  methods: {
    play (key: Key) {
      if (!this.audioContext || !this.gainControl) return

      const octave = parseInt(key.note.slice(-1))
      const oscillator = this.audioContext.createOscillator()
      oscillator.frequency.value = 440.0 * (2 ** octave) // this.currentOctave
      oscillator.connect(this.gainControl)
      oscillator.start()
      oscillator.stop(this.audioContext.currentTime + 0.3)
    },

    playSnare () {
      //
    }
  }
})
</script>

<style lang="sass">
$key-width: 50px

*
  padding: 0
  margin: 0
  box-sizing: border-box

#app
  font-family: Avenir, Helvetica, Arial, sans-serif
  -webkit-font-smoothing: antialiased
  -moz-osx-font-smoothing: grayscale
  text-align: center
  color: #2c3e50
  margin-top: 60px
  padding: 10px
  display: flex
  flex-direction: column
  align-items: center

.keyboard
  display: flex
  justify-content: flex-start
  // gap: 2px
  // width: 20vw
  // width: 300px
  height: 300px
  padding: 20px
  border-radius: 10px
  background: #5af

.keys
  display: flex

.key
  background-color: white
  width: $key-width
  height: 100%
  box-shadow: 1px 0px 2px 2px #0005
  border-radius: 5px
  display: flex
  flex-direction: column
  justify-content: flex-end
  padding: 20px 0
  font-weight: bold

  &.active
    background-color: #ccf

.key.sharp
  box-shadow: none
  z-index: 1
  $width: calc($key-width - 20px)
  width: calc($width)
  height: 70%
  margin-left: calc($width / -2)
  margin-right: calc($width / -2)
  background-color: #222
  color: white

  &.active
    background-color: #55a

</style>
