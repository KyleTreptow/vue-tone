<template lang="html">
  <div class="">
    <h4>Phaser</h4>
    
    <div class="control">
      <span>Dry/Wet</span>
      <input type="range" min="0" max="1" step="0.1"
      v-model="params.wet"
      @change="module.wet.value = params.wet">
      {{ params.wet }}
    </div>

    <div class="">
      <span>Frequency</span>
      <input type="range" min="0" max="300"
      v-model="params.frequency"
      @change="module.frequency.value = params.frequency">
      {{ params.frequency }}
    </div>
    <div class="">
      <span>Octaves</span>
      <select v-model="params.octaves"
      @change="module.octaves = params.octaves">
      {{ params.octaves }} >
        <option v-for="i in 5" :value="i" :key="i">{{ i }}</option>
      </select>
    </div>
  </div>

</template>

<script>
import * as Tone from 'tone'
export default {
  data(){
    return {
      params: {
        "wet": 1,
        "frequency": 1,
        "octaves": 3,
        "stages": 10,
        "Q": 10,
        "baseFrequency": 350
      },
      module: null
    }
  },
  mounted(){
    this.module = new Tone.Phaser(this.phaserParams)
    this.$emit('init', this.module)
  }
}
</script>

<style lang="css">
</style>
