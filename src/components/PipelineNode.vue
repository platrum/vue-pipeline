<template>
  <g :transform="'translate('+x+','+y+')'" :class="nodeClass" :cursor="canHoverSteps ? 'pointer' : 'default'" @click="handleClick" @mouseenter="handleMouseEnter" @mouseleave="handleMouseLeave">

    <pipeline-node-start v-if="status=='start'" :label="label" :nodeColor="nodeColor"/>
    <pipeline-node-end v-if="status=='end'" :label="label" />

    <g v-if="status!=='start' && status!=='end'">
      <g>
        <text :x="getText().x" :y="getText().y" class="pipeline-node-label" :class="{'label-shadow': nodeColor}">{{getText().text}}</text>
        <title>{{label}}</title>
      </g>
      <g v-if="getHint().hint && isVisibleHint">
        <text :x="getHint().x" :y="getHint().y" class="pipeline-node-hint" :class="{ active: status=='running' }">{{getHint().hint}}</text>
        <title v-if="labelHint">{{labelHint}}</title>
      </g>
      <g class="svgResultStatus" :id="`node-${index}`">
        <circle cx="0" cy="0" r="12" :class="'circle-bg '+status" :style="getNodeStyle()"></circle>
        <g class="result-status-glyph">
          <polygon fill="white" v-if="status=='failure'"
                   points="4.67 -3.73 3.73 -4.67 0 -0.94 -3.73 -4.67 -4.67 -3.73 -0.94 0 -4.67 3.73 -3.73 4.67 0 0.94 3.73 4.67 4.67 3.73 0.94 0">
          </polygon>
        </g>
        <g class="result-status-glyph" v-if="status=='success'">
          <polygon fill="white" points="-2.00 2.80 -4.80 0.00 -5.73 0.933 -2.00 4.67 6.00 -3.33 5.07 -4.27">
          </polygon>
        </g>
        <g class="result-status-glyph" v-if="status=='paused'">
          <polygon points="-4,-4.65 -4,4.65 -4,4.65 -1.5,4.65 -1.5,-4.65" />
          <polygon points="4,-4.65 1.5,-4.65 1.5,-4.65 1.5,4.65 4,4.65" />
        </g>
        <g class="result-status-glyph" v-if="status=='unstable'">
          <polygon points="-1 -5 1 -5 1 1 -1 1" />
          <polygon points="-1 3 1 3 1 5 -1 5" />
        </g>
        <g class="result-status-glyph running" v-if="status=='running'">
          <path transform="scale(0.03 0.03) translate(-514,-510)"
                d="M604.16 1003.52V898.458A410.317 410.317 0 0 0 898.458 604.16h105.062a512.614 512.614 0 0 1-399.36 399.36z m-204.8 0A512.614 512.614 0 0 1 0 604.16h105.062A410.317 410.317 0 0 0 399.36 898.458v105.062z m204.8-898.458V0a512.614 512.614 0 0 1 399.36 399.36H898.458A410.317 410.317 0 0 0 604.16 105.062z m-204.8 0A410.317 410.317 0 0 0 105.062 399.36H0A512.614 512.614 0 0 1 399.36 0v105.062zM512 665.6a153.6 153.6 0 1 0 0-307.2 153.6 153.6 0 0 0 0 307.2z m0 102.4a256 256 0 1 1 0-512 256 256 0 0 1 0 512z"
                p-id="10371"></path>
        </g>
      </g>
    </g>

    <!-- high light -->
    <circle r="19" class="pipeline-node-hittarget" fill-opacity="0" stroke="none">
    </circle>
    <g class="pipeline-selection-highlight" v-if="selected">
      <circle class="white-highlight" r="13" stroke-width="10"></circle>
      <circle r="15" stroke-width="2"></circle>
    </g>

  </g>
