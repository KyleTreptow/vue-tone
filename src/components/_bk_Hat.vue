<template lang="html">
  <div class="">
    <section>
      <button @click="startAudio()">Play Hat</button>
    </section>
  </div>
</template>

<script>
import * as Tone from 'tone'
export default {
  name: 'Hat',
  data(){
    return {
      drum: null
    }
  },
  mounted() {
    this.drum = new Tone.NoiseSynth({
      volume: -16,
			noise: {
				type: 'white',
				playbackRate: 3
			},
			envelope: {
				attack: 0.001,
				decay: 0.08,
				sustain: 0,
				release: 0
			}
    }).toDestination()
  },
  methods: {
    startAudio() {
      this.drum.triggerAttackRelease("4n")
    },
    createSequence(){ // creates a Tone sequence from pattern of notes
      // let that = this
      let d = this.drum
      let synthPart = new Tone.Sequence(
        function(time) {
          d.triggerAttackRelease("10hz", time) // note, release (10hz or 16n?), time
        },
        ["C2"], "8n"
      );
      return synthPart
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
</style>
