<template>
  <svg width="30" height="15">
    <polygon v-if="isUp" points="0 10, 20 10, 10 0" style="fill: green" />
    <polygon v-else points="0 0, 20 0, 10 10" style="fill: red" />
  </svg>
</template>

<script>
export default {
  name: "ArrowTrend",
  props: {
    currentV: {
      type: Number,
      default: 0,
    },
    lastV: {
      type: Number,
      default: 0,
    },
  },
  data() {
    return {
      isUp: true,
    };
  },
  methods: {
    handleArrow() {
      this.isUp = this.currentV >= this.lastV;
    },
  },
  watch: {
    $props: {
      handler() {
        this.handleArrow();
      },
      deep: true,
    },
  },
};
</script>

<style scoped>
</style>