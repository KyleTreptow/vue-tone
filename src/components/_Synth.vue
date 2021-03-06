<template>
  <div class="">
    <section>
      <div class="block">
        <span><b>{{ name }}: &nbsp; </b></span>
        <select v-model="activeKey">
          <option v-for="note in notes" :value="note" :key="note">{{ note }}</option>
        </select>
        <select v-model="octave">
          <option v-for="n in 5" :value="n" :key="n">{{ n }}</option>
        </select>
        <select v-model="activeMode">
          <option v-for="mode in modes" :value="mode" :key="mode">{{ mode }}</option>
        </select>
        <select v-model="synthWaveForm">
          <option v-for="wf in waveForms" :value="wf" :key="wf">{{ wf }}</option>
        </select>
        &nbsp;
        <span>Phrase Length:</span>
        &nbsp;
        <select v-model="phraseLength">
          <option value="2" key="2">2</option>
          <option value="4" key="4">4</option>
          <option value="8" key="8">8</option>
          <option value="16" key="16">16</option>
          <option value="32" key="32">32</option>
          <option value="64" key="64">64</option>
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
        <div>
          <button type="button" @click="displaySeqArr = !displaySeqArr">Display Sequence Notes</button>
        </div>
        <span class="seq-array" v-if="displaySeqArr">{{ seqArray }}</span>
      </div>
      <!-- <div class="block">
        <button class="seq-btn" @click="playSequence(modeNotes)">
          Play Scale
        </button>
        <button class="seq-btn" @click="playSequence(suffleNotes())">
          Play Shuffle
        </button>
        <button class="seq-btn" @click="playRandomSequence()">
          Play Random
        </button>
      </div>

    <div>
      <button class="foot-link" @click="showOffKeys = !showOffKeys">
        {{ showOffKeys ? 'Hide' : 'Show' }} Off Keys
      </button>
      <button class="foot-link" @click="log(synth)">
        Log Synth
      </button>
      <button class="foot-link" @click="randomize()">
        Randomize
      </button>
    </div> -->
  </section>
  </div>
</template>

<script>
import * as Tone from 'tone'
export default {
  name: 'Synth',
  props: ['notes', 'modes', 'scales'],
  data(){
    return {
      name: 'Synth',
      synth: null,
      activeKey: 'C',
      octave: 4,
      activeMode: 'aeolian',
      phraseLength: 32,
      showOffKeys: true,
      playing: false,
      synthPart: null,
      displaySeqArr: false,
      seqArray: [],
      liveNote: null,
      synthWaveForm: 'triangle',
      waveForms: ['sine', 'triangle', 'square', 'sawtooth'],
      effects: {
        fbDelay: null,
        reverb: null,
        chorus: null,
        distortion: null,
        phaser: null
      }
    }
  },
  mounted() {
    // Init Synths & Effects
    this.effects.fbDelay = new Tone.FeedbackDelay("8n", 0.35).toDestination();
    this.synth = new Tone.Synth().connect(this.effects.fbDelay);
    // this.synth = new Tone.Synth().toDestination()
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
      let modeSteps = this.scales[this.activeMode]
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
    playSequence(notes, d = "8n"){
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
        n, d
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
    playRandomSequence(duration){
      let d = duration || "8n"
      let seq = this.randomNotes()
      this.seqArray = seq
      this.playSequence(seq, d)
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
      let pl = this.phraseLength
      for (let i = 0; i < pl; i++) { // phraseLength: 2, 4, 8, 16, 32, 64 etc.
        let rand = Math.floor(Math.random() * 9)
        if(rand == 9){ array.push(null) } // push rest (null) note
        else { array.push(notes[rand]) } // push random note from mode
      }
      return array
    }
  },
  watch: {
    synthWaveForm() {
      this.synth.oscillator.type = this.synthWaveForm
    }
  }
}
</script>

<style lang="sass" scoped>
  section
    background: #fff
    padding: 6px 20px 6px 20px
    margin-bottom: 10px
    border-radius: 6px
    border: solid 1px #ddd
  ul
    list-style-type: none
    padding: 0
  li
    display: inline-block
    margin: 0 10px
  a
    color: #42b983
  .block
    margin-bottom: 6px
    &:last-child
      margin-bottom: 0
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
  .seq-array
    font-size: 12px
    color: #999
</style>
