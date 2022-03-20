<template>
  <div id="app" class="app">
    <h1>Music Generator</h1>
    <h2>Tone and Vue.js</h2>

    <High :notes="notes" :modes="modes" :scales="scales" ref="high"/>
    <Melody :notes="notes" :modes="modes" :scales="scales" ref="melody"/>
    <Bass :notes="notes" :modes="modes" :scales="scales" ref="bass" />
    <Sub :notes="notes" :modes="modes" :scales="scales" ref="sub" />


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
          <input type="range"
           min="70" max="200"
           v-model="bpm"
           @change="changeBpm(bpm)" >
        </div>
      </div>
      <button class="play-btn" type="button" @click="play()">Play</button>
    </div>


    <!-- <footer>
      Footer
    </footer> -->

  </div>
</template>

<script>

import * as Tone from 'tone'
import High from './components/High.vue'
import Melody from './components/Melody.vue'
import Bass from './components/Bass.vue'
import Sub from './components/Sub.vue'

export default {
  name: 'App',
  components: {
    High, Melody, Bass, Sub
  },
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
      bpm: 120
    }
  },
  mounted(){
    Tone.Transport.bpm.value = this.bpm
  },
  methods: {
    play(){
      this.$refs.high.playRandomSequence("8n")
      this.$refs.melody.playRandomSequence("4n")
      this.$refs.bass.playRandomSequence("2n")
      this.$refs.sub.playRandomSequence("1n")
    },
    changeBpm(bpm){
      Tone.Transport.bpm.value = bpm
    },
    changeGlobalKey(key){
      this.$refs.high.activeKey = key
      this.$refs.melody.activeKey = key
      this.$refs.bass.activeKey = key
      this.$refs.sub.activeKey = key
    },
    changeGlobalMode(mode){
      this.$refs.high.activeMode = mode
      this.$refs.melody.activeMode = mode
      this.$refs.bass.activeMode = mode
      this.$refs.sub.activeMode = mode
    },
    randomizeGlobalKeyMode(){
      let notesNum = this.notes.length
      let modesNum = this.modes.length
      this.globalKey = this.notes[Math.floor(Math.random() * notesNum)]
      this.globalMode = this.modes[Math.floor(Math.random() * modesNum)]
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
