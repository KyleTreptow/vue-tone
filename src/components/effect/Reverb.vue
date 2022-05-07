<template lang="html">
  <div class="fx" v-if="active">
    <h4>Reverb</h4>
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
          <span>Decay: </span>
          <span>{{ params.decay }} sec</span>
        </h4>
        <input type="range" min="0.01" max="3.00" step="0.01"
        v-model="params.decay"
        @change="module.decay = params.decay">
      </div>
      <div class="control">
        <h4>
          <span>PreDelay: </span>
          <span>{{ params.preDelay }} sec</span>
        </h4>
        <input type="range" min="0.00" max="0.50" step="0.01"
        v-model="params.preDelay"
        @change="module.preDelay = params.preDelay">
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
      name: 'reverb',
      params: {
        "wet": 0.4,
        "decay": 1.5,
        "preDelay": 0.01
      },
      module: null
    }
  },
  mounted(){
    this.module = new Tone.Reverb(this.params)
    this.$emit('init', { module: this.module, target: this.name })
  }
}
</script>
