<script>
import Synth from './_Synth.vue'
export default {
  name: 'Sub',
  extends: Synth,
  data(){
    return {
      name: 'Sub',
      octave: 2,
      phraseLength: 4,
      noteLength: "1n"
    }
  }
}
</script>
