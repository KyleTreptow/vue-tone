<template lang="html">
  <div class="#">
    <section>
      <div class="#">
        <button v-for="x in drumPattern.length"
          :key="'kick_input_'+x"
          @click="changeDrumHit(x-1)"
          class="drum-input"
          :class="{ active: drumPattern[x-1] != null }">
          {{ x }}
        </button>
      </div>
      <div v-if="false">
        {{ drumPattern }}
      </div>
      <small>Drum Hit #{{ playingIndex }}</small>
    </section>
  </div>
</template>

<script>
import * as Tone from 'tone'
export default {
  name: 'Drum',
  data(){
    return {
      drum: null,
      playingIndex: 0,
      drumPattern: [
        "C2", null, null, null, "C2", null, null, null
      ],
      hitLength: "8n"
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
      // Seq
      let that = this
      let d = this.drum
      let synthPart = new Tone.Sequence(
        function(time, note) {
          d.triggerAttackRelease(note, "10hz", time) // note, release (10hz or 16n?), time
          // drum index
          that.playingIndex++
        },
        this.drumPattern, "8n"
      );
      return synthPart
    },
    changeDrumHit(idx){
      // play kick
      this.startAudio()
      // change input
      let pattern = [...this.drumPattern]
      if(pattern[idx] == null){ pattern[idx] = 'C2' }
      else { pattern[idx] = null }
      this.drumPattern = pattern
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
    .drum-input
      border: solid 1px #ddd
      &.active
        background-color: #2c3e50
        color: #fff
        border-color: #2c3e50
  </style>
