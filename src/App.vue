<template>
  <div id="app" class="app">
    <h1>Tone and Vue.js</h1>
    <h2>Key + Octave + Mode</h2>

    <High :notes="notes" :modes="modes" :scales="scales" ref="high"/>
    <Melody :notes="notes" :modes="modes" :scales="scales" ref="melody"/>
    <Bass :notes="notes" :modes="modes" :scales="scales" ref="bass" />
    <Sub :notes="notes" :modes="modes" :scales="scales" ref="sub" />

    <div class="bpm">
      <b>BPM: {{ bpm }}</b>
      <div>
        <input type="range"
         min="70" max="200"
         v-model="bpm"
         @change="changeBpm(bpm)" >
      </div>
    </div>

    <button type="button" @click="play()">Play</button>

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
  h1
    color: #2c3e50
    margin: 0 0 10px 0
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
    padding: 60px 0
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
