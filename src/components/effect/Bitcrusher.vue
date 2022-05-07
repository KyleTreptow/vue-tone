<template lang="html">
  <div class="fx" v-if="active">
    <h4>BitCrusher</h4>
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
          <span>Bits: </span>
          <span>{{ params.bits }}</span>
        </h4>
        <input type="range" min="1" max="8"
        v-model="params.bits"
        @change="module.bits = params.bits">
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
      name: 'bitcrusher',
      params: {
        "wet": 0.75,
        "bits": 5
      },
      module: null
    }
  },
  mounted(){
    this.module = new Tone.BitCrusher(this.params)
    this.$emit('init', { module: this.module, target: this.name })
  }
}
</script>
