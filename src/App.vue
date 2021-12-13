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
            <grid-item class="cell" v-for="cell in cells"
                       :key="cell.id"
                       :x="cell.y"
                       :y="cell.x"
                       :w="1"
                       :h="1"
                       :i="cell.id"
                       v-bind:class="{ live: cell.is_live }"
            >
            <span class="span" @click="onCellSelect(cell)"></span>
            </grid-item>
        </grid-layout>
        </div>
        <div class="controller">
          <button class="btn" @click="onClickStart()"> {{button.text}} </button>
          <button class="btn" @click="onClickNext()"> Next </button>
          <button class="btn" @click="onClickReset()"> Reset </button>
        </div>
  </div>
  
</template>

<script>
import axios from 'axios';
// import CellList from './components/CellList'
import VueGridLayout from 'vue-grid-layout';

export default {
  name: 'App',
  components: {
        // CellList: CellList,
        GridLayout: VueGridLayout.GridLayout,
        GridItem: VueGridLayout.GridItem
    },
  data () {
        return { 
            cells: [],
            layout: [],
            button: {
              text: 'Start',
              isStart: false
            }
        };
    },
  methods: {
    onOpen() {
      axios.get('http://localhost:8081/', {
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
            axios.post('http://localhost:8081/reverse', {
              id: cell.id
            }, {
              headers: {"Content-Type": "application/x-www-form-urlencoded"}
            }).then(response => {
                if (response.data.success) {
                    cell.is_live = !cell.is_live
                }
            }).catch(reason => {
                console.log(reason);
            });
    },
    onClickStart() {
      if (this.button.isStart) {
        axios.post('http://localhost:8081/stop', {}, {
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
        axios.post('http://localhost:8081/start', {}, {
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
        axios.post('http://localhost:8081/next', {}, {
          headers: {"Content-Type": "application/x-www-form-urlencoded"}
        }).then(() => {
          // do nothing
        }).catch(reason => {
          console.log(reason);
        });
    },
    onClickReset() {
        axios.post('http://localhost:8081/reset', {}, {
          headers: {"Content-Type": "application/x-www-form-urlencoded"}
        }).then(() => {
          // do nothing
        }).catch(reason => {
          console.log(reason);
        });
    }
  },
  mounted(){
    this.$nextTick(function () {
            window.setInterval(() => {
                this.onOpen();
            },1000);
        })
 },
 
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

  .cell {
    background-color: white;
  }
  .live{
    background-color: black;
  }
  .span {
    position: absolute;
    top: 0;
    left: 0;
    height: 100%;
    width: 100%;
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
  }
 
</style>