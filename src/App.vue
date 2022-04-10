<template>
  <div class="game">
    <div class="falling-notes">
      <div
        v-for="(key, i) in keys"
        :key="key.note"
        class="falling-note-placeholder"
        :class="{ sharp: key.sharp }"
      >
        <div
          v-for="falling in fallingNotes[key.note]"
          :key="falling"
          :style="{
            height: 50 * falling.duration - 5 + 'px',
            bottom: (falling.startsAt - currentGameTime) * 50 - 10 + 'px'
          }"
          class="falling-note"
          :class="classesForFallingNote(i, falling)"
        >
          <div />
        </div>
      </div>
    </div>
    <div
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
          :class="{ sharp: key.sharp, active: key.active, wrong: key.wrong }"
        >
          {{ key.displayedKeymap }}
        </div>
      </div>
    </div>
    <div class="below-keyboard">
      <div />
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
  wrong: boolean,
  semitones: number
}

type FallingNote = {
  duration: number,
  startsAt: number,
  hitting: boolean
}

function indexOfMin (arr: number[]): number {
  let min = Infinity
  let minIndex = 0

  for (let i = 0; i < arr.length; i++) {
    if (arr[i] < min) {
      min = arr[i]
      minIndex = i
    }
  }

  return minIndex
}

export default defineComponent({
  name: 'App',

  data () {
    return {
      audioContext: null as AudioContext | null,
      gainControl: null as GainNode | null,
      currentOctave: 2,
      currentGameTime: 0,
      fallingNotes: {
        A1: [],
        AS1: [],
        B1: [],
        C2: [],
        CS2: [],
        D2: [],
        DS2: [],
        E2: [],
        F2: [],
        FS2: [],
        G2: [],
        GS2: [],
        A2: [],
        AS2: [],
        B2: [],
        C3: [],
        CS3: [],
        D3: [],
        DS3: [],
        E3: [],
        F3: [],
        FS3: [],
        G3: [],
        GS3: [],
        A3: [],
        AS3: [],
        B3: [],
        C4: []
      } as Record<string, FallingNote[]>,
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
        F3: null,
        FS3: null,
        G3: null,
        GS3: null,
        A3: null,
        AS3: null,
        B3: null,
        C4: null
      } as Record<string, GainNode|null>,
      keys: [
        { active: false, wrong: false, displayedKeymap: 'A', keymap: 'KeyA', note: 'A1', sharp: false, semitones: 0 },
        { active: false, wrong: false, displayedKeymap: 'W', keymap: 'KeyW', note: 'AS1', sharp: true, semitones: 1 },
        { active: false, wrong: false, displayedKeymap: 'S', keymap: 'KeyS', note: 'B1', sharp: false, semitones: 2 },
        { active: false, wrong: false, displayedKeymap: 'D', keymap: 'KeyD', note: 'C2', sharp: false, semitones: 3 },
        { active: false, wrong: false, displayedKeymap: 'R', keymap: 'KeyR', note: 'CS2', sharp: true, semitones: 4 },
        { active: false, wrong: false, displayedKeymap: 'F', keymap: 'KeyF', note: 'D2', sharp: false, semitones: 5 },
        { active: false, wrong: false, displayedKeymap: 'T', keymap: 'KeyT', note: 'DS2', sharp: true, semitones: 6 },
        { active: false, wrong: false, displayedKeymap: 'G', keymap: 'KeyG', note: 'E2', sharp: false, semitones: 7 },
        { active: false, wrong: false, displayedKeymap: 'H', keymap: 'KeyH', note: 'F2', sharp: false, semitones: 8 },
        { active: false, wrong: false, displayedKeymap: 'U', keymap: 'KeyU', note: 'FS2', sharp: true, semitones: 9 },
        { active: false, wrong: false, displayedKeymap: 'J', keymap: 'KeyJ', note: 'G2', sharp: false, semitones: 10 },
        { active: false, wrong: false, displayedKeymap: 'I', keymap: 'KeyI', note: 'GS2', sharp: true, semitones: 11 },
        { active: false, wrong: false, displayedKeymap: 'K', keymap: 'KeyK', note: 'A2', sharp: false, semitones: 12 },
        { active: false, wrong: false, displayedKeymap: 'O', keymap: 'KeyO', note: 'AS2', sharp: true, semitones: 13 },
        { active: false, wrong: false, displayedKeymap: 'L', keymap: 'KeyL', note: 'B2', sharp: false, semitones: 14 },
        { active: false, wrong: false, displayedKeymap: 'Ç', keymap: 'Semicolon', note: 'C3', sharp: false, semitones: 15 },
        { active: false, wrong: false, displayedKeymap: '´', keymap: 'BracketLeft', note: 'CS3', sharp: true, semitones: 16 },
        { active: false, wrong: false, displayedKeymap: '~', keymap: 'Quote', note: 'D3', sharp: false, semitones: 17 },
        { active: false, wrong: false, displayedKeymap: '[', keymap: 'BracketRight', note: 'DS3', sharp: true, semitones: 18 },
        { active: false, wrong: false, displayedKeymap: ']', keymap: 'Backslash', note: 'E3', sharp: false, semitones: 19 },
        { active: false, wrong: false, displayedKeymap: 'Enter', keymap: 'Enter', note: 'F3', sharp: false, semitones: 20 },
        { active: false, wrong: false, displayedKeymap: '7', keymap: 'Numpad7', note: 'FS3', sharp: true, semitones: 21 },
        { active: false, wrong: false, displayedKeymap: '4', keymap: 'Numpad4', note: 'G3', sharp: false, semitones: 22 },
        { active: false, wrong: false, displayedKeymap: '8', keymap: 'Numpad8', note: 'GS3', sharp: true, semitones: 23 },
        { active: false, wrong: false, displayedKeymap: '5', keymap: 'Numpad5', note: 'A3', sharp: false, semitones: 24 },
        { active: false, wrong: false, displayedKeymap: '9', keymap: 'Numpad9', note: 'AS3', sharp: true, semitones: 25 },
        { active: false, wrong: false, displayedKeymap: '6', keymap: 'Numpad6', note: 'B3', sharp: false, semitones: 26 },
        { active: false, wrong: false, displayedKeymap: '+', keymap: 'NumpadAdd', note: 'C4', sharp: false, semitones: 27 }
      ] as Key[]
    }
  },

  created () {
    // setInterval(() => { this.fallingNotes.C2.startsAt -= 0.05 }, 10)
    // const barDuration = 10
    const beatsPerSecond = 2
    // setInterval(() => { this.playBeat() }, 1 / beatsPerSecond * 1000)
    setInterval(() => { this.currentGameTime += beatsPerSecond / 100 }, 1000 / 100)

    const begin = 3
    // this.fallingNotes.C2.push({ duration: 1, startsAt: 1 + begin, hitting: false })
    // this.fallingNotes.E2.push({ duration: 1, startsAt: 1 + begin, hitting: false })
    // this.fallingNotes.C2.push({ duration: 1, startsAt: 3 + begin, hitting: false })
    // this.fallingNotes.E2.push({ duration: 1, startsAt: 3 + begin, hitting: false })

    // this.fallingNotes.C2.push({ duration: 1, startsAt: 1 + begin, hitting: false })
    // this.fallingNotes.D2.push({ duration: 1, startsAt: 2 + begin, hitting: false })
    // this.fallingNotes.E2.push({ duration: 1, startsAt: 3 + begin, hitting: false })
    // this.fallingNotes.F2.push({ duration: 1, startsAt: 4 + begin, hitting: false })
    // this.fallingNotes.G2.push({ duration: 1, startsAt: 5 + begin, hitting: false })
    // this.fallingNotes.A2.push({ duration: 1, startsAt: 6 + begin, hitting: false })
    // this.fallingNotes.B2.push({ duration: 1, startsAt: 7 + begin, hitting: false })
    // this.fallingNotes.CS3.push({ duration: 1, startsAt: 1 + begin, hitting: false })

    // this.fallingNotes.C2.push({ duration: 2, startsAt: 1 + begin, hitting: false })
    // this.fallingNotes.E2.push({ duration: 2, startsAt: 1 + begin, hitting: false })
    // this.fallingNotes.G2.push({ duration: 2, startsAt: 1 + begin, hitting: false })
    // this.fallingNotes.B1.push({ duration: 2, startsAt: 3 + begin, hitting: false })
    // this.fallingNotes.D2.push({ duration: 2, startsAt: 3 + begin, hitting: false })
    // this.fallingNotes.F2.push({ duration: 2, startsAt: 3 + begin, hitting: false })
    // this.fallingNotes.A1.push({ duration: 2, startsAt: 5 + begin, hitting: false })
    // this.fallingNotes.CS2.push({ duration: 2, startsAt: 5 + begin, hitting: false })
    // this.fallingNotes.E2.push({ duration: 2, startsAt: 5 + begin, hitting: false })

    // this.fallingNotes.C2.push({ duration: 1, startsAt: 1 + begin, hitting: false })
    // this.fallingNotes.CS2.push({ duration: 1, startsAt: 2 + begin, hitting: false })
    // this.fallingNotes.D2.push({ duration: 1, startsAt: 3 + begin, hitting: false })
    // this.fallingNotes.DS2.push({ duration: 1, startsAt: 4 + begin, hitting: false })
    // this.fallingNotes.E2.push({ duration: 1, startsAt: 5 + begin, hitting: false })
    // this.fallingNotes.F2.push({ duration: 1, startsAt: 6 + begin, hitting: false })
    // this.fallingNotes.FS2.push({ duration: 1, startsAt: 7 + begin, hitting: false })
    // this.fallingNotes.G2.push({ duration: 1, startsAt: 8 + begin, hitting: false })
    // this.fallingNotes.GS2.push({ duration: 1, startsAt: 9 + begin, hitting: false })
    // this.fallingNotes.A2.push({ duration: 1, startsAt: 10 + begin, hitting: false })
    // this.fallingNotes.AS2.push({ duration: 1, startsAt: 11 + begin, hitting: false })
    // this.fallingNotes.B2.push({ duration: 1, startsAt: 12 + begin, hitting: false })
    // this.fallingNotes.C3.push({ duration: 1, startsAt: 13 + begin, hitting: false })
    // this.fallingNotes.CS3.push({ duration: 1, startsAt: 14 + begin, hitting: false })

    this.fallingNotes.B2.push({ duration: 1, startsAt: 1 + begin, hitting: false })
    this.fallingNotes.B2.push({ duration: 1, startsAt: 2 + begin, hitting: false })
    this.fallingNotes.B2.push({ duration: 2, startsAt: 3 + begin, hitting: false })
    this.fallingNotes.B2.push({ duration: 1, startsAt: 5 + begin, hitting: false })
    this.fallingNotes.B2.push({ duration: 1, startsAt: 6 + begin, hitting: false })
    this.fallingNotes.B2.push({ duration: 2, startsAt: 7 + begin, hitting: false })
    this.fallingNotes.B2.push({ duration: 1, startsAt: 9 + begin, hitting: false })
    this.fallingNotes.D3.push({ duration: 1, startsAt: 10 + begin, hitting: false })
    this.fallingNotes.G2.push({ duration: 1, startsAt: 11 + begin, hitting: false })
    this.fallingNotes.A2.push({ duration: 1, startsAt: 12 + begin, hitting: false })
    this.fallingNotes.B2.push({ duration: 4, startsAt: 13 + begin, hitting: false })
    this.fallingNotes.C3.push({ duration: 1, startsAt: 17 + begin, hitting: false })
    this.fallingNotes.C3.push({ duration: 1, startsAt: 18 + begin, hitting: false })
    this.fallingNotes.C3.push({ duration: 2, startsAt: 19 + begin, hitting: false })
    this.fallingNotes.B2.push({ duration: 1, startsAt: 21 + begin, hitting: false })
    this.fallingNotes.B2.push({ duration: 1, startsAt: 22 + begin, hitting: false })
    this.fallingNotes.B2.push({ duration: 2, startsAt: 23 + begin, hitting: false })
    this.fallingNotes.A2.push({ duration: 1, startsAt: 25 + begin, hitting: false })
    this.fallingNotes.B2.push({ duration: 1, startsAt: 26 + begin, hitting: false })
    this.fallingNotes.A2.push({ duration: 1, startsAt: 27 + begin, hitting: false })
    this.fallingNotes.B2.push({ duration: 1, startsAt: 28 + begin, hitting: false })
    this.fallingNotes.A2.push({ duration: 2, startsAt: 29 + begin, hitting: false })
    this.fallingNotes.D3.push({ duration: 2, startsAt: 31 + begin, hitting: false })

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
        const tolerance = 0.5
        const fallingNotes = this.fallingNotes[key.note]
        // .filter(({ hitting }) => !hitting)
        const delays = fallingNotes.map(({ startsAt }) => Math.abs(startsAt - this.currentGameTime))
        const index = indexOfMin(delays)
        const fallingNote = fallingNotes[index]
        // const timeDelay = Math.abs(fallingNote.startsAt - this.currentGameTime)
        if (fallingNote && !fallingNote.hitting && delays[index] < tolerance) {
          fallingNote.hitting = true
          key.wrong = false
          // console.log(fallingNote.startsAt)
          // if (Math.abs(fallingNote.startsAt - this.currentGameTime) < tolerance) {
          // }
        } else {
          key.wrong = true
        }
        this.play(key)
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

    classesForFallingNote (keyIndex: number, fallingNote: FallingNote) {
      return {
        // missed: fallingNote.missed,
        capleft: !this.keys[keyIndex]?.sharp && this.keys[keyIndex - 1]?.sharp && !this.keys[keyIndex + 1]?.sharp,
        capright: !this.keys[keyIndex]?.sharp && this.keys[keyIndex + 1]?.sharp && !this.keys[keyIndex - 1]?.sharp,
        capboth: !this.keys[keyIndex]?.sharp && this.keys[keyIndex + 1]?.sharp && this.keys[keyIndex - 1]?.sharp,
        hitting: fallingNote.hitting
      }
    },
    randomColor () {
      return '#' + Math.random().toString(16).slice(2, 8)
    },

    playBeat () {
      if (!this.audioContext || !this.gainControl) return

      const beatGain = this.audioContext.createGain()
      beatGain.connect(this.gainControl)

      const beatOscillator = this.audioContext.createOscillator()
      beatOscillator.connect(beatGain)
      beatOscillator.frequency.value = 5000

      const now = this.audioContext.currentTime
      const attackTime = 0.1
      const releaseTime = 0.1

      // beatOscillator
      beatOscillator.start()
      beatGain.gain.setValueAtTime(0, 0)
      beatGain.gain.linearRampToValueAtTime(1, now + attackTime)
      beatGain.gain.linearRampToValueAtTime(0, now + attackTime + releaseTime)
    },

    isPlaying (key: Key): boolean {
      return !!this.keyNodes[key.note]
    },

    play (key: Key) {
      if (!this.audioContext || !this.gainControl) return

      const fadeOut = this.audioContext.createGain()
      fadeOut.connect(this.gainControl)

      const oscillator = this.audioContext.createOscillator()
      oscillator.type = 'sine'
      oscillator.connect(fadeOut)

      const gain2 = this.audioContext.createGain()
      gain2.gain.setValueAtTime(0.5, 0)

      const oscillator2 = this.audioContext.createOscillator()
      oscillator2.type = 'triangle'
      oscillator2.connect(gain2)
      gain2.connect(fadeOut)

      const vibrato = this.audioContext.createOscillator()
      vibrato.frequency.setValueAtTime(4, 0)

      const vibratoGain = this.audioContext.createGain()
      vibratoGain.gain.setValueAtTime(2, 0)

      vibrato.connect(vibratoGain)
      vibratoGain.connect(oscillator.frequency)
      vibratoGain.connect(oscillator2.frequency)
      vibrato.start()

      this.keyNodes[key.note] = fadeOut

      const attackTime = 0.01
      const decayTime = 0.01
      const sustainTime = 0.5
      const releaseTime = 1

      const semitoneFactor = 2.0 ** (1 / 12)
      const firstNote = 440.0 * (2 ** (this.currentOctave - 2))
      const frequency = firstNote * semitoneFactor ** key.semitones

      oscillator.frequency.value = frequency
      oscillator2.frequency.value = frequency

      const now = this.audioContext.currentTime
      fadeOut.gain.setValueAtTime(0.1, 0)
      fadeOut.gain.linearRampToValueAtTime(1.5, now + attackTime)
      fadeOut.gain.linearRampToValueAtTime(0.7, now + attackTime + decayTime)
      fadeOut.gain.linearRampToValueAtTime(0.2, now + attackTime + decayTime + sustainTime)
      fadeOut.gain.linearRampToValueAtTime(0, now + attackTime + decayTime + sustainTime + releaseTime)
      oscillator.start()
      oscillator2.start()
    },

    stop (key: Key) {
      const fadeOut = this.keyNodes[key.note]
      if (!this.audioContext || !this.gainControl || !fadeOut) return

      this.keyNodes[key.note] = null
    }
  }
})
</script>

