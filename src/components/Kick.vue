<template lang="html">
  <div class="">
    <section>
      <button @click="startAudio()">Play Kick</button>
    </section>
  </div>
</template>

<script>
import * as Tone from 'tone'
export default {
  name: 'Kick',
  data(){
    return {
      drum: null
    }
  },
  mounted() {
    this.drum = new Tone.MembraneSynth().toDestination()
  },
  methods: {
    startAudio() {
      this.drum.triggerAttackRelease("C2", "4n")
    },
    createSequence(){ // creates a Tone sequence from pattern of notes
      // let that = this
      let d = this.drum
      let synthPart = new Tone.Sequence(
        function(time, note) {
          d.triggerAttackRelease(note, "10hz", time) // note, release (10hz or 16n?), time
        },
        ["C2", "C2", null, "C2", null, null, null, "C2"], "8n"
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
