<script>
import * as Tone from 'tone'
import Drum from './_Drum.vue'
export default {
  name: 'Kick',
  extends: Drum,
  data(){
    return {
      drumPattern: [
        "C2", "C2", null, "C2", null, null, null, "C2",
        "C2", null, null, "C2", null, "C2", null, "C2"
      ]
    }
  },
  mounted() {
    this.drum = new Tone.MembraneSynth().toDestination()
  }
}
</script>
