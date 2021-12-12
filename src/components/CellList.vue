<template>
<div>
    <CellListItem 
            v-for="cell in cells" 
            :key="cell.id" 
            :cell="cell"
            @cellSelect="onCellSelect"
        >
        </CellListItem>
</div>
</template>>

<script>
import CellListItem from './CellListItem'

export default {
    name: "CellList",
    components: {
        CellListItem: CellListItem,
    },
    props: {
        cells: Array,
    },
    methods: {
        onCellSelect(cell) {
            this.$emit('cellSelect', cell);
        },
        showData() {
            console.log(this.cells)
        }
    },
    mounted() {
        this.showData();
    },

}
</script>>

<style>

</style>>