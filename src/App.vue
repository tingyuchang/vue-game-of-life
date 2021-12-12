<template>
  <div id="container">
        <!-- <CellList :cells="cells" @cellSelect="onCellSelect"/> -->
          <grid-layout 
            :layout.sync="layout"
            :col-num="20"
            :row-height="30"
            :is-draggable="false"
            :is-resizable="true"
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
            
            </grid-item>
        </grid-layout>
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
            selectedCell: null,
            layout: [],
        };
    },
  methods: {
    onOpen() {
      axios.get('http://localhost:8081/', {
            }).then(response => {
                this.cells = response.data;
            }).catch(reason => {
                console.log(reason);
            });
    },
    onCellSelect(cell) {
            console.log(cell.id);
            this.selectedCell = cell;
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
  .container .vue-grid-item.vue-grid-placeholder {
    background: green;
}
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
</style>