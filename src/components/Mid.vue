<script>
// import * as Tone from 'tone'
import Synth from './_Synth.vue'
export default {
  name: 'Melody',
  extends: Synth,
  data(){
    return {
      name: 'Mid',
      octave: 4,
      phraseLength: 16,
      noteLength: "4n"
    }
  }
}
</script>
