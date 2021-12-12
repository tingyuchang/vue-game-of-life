<template>
    <div class="cell">
        cell
    </div>
</template>


<script>
export default {
    name: "CellListItem",
    props: ['cell'],
    computed: {
        
    },
    methods: {
        onCellSelect() {
            this.$emit('cellSelect', this.cell);
        },
    },
}
</script>>

<style scoped>
    .cell {
         background-color: #ffff00;
    }
</style>