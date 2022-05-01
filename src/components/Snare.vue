<script>
import * as Tone from 'tone'
import Drum from './_Drum.vue'
export default {
  name: 'Snare',
  extends: Drum,
  data(){
    return {
      noteLength: "4n",
      drumPattern: [
        null, null, "C2", null, null, null, "C2", null,
        null, null, "C2", null, null, null, "C2", null
      ]
    }
  },
  mounted() {
    this.drum = new Tone.NoiseSynth({
      volume: 14,
      noise: {
        type: 'pink',
        playbackRate: 3
      },
      envelope: {
        attack: 0.001,
        decay: 0.18,
        sustain: 0,
        release: 0.05
      }
    }).toDestination()
  },
  methods: {
    startAudio() {
      this.drum.triggerAttackRelease("4n")
    },
    createSequence(){ // creates a Tone sequence from pattern of notes
      let that = this
      let d = this.drum
      let synthPart = new Tone.Sequence(
        function(time) {
          d.triggerAttackRelease("10hz", time) // note, release (10hz or 16n?), time
          // drum index
          that.playingIndex++
        },
        this.drumPattern, this.noteLength
      );
      return synthPart
    }
  }
}
</script>
