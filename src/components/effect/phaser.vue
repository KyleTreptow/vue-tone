<template lang="html">
  <div class="">
    <h4>
      <span>Phaser</span>
    </h4>
    <main class="params">
      <div class="control">
        <h4>
          <span>Dry/Wet: </span>
          <span>{{ params.wet }}</span>
        </h4>
        <input type="range" min="0" max="1" step="0.1"
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
        {{ params.octaves }} >
          <option v-for="i in 5" :value="i" :key="i">{{ i }}</option>
        </select>
      </div>
    </main>
  </div>
</template>

<script>
import * as Tone from 'tone'
import Effect from './_effect.vue'
export default {
  extends: Effect,
  data(){
    return {
      params: {
        "wet": 1,
        "frequency": 1,
        "octaves": 3,
        "stages": 10,
        "Q": 10,
        "baseFrequency": 350
      }
    }
  },
  mounted(){
    this.module = new Tone.Phaser(this.phaserParams)
    this.$emit('init', this.module)
  }
}
</script>
