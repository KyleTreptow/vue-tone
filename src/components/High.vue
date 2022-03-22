<script>
// import * as Tone from 'tone'
import Synth from './_Synth.vue'
export default {
  name: 'High',
  extends: Synth,
  data(){
    return {
      name: 'High',
      octave: 5,
      phraseLength: 32,
      noteLength: "8n"
    }
  },
  methods: {}
}
</script>
