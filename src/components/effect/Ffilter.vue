<template lang="html">
  <div class="fx" v-if="active">
    <h4>Filter</h4>
    <div class="params">
      <div class="control">
        <h4>
          <span>Frequency: </span>
          <span>{{ params.frequency }} hz</span>
        </h4>
        <input type="range" min="40" max="12000"
        v-model="params.frequency"
        @change="module.frequency.value = params.frequency">
      </div>
      <div class="control control--select">
        <h4>
          <span>Type: </span>
        </h4>
        <select v-model="params.type"
        @change="module.type = params.type">
          <option v-for="f in filterTypeList" :value="f" :key="f">{{ f }}</option>
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
      name: 'ffilter',
      params: {
        "type" : 'lowpass',
        "frequency" : 12000,
        "rolloff" : -24,
        "Q" : 1,
        "gain" : 2
      },
      module: null,
      filterTypeList: ["lowpass", "highpass", "bandpass", "lowshelf", "highshelf", "notch", "allpass", "peaking"]
    }
  },
  mounted(){
    this.module = new Tone.Filter(this.params)
    this.$emit('init', { module: this.module, target: this.name })
  }
}
</script>
