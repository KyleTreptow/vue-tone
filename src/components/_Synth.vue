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

      <button type="button" @click="paramsActive = !paramsActive">Synth Parameters</button>
      <div class="params" v-if="paramsActive">
        <main>
          <div class="osc">
            <h4>Oscillator</h4>
            <div>
              <span>Vol:</span>
              <input type="range" min="-24" max="12"
               v-model="params.volume">
               {{ params.volume }} DB
            </div>
            <div>
              <span>Portamento:</span>
              <input type="range" min="0.00" max="0.10" step="0.01"
               v-model="params.portamento">
               {{ params.portamento }}
            </div>
            <div>
              <select v-model="params.oscillator.type">
                <option v-for="wf in waveForms" :value="wf" :key="wf">{{ wf }}</option>
              </select>
            </div>
            <div>
              <span>Phase:</span>
              <input type="range" min="0" max="360"
               v-model="params.oscillator.phase">
               {{ params.oscillator.phase }} deg
            </div>
          </div>
          <div class="env">
            <h4>Envelope</h4>
            <div class="">
              <span>A:</span>
              <input type="range" min="0.01" max="2.00" step="0.01"
               v-model="params.envelope.attack">
               {{ params.envelope.attack }}
            </div>
            <div class="">
              <span>D:</span>
              <input type="range" min="0.01" max="2.00" step="0.01"
               v-model="params.envelope.decay">
               {{ params.envelope.decay }}
            </div>
            <div class="">
              <span>S:</span>
              <input type="range" min="0" max="1" step="0.01"
               v-model="params.envelope.sustain">
               {{ params.envelope.sustain }}
            </div>
            <div class="">
              <span>R:</span>
              <input type="range" min="0.01" max="2.00" step="0.01"
               v-model="params.envelope.release">
               {{ params.envelope.release }}
            </div>
          </div>
        </main>
        <footer>
          <div class="effectlist">
            <div class="inactive">
              <button v-for="e in inactiveEffects" :key="'effect_'+e"
              type="button"
              @click="activateEffect(e)">
                {{ e }}
              </button>
            </div>
            <div class="active">
              <button v-for="e in activeEffects" :key="'active_effect_'+e"
              type="button"
              @click="deactivateEffect(e)">
                {{ e }}
              </button>
            </div>
          </div>
          <!-- <div class="">
            <button type="button" @click="disconnect()">Disconnect</button>
            <button type="button" @click="connect()">Connect</button>
          </div> -->
          <br>
          <button type="button" @click="log(synth)">Log Synth</button>
        </footer>
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
      waveForms: ['sine', 'triangle', 'square', 'sawtooth'],
      // synth parameters
      paramsActive: true,
      params: {
        "volume": 0,
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
      // effects units
      effects: {
        fbDelay: null,
        reverb: null,
        chorus: null,
        distortion: null,
        phaser: null,
        bitcrusher: null,
        filter: null
      },
      effectsList: ['fbDelay', 'reverb', 'chorus', 'distortion', 'phaser', 'bitcrusher', 'filter'],
      activeEffects: []
    }
  },
  mounted() {
    // Init Synth
    this.synth = new Tone.Synth(this.params)
    // Init Effects
    this.effects.fbDelay = new Tone.FeedbackDelay({ "wet": 1, "delayTime": "8n", "feedback": 0.35 })
    this.effects.reverb = new Tone.Reverb({ "wet": 0.4, "decay": 1.5, "preDelay": 0.01 })
    this.effects.chorus = new Tone.Chorus({ "frequency": 1.5, "delayTime": 3.5, "depth": 0.7, "type": 'sine', "spread": 180 })
    this.effects.distortion = new Tone.Distortion({ "distortion": 2, "oversample": "none" })
    this.effects.phaser = new Tone.Phaser({ "frequency": 0.5, "octaves": 3, "stages": 10, "Q": 10, "baseFrequency": 350 })
    this.effects.bitcrusher = new Tone.BitCrusher({ "bits": 3 })
    this.effects.filter = new Tone.Filter({ "type" : 'lowpass', "frequency" : 5000, "rolloff" : -24, "Q" : 1, "gain" : 0 })
    // Route synth to destination clean
    this.synth.toDestination()
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
    },
    inactiveEffects(){
      return this.effectsList.filter(e => !this.activeEffects.includes(e))
    }
  },
  methods: {
    disconnect(){
      for(let e of this.effectsList){
        this.effects[e].disconnect()
      }
      this.synth.disconnect()
      console.log('Disconnect All Effects')
    },
    connect(){
      // get list of active effects, create array of effects
      let fx = this.activeEffects.map(x => this.effects[x])
      fx.push(Tone.Destination)
      this.synth.chain(...fx)
      console.log(fx)
    },
    activateEffect(item){
      this.activeEffects.push(item)
      this.disconnect()
      this.connect()
    },
    deactivateEffect(item){
      let index = this.activeEffects.indexOf(item)
      if (index !== -1) {
        this.activeEffects.splice(index, 1);
      }
      this.disconnect()
      this.connect()
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
    },
    log(data){
      console.log(data)
    }
  },
  watch: {
    'params.volume'(){
      this.synth.volume.value = this.params.volume
    },
    'params.portamento'(){
      this.synth.portamento = this.params.portamento
    },
    'params.oscillator.type'(){
      this.synth.oscillator.type = this.params.oscillator.type
    },
    'params.oscillator.phase'(){
      this.synth.oscillator.phase = this.params.oscillator.phase
    },
    'params.envelope.attack'(){
      this.synth.envelope.attack = this.params.envelope.attack
    },
    'params.envelope.decay'(){
      this.synth.envelope.decay = this.params.envelope.decay
    },
    'params.envelope.sustain'(){
      this.synth.envelope.sustain = this.params.envelope.sustain
    },
    'params.envelope.release'(){
      this.synth.envelope.release = this.params.envelope.release
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
    main
      display: grid
      grid-template-columns: 1fr 1fr
    footer
      display: block
      padding-top: 10px

  .effectlist
    margin-bottom: 10px
    .inactive
      button
        border: solid 1px #ddd
        background-color: #eee
        color: #333
    .active
      button
        border: solid 1px darken(#42b983, 10%)
        background-color: #42b983
        color: #fff

</style>
