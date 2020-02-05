<template>
  <div class="wrap">
    <div class="flex">
      <SASiteControls v-bind:site="1"/>
    </div>
    <div class="led-toggles">
      <Checkbox v-bind:name="'Toggle Backlights'" v-on:toggle="setPart('backlights', $event)"/>
      <Checkbox v-bind:name="'Toggle UV Lights'" v-on:toggle="setPart('uv_leds', $event)"/>
      <Checkbox ref="rgb" v-bind:name="'Toggle RGB Sensor Lights'" v-on:toggle="setRGBLeds($event)"/>
    </div>
    <div class="flex">
      <SASiteControls v-bind:site="2"/>
    </div>
    <div class="spectrometerInput">
      <input type="number" v-model="id">
      <button v-on:click="sendSpectralCmd(id)">Spectrometer ID</button>
    </div>
    <div class="flex">
      <button ref="raman" class="button" v-on:click="sendCollect($event)"> <span>Raman Test</span> </button>
    </div>
    <div class="spectrometerOutput">
      <span>
        r: {{SpectralData.r}}<br>
        g: {{SpectralData.g}}<br>
        a: {{SpectralData.a}}<br>
        s: {{SpectralData.s}}<br>
        h: {{SpectralData.h}}<br>
        b: {{SpectralData.b}}<br>
        t: {{SpectralData.t}}<br>
        i: {{SpectralData.i}}<br>
        c: {{SpectralData.c}}<br>
        u: {{SpectralData.u}}<br>
        j: {{SpectralData.j}}<br>
        d: {{SpectralData.d}}<br>
        v: {{SpectralData.v}}<br>
        k: {{SpectralData.k}}<br>
        e: {{SpectralData.e}}<br>
        w: {{SpectralData.w}}<br>
        l: {{SpectralData.l}}<br>
        f: {{SpectralData.f}}

      </span>
    </div>
  </div>
</template>

<style scoped>
  .wrap {
    display: grid;
    grid-gap: 10px;
    grid-template-columns: 1fr 1fr;
    grid-template-rows: 1fr 1fr;
  }

  .box {
    border-radius: 5px;
    padding: 10px;
    border: 1px solid black;
  }

  .flex {
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .led-toggles {
    display: grid;
    grid-template-rows: 1fr 1fr 1fr;
    justify-items: center;
  }

  .spectrometerInput {
    display: grid;
    grid-template-rows: 1fr;
    justify-items: center;
    padding: 40px;
  }

  .spectrometerOutput {
    display: grid;
    grid-template-rows: 1fr;
    border-radius: 5px;
    border: 1px solid black;
    height: 60px;
    width: 100px;
    overflow: auto;
    margin-left : 10px;
  }

</style>

<script>
  import SASiteControls from './SASiteControls.vue'
  import Checkbox from './Checkbox.vue'
  
  export default {
    data() {
      return {
        id: 0,

        SpectralData: {
          r: 0,
          g: 0,
          a: 0,
          s: 0,
          h: 0,
          b: 0,
          t: 0,
          i: 0,
          c: 0,
          u: 0,
          j: 0,
          d: 0,
          v: 0,
          k: 0,
          e: 0,
          w: 0,
          l: 0,
          f: 0
        }
      }
    },

    beforeDestroy: function () {

    },

    mounted: function () {
      this.$refs["rgb"].active = true
    },

    props: {

    },

    methods: {
      setPart: function(id, enabled) {
        this.$parent.publish("/mosfet", {
          'type': 'Mosfet',
          'id': id,
          'enable': enabled
        })
      },
      
      sendSpectralCmd: function(id) {
        this.$parent.publish("/triad_trigger",
         {'type' : "SpectralCmd",
           'id' : Number(id)})
      },

      setRGBLeds: function(enabled) {
        this.$parent.publish("/rgb_leds", {
          'type': 'RGBLED',
          'on': enabled
        })
      },

      sendCollect: function (button) {
        this.$parent.publish("/raman_collect", {"type": "Signal"})
        let obj = this.$refs["raman"]
        obj.disabled = true
        setTimeout(function() {
          obj.disabled = false;
        }, 2000);
      }
    },

    created: function() {
      this.$parent.subscribe('/triad_data', (msg) => { 
        this.SpectralData = msg
      })
    },

    components: {
      SASiteControls,
      Checkbox
    }
  }
</script>
