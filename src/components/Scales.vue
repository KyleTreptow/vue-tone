<template>
  <div class="">
    <main>
      <h1>{{ msg }}</h1>
      <div class="block">
        <h2>Key + Octave + Mode</h2>
        <select v-model="activeKey">
          <option v-for="note in notes" :value="note" :key="note">{{ note }}</option>
        </select>
        <select v-model="octave">
          <option v-for="n in 5" :value="n" :key="n">{{ n }}</option>
        </select>
        <select v-model="activeMode">
          <option v-for="mode in modes" :value="mode" :key="mode">{{ mode }}</option>
        </select>
      </div>
      <div class="block">
        <button v-for="note in notesFromKey"
          :key="'note-'+note"
          class="note-item"
          :class="{
            active: modeNotes.includes(note),
            inactive: !modeNotes.includes(note),
            off: !showOffKeys,
            live: note == liveNote
          }"
          @click="startAudio(note)">{{ note }}</button>
      </div>
      <div class="block">
        <button class="seq-btn" @click="playScale(modeNotes)">
          Play Scale
        </button>
        <button class="seq-btn" @click="playScale(suffleNotes())">
          Play Shuffle
        </button>
        <button class="seq-btn" @click="playScale(randomNotes())">
          Play Random
        </button>
      </div>
    </main>
    <footer>
      <button class="foot-link" @click="showOffKeys = !showOffKeys">
        {{ showOffKeys ? 'Hide' : 'Show' }} Off Keys
      </button>
      <button class="foot-link" @click="log(synth)">
        Log Synth
      </button>
      <button class="foot-link" @click="randomize()">
        Randomize
      </button>
    </footer>
  </div>
</template>

<script>
import * as Tone from 'tone'
export default {
  name: 'HelloWorld',
  data(){
    return {
      synth: null,
      activeKey: 'C',
      octave: 4,
      activeMode: 'aeolian',
      notes: ['C', 'Db', 'D', 'Eb', 'E', 'F', 'Gb', 'G', 'Ab', 'A', 'Bb', 'B'],
      modes: ['ionian', 'dorian', 'phrygian', 'lydian', 'mixolydian', 'aeolian', 'locrian'],
      modeScales: {
        ionian:     [2, 2, 1, 2, 2, 2, 1],
        dorian:     [2, 1, 2, 2, 2, 1, 2],
        phrygian:   [1, 2, 2, 2, 1, 2, 2],
        lydian:     [2, 2, 2, 1, 2, 2, 1],
        mixolydian: [2, 2, 1, 2, 2, 1, 2],
        aeolian:    [2, 1, 2, 2, 1, 2, 2],
        locrian:    [1, 2, 2, 1, 2, 2, 2]
      },
      showOffKeys: true,
      playing: false,
      synthPart: null,
      liveNote: null
    }
  },
  props: {
    msg: String
  },
  mounted() {
    this.synth = new Tone.Synth().toDestination()
  },
  computed: {
    notesFromKey(){
      let noteList = this.notes
      let keyIndex = noteList.indexOf(this.activeKey)
      noteList = noteList.concat(this.notes)
      noteList = noteList.map((n, i) => {
        return n + (i < 12 ? this.octave : this.octave +1)
      })
      return noteList.slice(keyIndex, keyIndex+13)
    },
    modeNotes(){
      let notes = this.notesFromKey
      let modeSteps = this.modeScales[this.activeMode]
      let modeList = []
      let indexMod = 0
      modeList.push(notes[0])
      for(let i = 0; i < modeSteps.length; i++){
        indexMod += modeSteps[i]
        modeList.push(notes[indexMod])
      }
      return modeList
    }
  },
  methods: {
    log(data){
      console.log(data)
    },
    startAudio(note) {
      this.synth.triggerAttackRelease(note, "8n")
    },
    randomize(){
      let octaveNum = 5
      let notesNum = this.notes.length
      let modesNum = this.modes.length
      this.octave = Math.floor(Math.random() * octaveNum) + 1
      this.activeKey = this.notes[Math.floor(Math.random() * notesNum)]
      this.activeMode = this.modes[Math.floor(Math.random() * modesNum)]
    },
    playScale(notes){
      let that = this
      if (Tone.context.state !== 'running') {
        Tone.context.resume()
      }
      let s = this.synth
      let n = notes
      this.synthPart = new Tone.Sequence(
        function(time, note) {
          s.triggerAttackRelease(note, "10hz", time)
          that.liveNote = note
        },
        n,
        "8n"
      );
      this.synthPart.start()
      if (!this.playing) {
        Tone.Transport.start()
        this.playing = true
      } else {
        Tone.Transport.stop()
        Tone.Transport.cancel()
        this.playing = false
        this.liveNote = null
      }
    },
    suffleNotes(){
      let array = [...this.modeNotes]
      for (let i = array.length - 1; i > 0; i--) {
        let j = Math.floor(Math.random() * (i + 1));
        let temp = array[i];
        array[i] = array[j];
        array[j] = temp;
      }
      return array
    },
    randomNotes(){
      let notes = [...this.modeNotes]
      let array = []
      for (let i = 0; i < 8; i++) {
        let rand = Math.floor(Math.random() * 9)
        if(rand == 9){ array.push(null) } // push rest (null) note
        else { array.push(notes[rand]) } // push random note from mode
      }
      return array
    }
  }
}
</script>

<style lang="sass" scoped>
  h1
    color: #2c3e50
    margin: 0 0 10px 0
  h2
    font-size: 14px
    font-weight: 900
    text-transform: uppercase
    margin: 0 0 30px 0
  ul
    list-style-type: none
    padding: 0
  li
    display: inline-block
    margin: 0 10px
  a
    color: #42b983
  .block
    margin-bottom: 20px
  button
    border: solid 1px #999
    border-radius: 3px
    background-color: #eee
    border-color: darken(#eee, 20%)
    &:hover,
    &:focus,
    &:active
      background-color: #ddd
      border-color: darken(#ddd, 20%)
      cursor: pointer
    &.active
      background-color: #42b983
      border-color: darken(#42b983, 10%)
      color: #fff
      &:hover,
      &:focus,
      &:active
        background-color: darken(#42b983, 5%)
        border-color: darken(#42b983, 15%)
  .note-item
    display: inline-block
    padding: 20px 10px
    font-weight: 600
    border: solid 1px #ddd
    color: #999
    &.active
      background-color: #42b983
      color: #fff
      &.live
        background-color: #2c3e50
        border-color: darken(#2c3e50, 5%)
    &.inactive.off
      display: none
  .seq-btn
    background-color: #2c3e50
    border: solid 1px darken(#2c3e50, 5%)
    color: #fff
    padding: 10px 30px
    border-radius: 6px
    margin-right: 10px
    &:last-child
      margin-right: 0
    &:hover,
    &:focus,
    &:active
      background-color: darken(#2c3e50, 5%)
      border: solid 1px darken(#2c3e50, 10%)
      color: #fff
  footer
    background: #eee
    padding: 20px
    position: absolute
    bottom: 0
    left: 0
    width: 100%
  .foot-link
    display: inline-block
    margin-right: 10px
    color: #2c3e50
    background-color: transparent
    border: solid 1px #2c3e50
    border-radius: 3px
    padding: 5px 20px
    &:last-child
      margin-right: 0
    &:hover,
    &:focus,
    &:active
      color: #fff
      background-color: #42b983
      border: solid 1px darken(#42b983, 10%)
</style>
