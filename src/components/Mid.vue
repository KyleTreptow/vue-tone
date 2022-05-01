<script>
import Synth from './_Synth.vue'
export default {
  name: 'Mid',
  extends: Synth,
  data(){
    return {
      name: 'Mid',
      octave: 4,
      phraseLength: 8,
      noteLength: "16n",
      volume: -10
    }
  }
}
</script>
