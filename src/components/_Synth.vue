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
          <h4>Effects</h4>
          <div class="effectlist">
            <div class="inactive">
              <button v-for="e in inactiveEffects" :key="'effect_'+e"
              type="button" @click="activateEffect(e)">
                {{ e }} <span>+</span>
              </button>
            </div>
            <div class="active">
              <span v-for="e in activeEffects" :key="'active_effect_'+e">
                <button type="button" @click="deactivateEffect(e)">
                  {{ e }}
                </button>
                <button type="button"
                @click="displayEffectParam == e ? displayEffectParam = null : displayEffectParam = e ">
                  {{ displayEffectParam == e ? '-' : '+' }}
                </button>
              </span>
            </div>
          </div>
          <div class="effect-params">

            <component :is="'delay'" :active="displayEffectParam == 'delay'"
            @init="initEffect" @close="closeEffectParams">
            </component>
            <component :is="'reverb'" :active="displayEffectParam == 'reverb'"
            @init="initEffect" @close="closeEffectParams">
            </component>
            <component :is="'chorus'" :active="displayEffectParam == 'chorus'"
            @init="initEffect" @close="closeEffectParams">
            </component>
            <component :is="'distortion'" :active="displayEffectParam == 'distortion'"
            @init="initEffect" @close="closeEffectParams">
            </component>
            <component :is="'phaser'" :active="displayEffectParam == 'phaser'"
            @init="initEffect" @close="closeEffectParams">
            </component>
            <component :is="'bitcrusher'" :active="displayEffectParam == 'bitcrusher'"
            @init="initEffect" @close="closeEffectParams">
            </component>
            <component :is="'ffilter'" :active="displayEffectParam == 'ffilter'"
            @init="initEffect" @close="closeEffectParams">
            </component>


          </div>
          <!-- <button type="button" @click="log(synth)">Log Synth</button> -->
        </footer>
      </div>

  </section>
  </div>
</template>

<script>
import * as Tone from 'tone'
import Phaser from './effect/Phaser.vue'
import Delay from './effect/Delay.vue'
import Reverb from './effect/Reverb.vue'
import Chorus from './effect/Chorus.vue'
import Distortion from './effect/Distortion.vue'
import Bitcrusher from './effect/Bitcrusher.vue'
import Ffilter from './effect/Ffilter.vue'
export default {
  name: 'Synth',
  components: { Phaser, Delay, Reverb, Chorus, Distortion, Bitcrusher, Ffilter },
  props: ['notes', 'modes', 'scales'],
  data(){
    return {
      name: 'Synth',
      synth: null,
      activeKey: 'C',
      octave: 4,
      activeMode: 'aeolian',
      phraseLengthList: [2, 4, 8, 16, 32, 64],
      phraseLength: 32,
      noteLengthList: ['1n', '2n', '4n', '8n', '16n'],
      noteLength: "8n",
      displaySeqArr: false,
      seqArray: [],
      liveNote: null,
      waveForms: ['sine', 'triangle', 'square', 'sawtooth'],
      // synth parameters
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
      // effects units
      effects: {
        delay: null,
        reverb: null,
        chorus: null,
        distortion: null,
        phaser: null,
        bitcrusher: null,
        filter: null
      },
      effectsList: ['delay', 'reverb', 'chorus', 'distortion', 'phaser', 'bitcrusher', 'ffilter'],
      activeEffects: [],
      displayEffectParam: null
    }
  },
  mounted() {
    // Init Synth & route to destination clean
    this.synth = new Tone.Synth(this.params)
    this.synth.toDestination()

    // LFO Test
    // this.synth.chain(this.effects.filter, this.effects.delay, Tone.Destination)

    // const lfo = new Tone.LFO({
    //   min: 0,
    //   max: 1,
    //   frequency: '1n'
    // })
    // lfo.start()
    // lfo.connect(this.effects.phaser.wet)

    // Ramp Values
    // this.effects.delay.wet.rampTo(1, 3)

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
      console.log(this.effects)
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
      console.log(fx)
    },
    activateEffect(item){
      this.activeEffects.push(item)
      this.disconnect()
      this.connect()
    },
    deactivateEffect(item){
      let e = this.displayEffectParam
      this.displayEffectParam = (e == item) ? null : e
      // ####
      let index = this.activeEffects.indexOf(item)
      if (index !== -1) {
        this.activeEffects.splice(index, 1);
      }
      this.disconnect()
      this.connect()
    },
    // EFFECT MODULE
    initEffect({ module, target }){
      this.effects[target] = module
    },
    closeEffectParams(value){
      this.displayEffectParam = value
    },
    openEffectparam(module){
      this.displayEffectParam = module
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

</style>
