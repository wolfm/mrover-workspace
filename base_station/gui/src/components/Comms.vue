<template>
  <div class="wrapper">
    <div class="box header">
      <img src="/static/mrover.png" alt="MRover" title="MRover" width="48" height="48" />
      <h1>Comms</h1>
      <div class="spacer"></div>
      <div class="comms">
        <ul id="vitals">
          <li><CommIndicator v-bind:connected="connections.websocket" name="Web Socket" /></li>
          <li><CommIndicator v-bind:connected="connections.lcm" name="Rover Connection Status" /></li>
        </ul>
      </div>
      <div class="spacer"></div>
    </div>
    <div class="box">

        <RadioSignalStrength/>
      
    </div>

  </div>
</template>

<script>
  import LCMBridge from 'lcm_bridge_client/dist/bridge.js';
  import CommIndicator from './CommIndicator.vue'
  import RadioSignalStrength from './RadioSignalStrength.vue'
  import msgs from '../static/rover_msgs.json'

  export default {
    name: 'Comms',
    data () {
      return {
        lcm_: null,
        connections: {
          websocket: false,
          lcm: false
        },
        subscriptions: [
          {'topic': '/radio_update', 'type': 'RadioSignalStrength' },
        ]
      }
    },

    methods: {
      publish: function (channel, payload) {
        this.lcm_.publish(channel, payload)
      },

      subscribe: function (channel, callbackFn) {
        if( (typeof callbackFn !== "function") || (callbackFn.length !== 1)) {
          console.error("Callback Function is invalid (should take 1 parameter)")
        }
        this.lcm_.subscribe(channel, callbackFn)
      }
    },

    created: function () {
      this.lcm_ = new LCMBridge(
        'ws://localhost:8001',
        // Update WebSocket connection state
        (online) => {
          this.lcm_.setHomePage()
          this.connections.websocket = online
        },
        // Update connection states
        (online) => {
          this.connections.lcm = online[0]
        },
        // Subscribed LCM message received
        (msg) => {
        },
        // Subscriptions
        this.subscriptions
      )
    },
    components: {
      CommIndicator,
      RadioSignalStrength
    }
  }
</script>

<style scoped>
  .wrapper {
    display: grid;
    grid-gap: 10px;
    grid-template-columns: 1fr 1fr;
    grid-template-rows: 60px 1fr;
    grid-template-areas: "header header" "checklist feed";

    font-family: sans-serif;
    height: 98vh;
  }

  .header {
    grid-area: header;
    display: flex;
    align-items: center;
  }

  .header h1 {
    margin-left: 5px;
  }

  .box {
    border-radius: 5px;
    padding: 10px;
    border: 1px solid black;
  }

  .spacer {
    flex-grow: 0.8;
  }

  .comms {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
  }

  .comms * {
    margin-top: 2px;
    margin-bottom: 2px;
  }

  .messagefeed {
    overflow: auto;
  }

  .message_textarea {
    height: 500px;
  }

  .param {
    margin-bottom: 5px
  }

  .param_type {
    color: grey;
    margin-right: 2px;
  }

  .param_label {
    margin-right: 2px;
  }

  p {
    margin-bottom: 1em;
  }

  ul#channels li {
    display: block;
    padding: 0px 10px 0px 0px;
  }

  ul#feed li {
    display: block;
    padding: 0px 10px 0px 0px;
    white-space: nowrap;
  }

  ul#vitals li {
    display: inline;
    float: left;
    padding: 0px 10px 0px 0px;
  }

  textarea {
    resize: none;
  }
</style>
