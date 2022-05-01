<script>
import Synth from './_Synth.vue'
export default {
  name: 'High',
  extends: Synth,
  data(){
    return {
      name: 'High',
      octave: 4,
      phraseLength: 32,
      noteLength: "8n",
      volume: 0,
      synthWaveForm: 'triangle'
    }
  }
}
</script>
