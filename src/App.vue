<template>
  <div id="app" class="app">

    <h2>Sandbox</h2>

    <!-- Layers (start) -->
    <Rack   v-if="true"  :notes="notes" :modes="modes" :scales="scales" ref="rack"/>
    <Rack   v-if="true"  :notes="notes" :modes="modes" :scales="scales" ref="rack2"/>
    <Rack   v-if="true"  :notes="notes" :modes="modes" :scales="scales" ref="rack3"/>

    <Kick   v-if="true"  ref="kick" />
    <Snare  v-if="true"  ref="snare" />
    <Hat    v-if="true"  ref="hat" />
    <!-- Layers (end) -->

    <div>
      <div class="globals">
        <select v-model="globalKey" @change="changeGlobalKey(globalKey)">
          <option v-for="note in notes" :value="note" :key="note">{{ note }}</option>
        </select>
        <select v-model="globalMode" @change="changeGlobalMode(globalMode)">
          <option v-for="mode in modes" :value="mode" :key="mode">{{ mode }}</option>
        </select>
        <button type="button"
        @click="randomizeGlobalKeyMode()">
          Global Key/Mode
        </button>
      </div>
      <div class="bpm">
        <b>BPM: {{ bpm }}</b>
        <div>
          <input type="range" min="70" max="200"
           v-model="bpm" @change="changeBpm(bpm)" >
        </div>
      </div>
      <button class="play-btn" type="button" @click="play()">Play</button>
    </div>
  </div>
</template>

<script>

import * as Tone from 'tone'

import Kick from './components/Kick.vue'
import Snare from './components/Snare.vue'
import Hat from './components/Hat.vue'
import Rack from './components/Rack.vue'

export default {
  name: 'App',
  components: { Kick, Snare, Hat, Rack },
  data(){
    return {
      notes: ['C', 'Db', 'D', 'Eb', 'E', 'F', 'Gb', 'G', 'Ab', 'A', 'Bb', 'B'],
      modes: ['ionian', 'dorian', 'phrygian', 'lydian', 'mixolydian', 'aeolian', 'locrian'],
      scales: {
        ionian:     [2, 2, 1, 2, 2, 2, 1],
        dorian:     [2, 1, 2, 2, 2, 1, 2],
        phrygian:   [1, 2, 2, 2, 1, 2, 2],
        lydian:     [2, 2, 2, 1, 2, 2, 1],
        mixolydian: [2, 2, 1, 2, 2, 1, 2],
        aeolian:    [2, 1, 2, 2, 1, 2, 2],
        locrian:    [1, 2, 2, 1, 2, 2, 2]
      },
      globalKey: "C",
      globalMode: "aeolian",
      bpm: 120,
      playing: false
    }
  },
  mounted(){
    // Global BPM
    Tone.Transport.bpm.value = this.bpm
    // Start Random Key/Mode
    this.randomizeGlobalKeyMode()
  },
  computed: {
    layers(){
      return Object.keys(this.$refs).map(x => this.$refs[x])
    }
  },
  methods: {
    changeBpm(bpm){ Tone.Transport.bpm.value = bpm },
    play(){
      if (Tone.context.state !== 'running') {
        Tone.context.resume()
      }
      if (!this.playing) {
        for(const layer of this.layers) {
          let synthPart = layer.createSequence()
          synthPart.start()
        }
        Tone.Transport.start()
        this.playing = true
      } else {
        Tone.Transport.stop()
        Tone.Transport.cancel()
        this.playing = false
      }
    },
    changeGlobalKey(key){
      for(const layer of this.layers) {
        layer.activeKey = key
      }
    },
    changeGlobalMode(mode){
      for(const layer of this.layers) {
        layer.activeMode = mode
      }
    },
    randomizeGlobalKeyMode(){
      this.globalKey = this.notes[Math.floor(Math.random() * this.notes.length)]
      this.globalMode = this.modes[Math.floor(Math.random() * this.modes.length)]
      this.changeGlobalKey(this.globalKey)
      this.changeGlobalMode(this.globalMode)
    }
  }
}
</script>

<style lang="sass">
  html
    box-sizing: border-box
  *, *:before, *:after
    box-sizing: inherit
  body
    position: relative
    min-height: 100vh
    margin: 0
    padding: 0
    background: #eee
    padding-left: 20px
    padding-right: 20px
  h1
    color: #2c3e50
    margin: 0
  h2
    font-size: 14px
    font-weight: 900
    text-transform: uppercase
    margin: 0 0 30px 0
  .app
    font-family: Avenir, Helvetica, Arial, sans-serif
    -webkit-font-smoothing: antialiased
    -moz-osx-font-smoothing: grayscale
    text-align: center
    color: #2c3e50
    padding: 20px 0
  .globals
    margin-bottom: 10px
  .bpm
    margin-bottom: 10px
  .play-btn
    display: inline-block
    padding: 8px 20px
    text-transform: uppercase
    font-weight: 600
    background-color: #2c3e50
    border: solid 1px darken(#2c3e50, 8%)
    border-radius: 6px
    color: #fff
    &:hover
      cursor: pointer
    &:hover,
    &:focus,
    &:active
      background-color: darken(#2c3e50, 8%)
      border-color: darken(#2c3e50, 16%)
</style>
