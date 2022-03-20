<script>
// import * as Tone from 'tone'
import Synth from './_Synth.vue'
export default {
  name: 'Bass',
  extends: Synth,
  data(){
    return {
      name: 'Bass',
      octave: 3,
      synthWaveForm: 'sine',
      phraseLength: 8,
    }
  },
  methods: {}
}
</script>
