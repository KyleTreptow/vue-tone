<button type="button" @click="createSequence()">Create Sequence</button>

<button type="button" @click="paramsActive = !paramsActive">Synth Parameters</button>

<div class="params" v-if="paramsActive">
  <main>
    <div class="param">
      <h4 class="param__header">Oscillator</h4>
      <div class="control">
        <h4>
          <span>Volume:</span>
          <span>{{ params.volume }} dB</span>
        </h4>
        <input type="range" min="-24" max="24"
         v-model="params.volume"
         @change="synth.volume.value = params.volume">
      </div>

      <div class="control">
        <h4>
          <span>Portamento:</span>
          <span>{{ params.portamento }}</span>
        </h4>
        <input type="range" min="0.00" max="0.10" step="0.01"
         v-model="params.portamento"
         @change="synth.portamento = params.portamento">
      </div>

      <div class="control">
        <h4>
          <span>Phase:</span>
          <span>{{ params.oscillator.phase }} deg</span>
        </h4>
        <input type="range" min="0" max="360"
         v-model="params.oscillator.phase">
      </div>

      <div class="control control--select">
        <h4>
          <span>Waveform:</span>
        </h4>
        <select v-model="params.oscillator.type">
          <option v-for="wf in waveForms" :value="wf" :key="wf">{{ wf }}</option>
        </select>
      </div>

    </div>
    <div class="param">
      <h4>Envelope</h4>

      <div class="control">
        <h4>
          <span>Attack: </span>
          <span>{{ params.envelope.attack }}</span>
        </h4>
        <input type="range" min="0.01" max="2.00" step="0.01"
         v-model="params.envelope.attack">
      </div>

      <div class="control">
        <h4>
          <span>Decay: </span>
          <span>{{ params.envelope.decay }}</span>
        </h4>
        <input type="range" min="0.01" max="2.00" step="0.01"
         v-model="params.envelope.decay">
      </div>

      <div class="control">
        <h4>
          <span>Sustain:</span>
          <span>{{ params.envelope.sustain }}</span>
        </h4>
        <input type="range" min="0" max="1" step="0.01"
         v-model="params.envelope.sustain">
      </div>

      <div class="control">
        <h4>
          <span>Release:</span>
          <span>{{ params.envelope.release }}</span>
        </h4>
        <input type="range" min="0.01" max="2.00" step="0.01"
         v-model="params.envelope.release">
      </div>

    </div>
  </main>
  <footer>
    <!-- <button type="button" @click="log(synth)">Log Synth</button> -->
  </footer>
</div>
