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
          <option value="8n" key="8n">8n</option>
          <option value="16n" key="16n">16n</option>
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
          <div class="param">
            <h4 class="param__header">Oscillator</h4>

            <div class="control">
              <h4>
                <span>Volume:</span>
                <span>{{ params.volume }} dB</span>
              </h4>
              <input type="range" min="-12" max="12"
               v-model="params.volume">
            </div>

            <div class="control">
              <h4>
                <span>Portamento:</span>
                <span>{{ params.portamento }}</span>
              </h4>
              <input type="range" min="0.00" max="0.10" step="0.01"
               v-model="params.portamento">
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
          <h4>Effects</h4>
          <div class="effectlist">
            <div class="inactive">
              <button v-for="e in inactiveEffects" :key="'effect_'+e"
              type="button" @click="activateEffect(e)">
                {{ e }} <span>+</span>
              </button>
            </div>
            <div class="active">
              <button v-for="e in activeEffects" :key="'active_effect_'+e"
              type="button" @click="deactivateEffect(e)">
                {{ e }}
              </button>
            </div>
          </div>
          <div class="effect-params">
            <Phaser @init="initPhaser"/>
          </div>
          <button type="button" @click="log(synth)">Log Synth</button>
        </footer>
      </div>

  </section>
  </div>
</template>

<script>
import * as Tone from 'tone'
import Phaser from './effect/phaser.vue'
export default {
  name: 'Synth',
  components: { Phaser },
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
      activeEffects: [],
      // filter params
      filterParams: {
        "type" : 'lowpass',
        "frequency" : 12000,
        "rolloff" : -24,
        "Q" : 1,
        "gain" : 2
      },
      // delay params
      delayParams: {
        "wet": 0.6,
        "delayTime": "16n",
        "feedback": 0.75
      },
      // reverb params
      reverbParams: {
        "wet": 0.4,
        "decay": 1.5,
        "preDelay": 0.01
      },
      // chorus params
      chorusParams: {
        "frequency": 1.75, // 0 - 20hz? or 0-20k??
        "delayTime": 3.5,
        "depth": 0.7,
        "type": 'sine',
        "spread": 180
      },
      // distortion params
      distortionParams: {
        "distortion": 0.5,
        "oversample": 'none'
      },
      // bitcrusher params
      bitcrusherParams: {
        "bits": 5
      }

    }
  },
  mounted() {
    // Init Synth
    this.synth = new Tone.Synth(this.params)
    // Init Effects
    this.effects.fbDelay = new Tone.FeedbackDelay(this.delayParams)
      // this.effects.fbDelay.wet.rampTo(1, 3)
    this.effects.reverb = new Tone.Reverb(this.reverbParams)
    this.effects.chorus = new Tone.Chorus(this.chorusParams)
    this.effects.distortion = new Tone.Distortion(this.distortionParams)
    // this.effects.phaser = new Tone.Phaser(this.phaserParams)
    this.effects.bitcrusher = new Tone.BitCrusher(this.bitcrusherParams)
    this.effects.filter = new Tone.Filter(this.filterParams)
    // Route synth to destination clean
    this.synth.toDestination()

    // LFO Test
    // this.synth.chain(this.effects.filter, this.effects.fbDelay, Tone.Destination)

    // const lfo = new Tone.LFO({
    //   min: 0,
    //   max: 1,
    //   frequency: '1n'
    // })
    // lfo.start()
    // lfo.connect(this.effects.phaser.wet)


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
    // ROUTING
    disconnect(){
      for(let e of this.effectsList){
        this.effects[e].disconnect()
      }
      this.synth.disconnect()
      console.log('Disconnect All Effects')
    },
    connect(){
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
    // EFFECT MODULE
    initPhaser(module){
      this.effects.phaser = module
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
    // DEV
    log(data){
      console.log(data)
    }
  },
  watch: {
    params: {
       handler(){
         this.synth.volume.value = this.params.volume
         this.synth.portamento = this.params.portamento
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
      button
        border: solid 1px #ddd
        background-color: #eee
        color: #333
    .active
      button
        border: solid 1px darken(#42b983, 10%)
        background-color: #42b983
        color: #fff

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

</style>
