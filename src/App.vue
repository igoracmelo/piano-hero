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
  sharp: boolean,
  semitones: number
}

export default defineComponent({
  name: 'App',

  data () {
    return {
      audioContext: null as AudioContext | null,
      gainControl: null as GainNode | null,
      currentOctave: 2,
      keyNodes: {
        A1: null,
        AS1: null,
        B1: null,
        C2: null,
        CS2: null,
        D2: null,
        DS2: null,
        E2: null,
        F2: null,
        FS2: null,
        G2: null,
        GS2: null,
        A2: null,
        AS2: null,
        B2: null,
        C3: null,
        CS3: null,
        D3: null,
        DS3: null,
        E3: null,
        F3: null
      } as Record<string, GainNode|null>,
      keys: [
        { active: false, displayedKeymap: 'A', keymap: 'KeyA', note: 'A1', sharp: false, semitones: 0 },
        { active: false, displayedKeymap: 'W', keymap: 'KeyW', note: 'AS1', sharp: true, semitones: 1 },
        { active: false, displayedKeymap: 'S', keymap: 'KeyS', note: 'B1', sharp: false, semitones: 2 },
        { active: false, displayedKeymap: 'D', keymap: 'KeyD', note: 'C2', sharp: false, semitones: 3 },
        { active: false, displayedKeymap: 'R', keymap: 'KeyR', note: 'CS2', sharp: true, semitones: 4 },
        { active: false, displayedKeymap: 'F', keymap: 'KeyF', note: 'D2', sharp: false, semitones: 5 },
        { active: false, displayedKeymap: 'T', keymap: 'KeyT', note: 'DS2', sharp: true, semitones: 6 },
        { active: false, displayedKeymap: 'G', keymap: 'KeyG', note: 'E2', sharp: false, semitones: 7 },
        { active: false, displayedKeymap: 'H', keymap: 'KeyH', note: 'F2', sharp: false, semitones: 8 },
        { active: false, displayedKeymap: 'U', keymap: 'KeyU', note: 'FS2', sharp: true, semitones: 9 },
        { active: false, displayedKeymap: 'J', keymap: 'KeyJ', note: 'G2', sharp: false, semitones: 10 },
        { active: false, displayedKeymap: 'I', keymap: 'KeyI', note: 'GS2', sharp: true, semitones: 11 },
        { active: false, displayedKeymap: 'K', keymap: 'KeyK', note: 'A2', sharp: false, semitones: 12 },
        { active: false, displayedKeymap: 'O', keymap: 'KeyO', note: 'AS2', sharp: true, semitones: 13 },
        { active: false, displayedKeymap: 'L', keymap: 'KeyL', note: 'B2', sharp: false, semitones: 14 },
        { active: false, displayedKeymap: 'Ç', keymap: 'Semicolon', note: 'C3', sharp: false, semitones: 15 },
        { active: false, displayedKeymap: '´', keymap: 'BracketLeft', note: 'CS3', sharp: true, semitones: 16 },
        { active: false, displayedKeymap: '~', keymap: 'Quote', note: 'D3', sharp: false, semitones: 17 },
        { active: false, displayedKeymap: '[', keymap: 'BracketRight', note: 'DS3', sharp: true, semitones: 18 },
        { active: false, displayedKeymap: ']', keymap: 'Backslash', note: 'E3', sharp: false, semitones: 19 },
        { active: false, displayedKeymap: 'Enter', keymap: 'Enter', note: 'F3', sharp: false, semitones: 20 },
        { active: false, displayedKeymap: '7', keymap: 'Numpad7', note: 'FS3', sharp: true, semitones: 21 },
        { active: false, displayedKeymap: '4', keymap: 'Numpad4', note: 'G3', sharp: false, semitones: 22 },
        { active: false, displayedKeymap: '8', keymap: 'Numpad8', note: 'GS3', sharp: true, semitones: 23 },
        { active: false, displayedKeymap: '5', keymap: 'Numpad5', note: 'A3', sharp: false, semitones: 24 }
        //
        // App.vue?47b3:104
        // App.vue?47b3:104
        // App.vue?47b3:104
      ] as Key[]
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

    this.gainControl.gain.setValueAtTime(0.1, 0)
    this.gainControl.connect(this.audioContext.destination)

    window.addEventListener('keydown', (e) => {
      if (e.repeat) return

      if (e.code.startsWith('Digit')) {
        this.currentOctave = parseInt(e.code.slice(-1))
      }

      const key = this.keys.find(key => key.keymap === e.code)

      if (key && !this.isPlaying(key)) {
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

      if (key && this.isPlaying(key)) {
        key.active = false
        this.stop(key)
      }
    })
  },

  methods: {

    // createFadeOut (duration?: number): GainNode {
    //   if (!this.audioContext) throw new Error()

    //   const fadeOut = this.audioContext.createGain()
    //   // fadeOut.gain.setValueAtTime(1, 0)

    //   return fadeOut
    // },

    isPlaying (key: Key): boolean {
      return !!this.keyNodes[key.note]
    },

    play (key: Key) {
      if (!this.audioContext || !this.gainControl) return

      const fadeOut = this.audioContext.createGain()
      fadeOut.connect(this.gainControl)

      // const startFadeAt = this.audioContext.currentTime + 10
      // setTimeout(() => this.stop(key), startFadeAt)
      // fadeOut.gain.setValueAtTime(1, startFadeAt)
      // fadeOut.gain.exponentialRampToValueAtTime(0.001, startFadeAt + 0.5)

      const oscillator = this.audioContext.createOscillator()
      oscillator.connect(fadeOut)

      this.keyNodes[key.note] = fadeOut

      const semitoneFactor = 2.0 ** (1 / 12)
      const firstNote = 440.0 * (2 ** (this.currentOctave - 2))
      const frequency = firstNote * semitoneFactor ** key.semitones

      oscillator.frequency.value = frequency
      // oscillator.connect(this.gainControl)
      oscillator.start()
    },

    stop (key: Key) {
      const fadeOut = this.keyNodes[key.note]
      if (!this.audioContext || !this.gainControl || !fadeOut) return

      const stopAt = this.audioContext.currentTime + 1
      // fadeOut.gain.setValueAtTime(1, 0)
      fadeOut.gain.exponentialRampToValueAtTime(0.001, stopAt)

      this.keyNodes[key.note] = null
      // const fadeOut = this.createFadeOut(stopAt)
      // oscillator.connect(fadeOut)
      // oscillator.stop(stopAt)
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
  // margin-top: 60px
  padding: 10px
  display: grid
  place-items: center
  width: 100%
  height: 100vh
  // flex-direction: column
  // align-items: center
  background: #5af

.keyboard
  display: flex
  justify-content: flex-start
  // gap: 2px
  // width: 20vw
  // width: 300px
  height: 300px
  padding: 20px
  border-radius: 10px
  // background: #5af

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
