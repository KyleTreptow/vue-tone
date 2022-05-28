<template>
  <div class="">
    <section>

      <div class="block">
        <span><b>{{ name }}: &nbsp; </b></span>
        <select v-model="octave">
          <option v-for="n in 5" :value="n" :key="n">{{ n }}</option>
        </select>
        &nbsp; <span>Phrase Length:</span> &nbsp;
        <select v-model="phraseLength">
          <option v-for="n in phraseLengthList" :value="n" :key="n">{{ n }}</option>
        </select>
        &nbsp; <span>Note Length:</span> &nbsp;
        <select v-model="noteLength">
          <option v-for="n in noteLengthList" :value="n" :key="n">{{ n }}</option>
        </select>
      </div>

      <div class="block">
        <button v-for="note in octaveTones"
          :key="'note-'+note"
          class="note-item"
          :class="{
            active: modeTones.includes(note),
            inactive: !modeTones.includes(note),
            live: note == liveNote
          }"
          @click="startAudio(note)">{{ note }}</button>
      </div>

      <div class="testing">
        <div class="">
          <h5>Octave from Root</h5>
          {{ getOctaveFromRoot() }}
        </div>
        <div class="">
          <h5>Mode Tones</h5>
          {{ reduceOctaveToMode() }}
        </div>
        <div class="">
          <h5>Mult Oct</h5>
          {{ getNotesFromMode({range: 2}) }}
        </div>
        <div class="">
          <h5>SEQUENCE</h5>
          {{ seqArray }}
        </div>
      </div>



      <button type="button" @click="createSequence()">Create Sequence</button>

      <button type="button" @click="paramsActive = !paramsActive">Synth Parameters</button>

      <div class="params" v-if="paramsActive">
        <main>
          <div class="param">
            <h4 class="param__header">Oscillator</h4>
            <div class="control">
              <h4>
                <span>Volume:</span>
                <span>{{ params.volume }} dB</span>
              </h4>
              <input type="range" min="-24" max="24"
               v-model="params.volume"
               @change="synth.volume.value = params.volume">
            </div>

            <div class="control">
              <h4>
                <span>Portamento:</span>
                <span>{{ params.portamento }}</span>
              </h4>
              <input type="range" min="0.00" max="0.10" step="0.01"
               v-model="params.portamento"
               @change="synth.portamento = params.portamento">
            </div>

            <div class="control">
              <h4>
                <span>Phase:</span>
                <span>{{ params.oscillator.phase }} deg</span>
              </h4>
              <input type="range" min="0" max="360"
               v-model="params.oscillator.phase">
            </div>

            <div class="control control--select">
              <h4>
                <span>Waveform:</span>
              </h4>
              <select v-model="params.oscillator.type">
                <option v-for="wf in waveForms" :value="wf" :key="wf">{{ wf }}</option>
              </select>
            </div>

          </div>
          <div class="param">
            <h4>Envelope</h4>

            <div class="control">
              <h4>
                <span>Attack: </span>
                <span>{{ params.envelope.attack }}</span>
              </h4>
              <input type="range" min="0.01" max="2.00" step="0.01"
               v-model="params.envelope.attack">
            </div>

            <div class="control">
              <h4>
                <span>Decay: </span>
                <span>{{ params.envelope.decay }}</span>
              </h4>
              <input type="range" min="0.01" max="2.00" step="0.01"
               v-model="params.envelope.decay">
            </div>

            <div class="control">
              <h4>
                <span>Sustain:</span>
                <span>{{ params.envelope.sustain }}</span>
              </h4>
              <input type="range" min="0" max="1" step="0.01"
               v-model="params.envelope.sustain">
            </div>

            <div class="control">
              <h4>
                <span>Release:</span>
                <span>{{ params.envelope.release }}</span>
              </h4>
              <input type="range" min="0.01" max="2.00" step="0.01"
               v-model="params.envelope.release">
            </div>

          </div>
        </main>
        <footer>
          <!-- <button type="button" @click="log(synth)">Log Synth</button> -->
        </footer>
      </div>

  </section>
  </div>
</template>

