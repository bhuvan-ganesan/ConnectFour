<template>
  <div class="board">
    <div v-for="(column, x) in board" :key="x" class="column" @click="handleClick(x)">
      <div v-for="(cell, y) in column" :key="y" class="cell" :class="cellClass(cell)"></div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    board: Array
  },
  methods: {
    handleClick(column) {
      this.$emit('dropPiece', column)
    },
    cellClass(cell) {
      return {
        empty: cell === 'EMPTY',
        human: cell === 'HUMAN',
        computer: cell === 'COMPUTER'
      }
    }
  }
}
</script>

<style scoped>
.board {
  display: flex;
  flex-direction: row;
}
.column {
  display: flex;
  flex-direction: column-reverse;
  cursor: pointer;
}
.cell {
  width: 50px;
  height: 50px;
  border: 1px solid black;
}
.empty {
  background-color: white;
}
.human {
  background-color: red;
}
.computer {
  background-color: black;
}
</style>
