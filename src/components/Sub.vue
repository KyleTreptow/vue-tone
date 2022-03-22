<script>
// import * as Tone from 'tone'
import Synth from './_Synth.vue'
export default {
  name: 'Sub',
  extends: Synth,
  data(){
    return {
      name: 'Sub',
      octave: 2,
      synthWaveForm: 'triangle',
      phraseLength: 4,
      noteLength: "1n"
    }
  },
  methods: {}
}
</script>
