<script>
import Synth from './_Synth.vue'
export default {
  name: 'Mid',
  extends: Synth,
  data(){
    return {
      name: 'Mid',
      octave: 3,
      phraseLength: 16,
      noteLength: "16n"
    }
  }
}
</script>