</template>
<script>
import PipelineNodeStart from './PipelineNodeStart'
import PipelineNodeEnd from './PipelineNodeEnd'
export default {
  components: {
    PipelineNodeStart,
    PipelineNodeEnd
  },
  props: {
    index: {
      type: Number
    },
    hint: {
      type: String
    },
    status: {
      type: String
    },
    label: {
      type: String
    },
    labelHint: {
      type: String
    },
    x: {
      type: Number
    },
    y: {
      type: Number
    },
    node: {
      type: Object
    },
    selected: {
      type: Boolean,
      default: false
    },
    isVisibleHint: {
      type: Boolean,
      default: false
    },
    nodeColor: String,
    canHoverSteps: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      nodeClass: 'pipeline-node'
    }
  },
  methods: {
    getText() {
      let maxLength = 14
      let text =
        this.label.length > maxLength
          ? this.label.substring(0, maxLength) + '...'
          : this.label
      let width = this.getTextWidth(text, '12px PT Root UI')
      return {
        x: -width / 2,
        y: -20,
        text,
      }
    },
    getHint() {
      if (!this.hint) {
        return {
          x: 0,
          y: 0,
          hint: '',
        }
      }
      let maxLength = 20
      let hint =
        this.hint.length > maxLength
          ? this.hint.substring(0, maxLength) + '...'
          : this.hint
      let width = this.getTextWidth(hint, '10px PT Root UI')
      return {
        x: -width / 2,
        y: 25,
        hint
      }
    },
    handleClick() {
      //Убрано за ненадобностью, но мб когда-то понадобится в будущем
      // this.nodeClass = 'pipeline-node-selected'
      // if (this.status != 'start' && this.status != 'end') {
      //   this.$emit('click', this.index, this.node)
      // }
    },
    handleMouseEnter() {
      if (!this.canHoverSteps) {
        return;
      }
      this.nodeClass = 'pipeline-node-selected'
      let rect = null;
      if (this.status!=='start' && this.status!=='end') {
        rect = document.getElementById(`node-${this.index}`).getBoundingClientRect();
      }
      this.$emit('mouseenter', this.node, rect)
    },
    handleMouseLeave() {
      this.nodeClass = 'pipeline-node'
      this.$emit('mouseleave', this.node)
    },
    getTextWidth(text, font) {
      // re-use canvas object for better performance
      var canvas =
        this.getTextWidth.canvas ||
        (this.getTextWidth.canvas = document.createElement('canvas'))
      var context = canvas.getContext('2d')
      context.font = font
      var metrics = context.measureText(text)
      return metrics.width
    },
    getNodeStyle() {
      if (this.nodeColor) {
        return {
          fill: this.nodeColor,
        }
      }
    },
  }
}
</script>
<style lang="css">
.pipeline-node-selected .svgResultStatus > circle {
  stroke: none;
}

.svgResultStatus {
  transform: translateZ(0);
}
.svgResultStatus > circle {
  stroke: white;
  stroke-width: 2px;
}
.svgResultStatus > circle.success {
  fill: #8cc04f;
}
.svgResultStatus > circle.failure {
  fill: #d54c53;
}
.svgResultStatus > circle.unstable {
  fill: #f6b44b;
}
.svgResultStatus > circle.aborted {
  fill: #949393;
}
.svgResultStatus > circle.paused {
  fill: #24b0d5;
}
.svgResultStatus > circle.unknown {
  fill: #d54cc4;
}
.svgResultStatus > circle.running {
  fill: #fff;
}

.pipeline-node-label {
  font: 12px sans-serif;
  font-size: 12px;
  /* fill: red; */
  width: 100px;
  overflow: hidden;
  overflow-wrap: break-word;
  /* top: 140px; */
  /* left: 339px; */
  position: absolute;
  width: 89px;
  /* max-height: 39px; */
  text-align: center;
}
.pipeline-node-hint {
  font: 12px sans-serif;
  font-size: 10px;
  width: 100px;
  overflow: hidden;
  overflow-wrap: break-word;
  position: absolute;
  text-align: center;
}
.pipeline-node-hint.active {
  color: #8CCC4F;
  fill: #8CCC4F;
}
.running {
  fill: #8ccc4f;
  animation: rotating 2s linear infinite;
  animation-iteration-count: infinite;
}
.label-shadow {
  text-shadow: 0 2px 8px rgba(255, 255, 255, 0.75);
}
</style>
