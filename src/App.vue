<template>
  <div id="container">
        <CellList :cells="cells" @cellSelect="onCellSelect"/>
  </div>
</template>

<script>
import axios from 'axios';
import CellList from './components/CellList'

export default {
  name: 'App',
  components: {
        CellList: CellList,
    },
  data () {
        return { 
            cells: [],
            selectedCell: null,
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
    this.onOpen()
 },
 
}
</script>

<style scoped>
</style>