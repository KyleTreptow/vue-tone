<template lang="html">
  <div class="fx" v-if="active">
    <h4>Phaser</h4>
    <div class="params">
      <div class="control">
        <h4>
          <span>Dry/Wet: </span>
          <span>{{ params.wet }}</span>
        </h4>
        <input type="range" min="0.00" max="1.00" step="0.01"
        v-model="params.wet"
        @change="module.wet.value = params.wet">
      </div>
      <div class="control">
        <h4>
          <span>Frequency: </span>
          <span>{{ params.frequency }}</span>
        </h4>
        <input type="range" min="0" max="300"
        v-model="params.frequency"
        @change="module.frequency.value = params.frequency">
      </div>
      <div class="control control--select">
        <h4>
          <span>Octaves: </span>
        </h4>
        <select v-model="params.octaves"
        @change="module.octaves = params.octaves">
          <option v-for="i in 5" :value="i" :key="i">{{ i }}</option>
        </select>
      </div>
      <div class="control">
        <h4>
          <span>Base Frequency: </span>
          <span>{{ params.baseFrequency }}</span>
        </h4>
        <input type="range" min="350" max="1000"
        v-model="params.baseFrequency"
        @change="module.baseFrequency = params.baseFrequency">
      </div>
    </div>
  </div>
</template>

<script>
import * as Tone from 'tone'
import Effect from './_effect.vue'
export default {
  extends: Effect,
  data(){
    return {
      name: 'phaser',
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
    this.module = new Tone.Phaser(this.params)
    this.$emit('init', { module: this.module, target: this.name })
  }
}
</script>
