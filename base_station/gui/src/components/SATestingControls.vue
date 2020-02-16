<template>
  <div class="wrap">
    <div class="flex">
      <SASiteControls v-bind:site="1"/>
    </div>
    <div class="led-toggles">
      <Checkbox v-bind:name="'Toggle Backlights'" v-on:toggle="setPart('backlights', $event)"/>
      <Checkbox v-bind:name="'Toggle UV Lights'" v-on:toggle="setPart('uv_leds', $event)"/>
      <Checkbox ref="rgb" v-bind:name="'Toggle RGB Sensor Lights'" v-on:toggle="setRGBLeds($event)"/>
      <Checkbox v-bind:name="'Turn Mosfet On/Off'" v-on:toggle="turnOnMosfet($event, '1')"/>
      
    </div>
    <div class="flex">
      <SASiteControls v-bind:site="2"/>
    </div>
    <div class="box">
      <h4>GPS Data</h4>
      Latitude: {{gps_data.latitude_deg}}<br>
      Min Latitude: {{gps_data.latitude_min}}<br>
      Longitude: {{gps_data.longitude_deg}}<br>
      Min Longitude: {{gps_data.longitude_min}}<br>
      Bearing: {{gps_data.bearing_deg}}<br>
      Speed: {{gps_data.speed}}
    </div>
    <div class="flex">
      <button ref="raman" class="button" v-on:click="sendCollect($event)"> <span>Raman Test</span> </button>
      <button v-on:click="sendGPSData()">GPS Data</button>
    </div>
    <div class="flex">
      <span>
      Temperature: {{thermistor_data.temperature}}
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

</style>

<script>
  import SASiteControls from './SASiteControls.vue'
  import Checkbox from './Checkbox.vue'
  
  export default {
    data() {
      return {
        thermistor_data: {
          temperature: 0
        },
        gps_data: {
          latitude_deg: 0,
          latitude_min: 0,
          longitude_deg: 0,
          longitude_min: 0,
          bearing_deg: 0,
          speed: 0
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

    created: function () {
      this.$parent.subscribe('/thermistor_data', (msg) => {
        this.thermistor_data = msg
      }),

      this.$parent.subscribe('/gps_data', (msg) => {
        this.gps_data = msg
      })
    },

    methods: {
      setPart: function(id, enabled) {
        this.$parent.publish("/mosfet", {
          'type': 'Mosfet',
          'id': id,
          'enable': enabled
        })
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
      },

      turnOnMosfet: function (enabled, id) {
        this.$parent.publish("/mosfet_cmd", {
          'type': 'MosfetCmd',
          'id': id,
          'state': enabled
        })
      },

      sendGPSData: function() {
        this.$parent.publish("/gps_data", {
          'GPS': GPS,
        })
      }
    },

    components: {
      SASiteControls,
      Checkbox
    }
  }
</script>
