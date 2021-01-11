<template>
  <div class="tile" :style="tile.styles" @click="move" :class="{ empty: tile.isEmpty }">
    <span class="indexBox" v-if="!tile.isEmpty">{{ parseInt(tile.position) + 1 }}</span>
  </div>
</template>

<script>
export default {
  props: ['tile'],
  methods: {
    move () {
      this.$emit('moving', this.tile)
    }
  }
}
</script>

<style lang="scss">
.tile {
  border: 2px solid #000;
  &.empty {
    box-shadow: rgba(0, 0, 0, 0.5) 0px 5px 10px inset;
    &:hover {
      border-color: #111;
      cursor: default;
    }
  }
  &:hover {
    border-color: #f00;
    cursor: pointer;
  }
}
.indexBox {
  font-size: 34px;
  padding: 10px;
  color: #000;
  -webkit-text-stroke: 1.5px black;
  -webkit-text-fill-color: #f3f5f7;
}
</style>