<template>
  <g :class="'pipeline-line '+'weight'+weight " :id="path">
    <path
      stroke-width="4.5"
      :d="this.path"
      fill="none"
      :marker-end="getMarkerEnd()"
      :style="getLineStyle()"
      :class="{'line-hovered': isHovered}"
      class="pipeline-line"
    />
    <path
      stroke-width="8"
      :d="this.path"
      fill="none"
      :marker-end="getMarkerEnd()"
      :style="{opacity: 0, 'pointer-events': 'visible', 'cursor': 'pointer'}"
      @mouseenter="handleMouseEnter"
      @mouseleave="handleMouseLeave"
    />
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
      ystep: this.y1 + 30,
      isHovered: false,
    };
  },
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

      return {
        'pointer-events': 'none',
      }
    },
    handleMouseEnter(e) {
      if (this.lineData.from.status === 'start') {
        return;
      }
      const node = document.getElementById(this.path).getBBox();
      const positionCoords  = {
        layerX: e.layerX,
        layerY: e.layerY,
        offsetX: e.offsetX,
        offsetY: e.offsetY,
        pageX: e.pageX,
        pageY: e.pageY,
        node: node,
        x: e.x,
        y: e.pageY,
      };

      this.$emit('mouseenter', {
        lineData: this.lineData,
        position: positionCoords,
      });
      this.isHovered = true;
    },
    handleMouseLeave() {
      this.$emit('mouseleave', {
        lineData: this.lineData,
      });
      this.isHovered = false;
    },
    getYCoordinate(layerY, nodeY, y, nodeHeight) {
      if (layerY === nodeY) {
        return y + nodeHeight;
      }
      if (layerY - nodeY > 0) {
        const diff = layerY - nodeY;
        if (nodeHeight > 0) {
          return y - diff + nodeHeight;
        }

        return y - diff;
      }
      if (nodeY - layerY > 0) {
        const diff = nodeY - layerY;
        if (nodeHeight > 0) {
          return y + (nodeHeight - diff);
        }

        return y + diff;
      }
    },
  }
};
</script>
<style lang="css">
.pipeline-connector {
  stroke: #949393;
}

.pipeline-line {
  transition: 0.2s filter linear;
}
.line-hovered {
  filter: brightness(0.65);
}
</style>
