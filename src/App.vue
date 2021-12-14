<template>
  <div id="container">
        <div>
           <grid-layout 
            :layout.sync="layout"
            :col-num="20"
            :row-height="30"
            :is-draggable="false"
            :is-resizable="false"
            :is-mirrored="false"
            :vertical-compact="false"
            :use-css-transforms="false"
            :useStyleCursor="false"
        >
            <grid-item  v-for="cell in cells"
                       :key="cell.id"
                       :x="cell.y"
                       :y="cell.x"
                       :w="1"
                       :h="1"
                       :i="cell.id"
                       v-bind:style="[cell.is_live ? {background: cell.color} : {background: 'white'}]"
                       
            >
            <span class="span"  @click="onCellSelect(cell)"></span>
            </grid-item>
        </grid-layout>
        </div>
        <div class="controller">
          <button class="btn" @click="onClickStart()"> {{button.text}} </button>
          <button class="btn" @click="onClickNext()"> Next </button>
          <button class="btn" @click="onClickReset()"> Reset </button>
          <button class="btn" value="1" v-on:click="onClickPattern($event)"> Glider </button>
          <button class="btn" value="2" v-on:click="onClickPattern($event)"> Beacon </button>
          <button class="btn" value="3" v-on:click="onClickPattern($event)"> Boat </button>
        </div>
        
  </div>
  
</template>

<script>
import axios from 'axios';
import VueGridLayout from 'vue-grid-layout';

// let backend_url = 'https://hidden-beach-63150.herokuapp.com'
// let ws_url = 'wss://hidden-beach-63150.herokuapp.com/ws'
let backend_url = 'http://localhost:8080'
let ws_url = 'ws://localhost:8080'

export default {
  name: 'App',
  components: {
        GridLayout: VueGridLayout.GridLayout,
        GridItem: VueGridLayout.GridItem
    },
  data () {
        return { 
            cells: [],
            layout: [],
            // update button text after click
            button: {
              text: 'Start',
              isStart: false
            },
            ws: null,
            // default color is black
            clientColor: '#000000',
            selectedCell: null
        };
    },
  methods: {
    onOpen() {
      // api for get cells
      axios.get(backend_url, {
            }).then(response => {
                this.cells = response.data.cells;
                if (response.data.is_start ) {
                  this.button.isStart = true
                  this.button.text = "Stop"
                } else {
                  this.button.isStart = false
                  this.button.text = "Start"
                }
            }).catch(reason => {
                console.log(reason);
            });
    },
    onCellSelect(cell) {
      // api for post reverse action
      // update cell status immediate
      this.selectedCell = cell;
      cell.is_live = !cell.is_live;
      cell.color = this.clientColor;
      axios.post(backend_url+'/reverse', {
        id: cell.id,
        color: this.clientColor
      }, {
            headers: {"Content-Type": "application/x-www-form-urlencoded"}
          }).then(() => {
              // do nothing
          }).catch(reason => {
              console.log(reason);
          });
    },
    onClickStart() {
      // api for post stop
      if (this.button.isStart) {
        axios.post(backend_url+'/stop', {}, {
          headers: {"Content-Type": "application/x-www-form-urlencoded"}
        }).then(response => {
          if (response.data.success) {
            // change stop btn to start
            this.button.isStart = false
            this.button.text = "Start"
          }
        }).catch(reason => {
          console.log(reason);
        });
      } else {
        // api for post start
        axios.post(backend_url+'/start', {}, {
          headers: {"Content-Type": "application/x-www-form-urlencoded"}
        }).then(response => {
          if (response.data.success) {
            // change start btn to stop
            this.button.isStart = true
            this.button.text = "Stop"
          }
        }).catch(reason => {
          console.log(reason);
        });
      }
        
    },
    onClickNext() {
      // api for post next
        axios.post(backend_url+'/next', {}, {
          headers: {"Content-Type": "application/x-www-form-urlencoded"}
        }).then(() => {
          // do nothing
        }).catch(reason => {
          console.log(reason);
        });
    },
    onClickReset() {
      // api for post reset
        axios.post(backend_url+'/reset', {}, {
          headers: {"Content-Type": "application/x-www-form-urlencoded"}
        }).then(() => {
          // do nothing
        }).catch(reason => {
          console.log(reason);
        });
    },
    onClickPattern(e) {
      // api for change pattern
        axios.post(backend_url+'/pattern', {
            pattern: parseInt(e.target.value, 10)
        }, {
          headers: {"Content-Type": "application/x-www-form-urlencoded"}
        }).then(response => {
          this.cells = response.data.cells
        }).catch(reason => {
          console.log(reason);
        });
    },
    getColor() {
      // api for get color
        axios.get(backend_url+'/color', {}, {
          headers: {"Content-Type": "application/x-www-form-urlencoded"}
        }).then(response => {
          if (response.data.color) {
            this.clientColor = response.data.color
          }
        }).catch(reason => {
          console.log(reason);
        });
    },
    initWebSocket(){ 
        this.ws = new WebSocket(ws_url+'/ws')
        this.ws.onmessage = this.websocketOnmessage;
        this.ws.onopen = this.websocketOnopen;
        this.ws.onerror = this.websocketOnerror;
        this.ws.onclose = this.websocketClose;
      },
      websocketOnopen(){ 
        // ask client color or say hi?
      },
      websocketOnerror(){
        this.initWebSocket();
      },
      websocketOnmessage(e){ 
        let data = JSON.parse(e.data);
        this.cells = data.cells;
        if (data.is_start) {
          this.button.isStart = true
          this.button.text = "Stop"
        } else {
          this.button.isStart = false
          this.button.text = "Start"
        }
      },
      websocketSend(Data){
        this.ws.send(Data);
      },
      websocketClose(e){ 
        console.log('websocket closed',e);
      },
  },
  mounted(){
    this.onOpen();
    this.getColor();
    this.$nextTick(function () {
            window.setInterval(() => {
              console.log('heartbeat')
                this.websocketSend('ping')
            },8000); // send ping to ws
        })
  },
  created() {
    if (process.env.NODE_ENV == 'production') {
      backend_url = process.env.VUE_APP_API_URL;
      ws_url = process.env.VUE_APP_WS_URL;
    }
    this.initWebSocket();
  },
  destroyed() {
      this.ws.close();
  },
  computed: {
    colourChange(color){
      console.log(color)
      return String.format('backgroundColor: {0};', color);
    }
  }
}
</script>

<style scoped>
.vue-grid-layout {
  background: #eee;
}
.vue-grid-item:not(.vue-grid-placeholder) {
  border: 1px solid black;
}
.vue-grid-item .resizing {
  opacity: 0.9;
}
.vue-grid-item .static {
  background: #cce;
}
.span {
  position: absolute;
  top: 0;
  left: 0;
  height: 100%;
  width: 100%;
}

.container {
  position: absolute;
  top: 30px;
  bottom: 30px;
  left: 20%;
  height: 70%;
  width: 50%;
}

.controller {
  position: absolute;
  bottom: 0;
  left: 35%;
  height: 10%;
  width: 30%;
}
.btn {
  width: 80px;
  height: 30px;
  margin-right: 5px;
  margin-bottom: 5px;
}
 
</style>