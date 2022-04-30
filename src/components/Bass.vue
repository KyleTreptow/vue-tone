<script>
import Synth from './_Synth.vue'
export default {
  name: 'Bass',
  extends: Synth,
  data(){
    return {
      name: 'Bass',
      octave: 3,
      phraseLength: 8,
      noteLength: "2n",
      volume: 0
    }
  }
}
</script>
