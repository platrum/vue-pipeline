<template>
  <g :class="'pipeline-line '+'weight'+weight " @mouseenter="handleMouseEnter" @mouseleave="handleMouseLeave">
    <path stroke-width="3.5" :d="this.path" fill="none" :marker-end="getMarkerEnd()" :style="getLineStyle()"> </path>
  </g>
</template>

<script>
export default {
  props: {
    path: {
      type: String
    },
    weight: {
      type: Number,
      default: 0
    },
    showArrow: {
      type: Boolean,
      default: false
    },
    lineColor: String,
    lineData: {
      type: Object,
      default() {
        return {}
      }
    },
  },
  data() {
    return {
      ystep: this.y1 + 30
    };
  },
  computed: {},
  methods: {
    getMarkerEnd() {
      if (this.showArrow) {
        return `url(#idArrow${this.weight})`;
      }
    },
    color() {
      switch (this.weight) {
        case 0:
          return "#ddd";
        case 1:
          return "#949393";
        case 2:
          return "#8cc04f";
        default:
          break;
      }
    },
    getLineStyle() {
      if (this.lineColor) {
        return {
          stroke: this.lineColor,
        }
      }
    },
    handleMouseEnter(e) {
      if (this.lineData.from.status === 'start') {
        return;
      }
      const positionCoords  = {
        layerX: e.layerX,
        layerY: e.layerY,
        offsetX: e.offsetX,
        offsetY: e.offsetY,
        pageX: e.pageX,
        pageY: e.pageY,
        x: e.x,
        y: e.y,
      };
      this.$emit('mouseenter', {
        lineData: this.lineData,
        position: positionCoords,
      });
    },
    handleMouseLeave() {
      this.$emit('mouseleave', {
        lineData: this.lineData,
      });
    },
  }
};
</script>
<style lang="css">
.pipeline-connector {
  stroke: #949393;
}
</style>
