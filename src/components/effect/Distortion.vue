<template lang="html">
  <div class="fx" v-if="active">
    <h4>Distortion</h4>
    <div class="params">
      <div class="control">
        <h4>
          <span>Distortion: </span>
          <span>{{ params.distortion }}</span>
        </h4>
        <input type="range" min="0.00" max="1.00" step="0.01"
        v-model="params.distortion"
        @change="module.distortion = params.distortion">
      </div>
      <div class="control control--select">
        <h4>
          <span>Type: </span>
        </h4>
        <select v-model="params.oversample"
        @change="module.oversample = params.oversample">
          <option value="none">None</option>
          <option value="2x">2x</option>
          <option value="4x">4x</option>
        </select>
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
      name: 'distortion',
      params: {
        "distortion": 0.5,
        "oversample": 'none'
      },
      module: null
    }
  },
  mounted(){
    this.module = new Tone.Distortion(this.params)
    this.$emit('init', { module: this.module, target: this.name })
  }
}
</script>