<script>
import * as Tone from 'tone'
export default {
  name: 'Rack',
  props: ['notes', 'modes', 'scales'],
  data(){
    return {
      name: 'Rack',
      synth: null,
      activeKey: 'C',
      octave: 3,
      activeMode: 'aeolian',
      phraseLengthList: [2, 4, 8, 16, 32, 64],
      phraseLength: 8,
      noteLengthList: ['1n', '2n', '4n', '8n', '16n'],
      noteLength: "8n",
      liveNote: null,
      waveForms: ['sine', 'triangle', 'square', 'sawtooth'],
      seqArray: [],
      paramsActive: false,
      params: {
        "volume": -10,
        "detune": 0,
        "portamento": 0.0,
          "envelope": {
            "attack": 0.05,
            "attackCurve": "exponential",
            "decay": 0.2,
            "decayCurve": "exponential",
            "release": 1.5,
            "releaseCurve": "exponential",
            "sustain": 0.2
          },
          "oscillator": {
            "phase": 0,
            "type": 'square'
          }
      },
      curveTypes: ['linear', 'exponential', 'sine', 'cosine', 'bounce', 'ripple', 'step'],
      effects: {
        delay: null,
        reverb: null,
        chorus: null,
        distortion: null,
        phaser: null,
        bitcrusher: null,
        filter: null
      },
      modSources: {
        freqLFO: null
      }
    }
  },
  mounted() {
    this.synth = new Tone.Synth(this.params)
    this.effects.delay = new Tone.FeedbackDelay({
      "wet": 0.6,
      "delayTime": "8n",
      "feedback": 0.45
    })
    this.effects.filter = new Tone.Filter({
      "type" : 'lowpass',
      "frequency" : 6500,
      "rolloff" : -24,
      "Q" : 1,
      "gain" : 2
    })
    this.effects.reverb = new Tone.Reverb({
      "wet": 0.22,
      "decay": 1.5,
      "preDelay": 0.01
    })
    this.freqLFO = new Tone.LFO({
      type: 'sine',
      min: 0,
      max: 10000,
      frequency: '16n'
    })
    // this.freqLFO.start()
    // this.freqLFO.connect(this.effects.filter.frequency)
    this.synth.chain(this.effects.delay, this.effects.reverb, this.effects.filter, Tone.Destination)

    // Ramp Values
    // this.effects.delay.wet.rampTo(1, 3)

  },
  computed: {
    octaveTones(){ // 12 tones from root with Octave #s
      let noteList = this.notes
      noteList = noteList.concat(this.notes)
      let keyIndex = noteList.indexOf(this.activeKey)
      noteList = noteList.map((n, i) => {
        return n + (i < 12 ? this.octave : this.octave +1)
      })
      return noteList.slice(keyIndex, keyIndex+12)
    },
    modeTones(){
      let notes = this.octaveTones
      let modeSteps = this.scales[this.activeMode]
      let modeList = []
      let indexMod = 0
      modeList.push(notes[0])
      for(let i = 1; i < modeSteps.length; i++){
        indexMod += modeSteps[i]
        modeList.push(notes[indexMod])
      }
      return modeList
    }
  },
  methods: {
    // ROUTING
    disconnect(){
      // console.log(this.effects)
      for(let e of this.effectsList){
        if(this.effects.length){
          this.effects[e].disconnect()
        }
      }
      this.synth.disconnect()
    },
    connect(){
      let fx = this.activeEffects.map(x => this.effects[x])
      fx.push(Tone.Destination)
      this.synth.chain(...fx)
      // console.log(fx)
    },
    activateEffect(item){
      this.activeEffects.push(item)
      this.disconnect()
      this.connect()
    },
    deactivateEffect(item){
      let e = this.displayEffectParam
      this.displayEffectParam = (e == item) ? null : e
      let index = this.activeEffects.indexOf(item)
      if (index !== -1) {
        this.activeEffects.splice(index, 1);
      }
      this.disconnect()
      this.connect()
    },
    // MUSIC GEN
    startAudio(note) {
      this.synth.triggerAttackRelease(note, "4n")
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
        notes, "8n"
      );
      return synthPart
    },
    // Pattern Gen
    getOctaveFromC({base = this.octave, range = 1, nums = true} = {}){
      let noteList = []
      console.log(base, range, nums)
      for(let i = 0; i < range; i++){
        noteList = noteList.concat(this.notes)
      }
      noteList = noteList.map((n, i) => {
        return n + (i < 12 ? this.octave : this.octave +1)
      })
      return noteList
    },
    getOctaveFromRoot({base = this.octave, range = 1, nums = true} = {}){
      const notes = this.getOctaveFromC({ 'base': base, 'range': range + 1, 'nums': nums })
      let keyIndex = notes.indexOf(this.activeKey + base)
      return notes.slice(keyIndex, keyIndex + (12 * range))
    },
    reduceOctaveToMode({base = this.octave} = {}){
      const notes = this.getOctaveFromRoot({'base': base})
      let modeSteps = this.scales[this.activeMode]
      let modeList = []
      let modeIndex = 0
      modeList.push(notes[0])
      for(let i = 1; i < (modeSteps.length); i++){
        modeIndex = modeIndex + modeSteps[i]
        modeList.push(notes[modeIndex])
      }
      return modeList
    },
    getNotesFromMode({range = 1} = {}){
      let notes = []
      for(let i = 0; i < range; i++){
        let octaveNotes = this.reduceOctaveToMode({'base': this.octave + i})
        notes.push(octaveNotes)
      }
      return notes.reduce((acc, curVal) => acc.concat(curVal), [])
    },
    generatePattern(){
      let notes = this.getNotesFromMode()
      // notes.push(null)
      let seq = []
      for (let i = 0; i < this.phraseLength; i++) { // phraseLength: 2, 4, 8, 16, 32, 64 etc.
        let nest = this.rand(3)
        if(nest == 0 || nest == 1){ // Single Note
          seq.push(this.randNote(notes))
        } else { // Nested Array
          let seq2 = []
          for (let j = 0; j < 2; j++){
            seq2.push(this.randNote(notes))
          }
          seq.push(seq2)
        }
      }
      this.seqArray = seq
      return seq
    },
    rand(max){
      let rnum = Math.floor(Math.random() * (max ? max : 10))
      return rnum
    },
    randNote(notes){
      return notes[this.rand(notes.length)]
    },
    genMotif(){
      // note length pattern: 1n, 2n, 4n, 8n, 16n
      // note intervals: [1 (root), 5] [ 2, 7, 3] [4, 6]
    },
    // DEV
    log(data){
      console.log(data)
    }
  },
  watch: {
    params: {
       handler(){
         // this.synth.volume.value = this.params.volume
         // this.synth.portamento = this.params.portamento

         this.synth.oscillator.type = this.params.oscillator.type
         this.synth.oscillator.phase = this.params.oscillator.phase

         this.synth.envelope.attack = this.params.envelope.attack
         this.synth.envelope.decay = this.params.envelope.decay
         this.synth.envelope.sustain = this.params.envelope.sustain
         this.synth.envelope.release = this.params.envelope.release
       },
       deep: true
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
    padding: 10px 5px
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

  .params
    border: solid 1px #eee
    background-color: #fafafa
    border-radius: 3px
    display: block
    width: 600px
    max-width: 100%
    margin: auto
    h4
      margin: 0
    main
      display: grid
      grid-template-columns: 1fr 1fr
    footer
      display: block

  .param
    padding-top: 10px
    > h4
      margin: 0 0 10px 0

  .effectlist
    margin-bottom: 10px
    .inactive
      margin-bottom: 10px
      button
        border: solid 1px #ddd
        background-color: #eee
        color: #333
    .active
      span
        display: inline-block
        font-size: 0
        button
          border: solid 1px darken(#42b983, 10%)
          background-color: #42b983
          color: #fff
          display: inline-block
          appearance: none
          font-size: 14px
          border-radius: 0
          &:first-child
            border-top-left-radius: 3px
            border-bottom-left-radius: 3px
          &:last-child
            border-top-right-radius: 3px
            border-bottom-right-radius: 3px
            background-color: #2c3e50
            border-color: #2c3e50

  // CONTROL
  .control
    display: block
    padding: 10px 20px
    border: dotted 1px #ddd
    h4
      font-weight: bold
      font-size: 12px
      text-align: left
      margin-bottom: 6px
      span
        display: inline-block
        &:first-child
          text-transform: uppercase
        &:last-child
          padding-left: 10px
    input
      &[type="range"]
        display: block
        width: 100%
        height: 6px
    &--select
      display: grid
      grid-template-columns: 2fr 3fr
      h4,
      select
        line-height: 30px
      h4
        margin-bottom: 0
      select
        padding: 0 8px

  .testing
    h5
      margin: 0 0 6px 0

</style>
