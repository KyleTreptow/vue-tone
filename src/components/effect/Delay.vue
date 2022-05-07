<template lang="html">
  <div class="fx" v-if="active">
    <h4>Delay</h4>
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
      <div class="control control--select">
        <h4>
          <span>Delay Time: </span>
        </h4>
        <select v-model="params.delayTime"
        @change="module.delayTime.value = params.delayTime">
          <option value="1n">1n</option>
          <option value="2n">2n</option>
          <option value="4n">4n</option>
          <option value="8n">8n</option>
          <option value="16n">16n</option>
        </select>
      </div>
      <div class="control">
        <h4>
          <span>Feedback: </span>
          <span>{{ params.feedback }}</span>
        </h4>
        <input type="range" min="0.00" max="1.00" step="0.01"
        v-model="params.feedback"
        @change="module.feedback.value = params.feedback">
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
      name: 'delay',
      params: {
        "wet": 0.6,
        "delayTime": "16n",
        "feedback": 0.75
      },
      module: null
    }
  },
  mounted(){
    this.module = new Tone.FeedbackDelay(this.params)
    this.$emit('init', { module: this.module, target: this.name })
  }
}
</script>
