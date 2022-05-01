<script>
import Synth from './_Synth.vue'
export default {
  name: 'Bass',
  extends: Synth,
  data(){
    return {
      name: 'Bass',
      octave: 2,
      phraseLength: 8,
      noteLength: "2n",
      volume: 8,
      synthWaveForm: 'sine'
    }
  }
}
</script>
