<template lang="html">
  <div class="fx" v-if="active">
    <h4>Chorus</h4>
    <div class="params">
      <div class="control">
        <h4>
          <span>Frequency: </span>
          <span>{{ params.frequency }}</span>
        </h4>
        <input type="range" min="0.05" max="20.00" step="0.01"
        v-model="params.frequency"
        @change="module.frequency.value = params.frequency">
      </div>
      <div class="control">
        <h4>
          <span>Delay Time: </span>
          <span>{{ params.delayTime }} sec</span>
        </h4>
        <input type="range" min="0.1" max="5.0" step="0.1"
        v-model="params.delayTime"
        @change="module.delayTime = params.delayTime">
      </div>
      <div class="control">
        <h4>
          <span>Depth: </span>
          <span>{{ params.depth }}</span>
        </h4>
        <input type="range" min="0.0" max="1.0" step="0.1"
        v-model="params.depth"
        @change="module.depth = params.depth">
      </div>
      <div class="control control--select">
        <h4>
          <span>Type: </span>
        </h4>
        <select v-model="params.type"
        @change="module.type = params.type">
          <option value="sine">Sine</option>
          <option value="triangle">Triangle</option>
          <option value="square">Square</option>
          <option value="sawtooth">Sawtooth</option>
        </select>
      </div>
      <div class="control">
        <h4>
          <span>Spread: </span>
          <span>{{ params.spread }} deg</span>
        </h4>
        <input type="range" min="0" max="180"
        v-model="params.spread"
        @change="module.spread = params.spread">
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
      name: 'chorus',
      params: {
        "frequency": 1.75, // 0 - 20hz? or 0-20k??
        "delayTime": 3.5,
        "depth": 0.7,
        "type": 'sine',
        "spread": 180
      },
      module: null
    }
  },
  mounted(){
    this.module = new Tone.Chorus(this.params)
    this.$emit('init', { module: this.module, target: this.name })
  }
}
</script>