<style lang="sass">
$key-width: 50px
$sharp-key-width: 30px
$game-bg: #5af

*
  padding: 0
  margin: 0
  box-sizing: border-box

body
  overflow: hidden

#app
  font-family: Avenir, Helvetica, Arial, sans-serif
  -webkit-font-smoothing: antialiased
  -moz-osx-font-smoothing: grayscale
  text-align: center
  color: #2c3e50
  width: 100%
  height: 100vh
  display: flex
  flex-direction: column
  align-items: center
  background: $game-bg

.game
  display: flex
  flex-direction: column
  height: 100vh

.falling-notes
  display: flex
  width: 100%
  flex: 1
  height: 300px

.falling-note-placeholder
  width: $key-width
  height: 100%
  position: relative

.falling-note-placeholder.sharp
  background: #0003
  z-index: 1
  width: $sharp-key-width
  margin-left: calc($sharp-key-width / -2)
  margin-right: calc($sharp-key-width / -2)

.falling-note
  position: absolute
  width: 100%
  bottom: 0
  z-index: 2
  display: flex
  flex-direction: column

  > div
    border-radius: 10px
    background-color: #5fa
    width: 100%
    height: 100%

  &.hitting
    > div
      background-color: #aaf

  &.capleft
    align-items: flex-end

    > div
      $spacing: calc($sharp-key-width / 2)
      width: calc($key-width - $spacing)

  &.capright
    align-items: flex-start

    > div
      $spacing: calc($sharp-key-width / 2)
      width: calc($key-width - $spacing)

  &.capboth
    align-items: center
    > div
      $spacing: calc($sharp-key-width / 2)
      width: calc($key-width - $spacing)

  // &.capright
  //   $spacing: calc($sharp-key-width / 2)
  //   width: calc($key-width - $spacing)
  //   left: 0

.sharp .falling-note
  > div
    background-color: #3c8
    width: 100%

  &.hitting
    > div
      background-color: #77c

.keyboard
  // position: absolute
  display: flex
  justify-content: flex-start
  height: 300px
  border-radius: 10px

.keys
  display: flex

.key
  z-index: 4
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

  &.active.wrong
    background-color: #fcc

.key.sharp
  box-shadow: none
  z-index: 5
  width: $sharp-key-width
  height: 70%
  margin-left: calc($sharp-key-width / -2)
  margin-right: calc($sharp-key-width / -2)
  background-color: #222
  color: white

  &.active
    background-color: #55a

  &.active.wrong
    background-color: #a55

.below-keyboard
  position: relative
  // flex: 1
  height: 100px
  width: 100%

  > div
    z-index: 3
    position: absolute
    height: 100%
    width: 100%
    background-color: $game-bg

</style>
