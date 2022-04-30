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
        &nbsp; <span>Phrase Length:</span> &nbsp;
        <select v-model="phraseLength">
          <option value="2" key="2">2</option>
          <option value="4" key="4">4</option>
          <option value="8" key="8">8</option>
          <option value="16" key="16">16</option>
          <option value="32" key="32">32</option>
          <option value="64" key="64">64</option>
        </select>
        &nbsp; <span>Note Length:</span> &nbsp;
        <select v-model="noteLength">
          <option value="1n" key="1n">1n</option>
          <option value="2n" key="2n">2n</option>
          <option value="4n" key="4n">4n</option>
          <option value="4t" key="4t">4t</option>
          <option value="8n" key="8n">8n</option>
          <option value="8t" key="8t">8t</option>
          <option value="16n" key="16n">16n</option>
          <option value="16t" key="16t">16t</option>
        </select>
      </div>
      <div>
        <span>Vol:</span>
        <input type="range" min="-24" max="0"
         v-model="volume" @change="synth.volume.value = volume" >
         {{ volume }} DB
      </div>
      <div class="block">
        <button v-for="note in notesFromKey"
          :key="'note-'+note"
          class="note-item"
          :class="{
            active: modeNotes.includes(note),
            inactive: !modeNotes.includes(note),
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
      noteLength: "8n",
      displaySeqArr: false,
      seqArray: [],
      liveNote: null,
      synthWaveForm: 'triangle',
      waveForms: ['sine', 'triangle', 'amtriangle', 'square', 'sawtooth'],
      effects: {
        fbDelay: null,
        reverb: null,
        chorus: null,
        distortion: null,
        phaser: null
      },
      volume: -12
    }
  },
  mounted() {
    // Init Synths & Effects
    this.effects.fbDelay = new Tone.FeedbackDelay({ "wet": 1, "delayTime": "8n", "feedback": 0.35 })
    this.effects.reverb = new Tone.Reverb({ "wet": 0.4, "decay": 1.5, "preDelay": 0.01 })
    this.synth = new Tone.Synth({ volume: this.volume })
    // Route
    this.route()
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
    route(){
      // == DISCONNECTS
      // this.effects.fbDelay.disconnect()
      // this.effects.reverb.disconnect()
      // this.synth.disconnect()

      // == REVERB ONLY
      // this.synth.connect(this.effects.reverb)
      // this.effects.reverb.toDestination()

      // == DELAY ONLY
      this.synth.connect(this.effects.fbDelay)
      this.effects.fbDelay.toDestination()

      // == DELAY && REVERB
      // this.synth.connect(this.effects.fbDelay)
      // this.effects.fbDelay.connect(this.effects.reverb)
      // this.effects.reverb.toDestination()

      // == SYNTH DIRECT OUT CLEAN
      // this.synth.toDestination()
    },
    startAudio(note) {
      this.synth.triggerAttackRelease(note, "8n")
    },
    createSequence(){ // creates a Tone sequence from pattern of notes
      let that = this
      let s = this.synth
      let notes = this.generatePattern()
      let synthPart = new Tone.Sequence(
        function(time, note) {
          s.triggerAttackRelease(note, "10hz", time) // note, release (10hz or 16n?), time
          that.liveNote = note
        },
        notes, this.noteLength
      );
      return synthPart
    },
    generatePattern(){ // generates pattern of notes from mode
      let notes = [...this.modeNotes]
      let seq = []
      for (let i = 0; i < this.phraseLength; i++) { // phraseLength: 2, 4, 8, 16, 32, 64 etc.
        let rand = Math.floor(Math.random() * 9)
        if(rand == 9){ seq.push(null) } // push rest (null) note
        else { seq.push(notes[rand]) } // push random note from mode
      }
      this.seqArray = seq // sets display-able array
      return seq
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
      &:active
        background-color: #2c3e50
        border-color: darken(#2c3e50, 5%)
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
