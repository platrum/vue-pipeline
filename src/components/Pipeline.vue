
<template>
  <svg class="pipeline" :width="width" :height="height">
    <defs>
      <marker :id="'idArrow'+i" v-for="i in [0,1,2,3,4,5]" :key="'arrow'+i" :class="'weight'+i" viewBox="0 0 20 20"
              refX="13" refY="10" markerUnits="strokeWidth" markerWidth="3" markerHeight="10" orient="auto">
        <path d="M 0 0 L 20 10 L 0 20 z" />
      </marker>
    </defs>

    <pipeline-line
      v-for="(item,index) in lineList"
      :key="'line'+index"
      :showArrow="showArrow"
      :path="item.path"
      :weight="item.weight"
      :lineStyle="lineStyle"
      :lineColor="item.color"
      :lineData="item"
      :can-hover-lines="canHoverLines"
      @mouseenter="handleLineMouseEnter"
      @mouseleave="$emit('line-mouseleave', $event)"
    />

    <pipeline-node
      v-for="(item,idx) in nodeList"
      :key="'node'+idx"
      :hint="item.hint"
      :status="item.status"
      :is-visible-hint="isVisibleHint"
      :label="item.name"
      :label-hint="item.labelHint"
      :x="item.x"
      :y="item.y"
      :node="item"
      :index="idx"
      :selected="selectedList[idx]"
      :nodeColor="item.color"
      :can-hover-steps="canHoverSteps"
      @click="handleClick"
      @mouseenter="handleStepMouseEnter"
      @mouseleave="handleStepMouseLeave"
    />
  </svg>
</template>
<script>
import PipelineNode from "./PipelineNode";
import PipelineLine from "./PipelineLine";
import { Pipeline } from "./service";

export default {
  components: {
    PipelineNode,
    PipelineLine
  },
  props: {
    // 第一个点的起始位置
    x: {
      type: Number,
      default: 50
    },
    //第一个点的y座标
    y: {
      type: Number,
      default: 55
    },
    // x轴上相邻两个点之间的距离
    xstep: {
      type: Number,
      default: 120
    },
    // y 轴上相邻两个点之间的距离
    ystep: {
      type: Number,
      default: 50
    },
    data: {
      type: Array,
      default: () => []
    },
    lineStyle: {
      type: String,
      default: "default"
    },
    showArrow: {
      type: Boolean,
      default: false
    },
    isVisibleHint: {
      type: Boolean,
      default: false
    },
    linesColorData: Array,
    stepsColorData: Array,
    startLineColor: {
      type: String,
      default: '#E4E5EC'
    },
    startNodeColor: {
      type: String,
      default: '#E4E5EC'
    },
    colored: {
      type: Boolean,
      default: false,
    },
    canHoverLines: {
      type: Boolean,
      default: false,
    },
    canHoverSteps: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      nodeList: [],
      width: 300,
      height: 300,
      lineList: [],
      selectedList: [],
      service: {}
    };
  },
  computed: {
    dataLength() {
      return this.data.length;
    }
  },
  watch: {
    dataLength() {
      this.render();
    }
  },
  methods: {
    handleClick(index, node) {
      this.selectedList.fill(false, 0, this.nodeList.length);
      this.$set(this.selectedList, index, true);
      this.selectedList[index] = true;
      this.$emit("select", node);
    },
    handleStepMouseEnter(node, rect) {
      this.$emit("step-mouseenter", node, rect);
    },
    handleStepMouseLeave(node) {
      this.$emit("step-mouseleave", node);
    },
    render() {
      this.service = new Pipeline(
        this.data,
        this.x,
        this.y,
        this.xstep,
        this.ystep,
        this.lineStyle
      );
      if (this.service.hasCircle()) {
        throw new Error("Error data, The graph should not contain any circle!");
      }
      this.service.calculateAllPosition();
      // this.service.optimize();
      this.nodeList = this.service.nodes;
      this.lineList = this.service.getLines();
      if (this.colored) {
        this.getColorNodeList();
        this.getColorLinesList();
      } else {
        this.clearNodeListColor();
        this.clearLinesListColor();
      }
      this.width = this.service.width;
      this.height = this.service.height;
    },
    getColorNodeList() {
      this.nodeList.forEach(node => {
        if (node.step_key === 'start') {
          node.color = this.startNodeColor;
          return;
        }
        const coloredStepData = this.stepsColorData.find(step => step.step_key === node.step_key);
        if (coloredStepData !== undefined) {
          node.color = coloredStepData.color;
        }
      });
    },
    getColorLinesList() {
      this.lineList.forEach(line => {
        if (line.from.step_key === 'start') {
          line.color = this.startNodeColor;
          return;
        }
        const coloredLineData = this.getColoredLineData(line);
        if (coloredLineData !== undefined) {
          line.color = coloredLineData.color;
        }
      })
    },
    getColoredLineData(line) {
      const coloredLineData = this.linesColorData.find(item => (item.from_step_key === line.from.step_key && item.to_step_key === line.to.step_key));
      const reversedColoredLineData = this.linesColorData.find(item => (item.to_step_key === line.from.step_key && item.from_step_key === line.to.step_key));

      return coloredLineData === undefined ? reversedColoredLineData : coloredLineData;
    },
    clearNodeListColor() {
      this.nodeList.forEach(node => {
        if (node.hasOwnProperty('color')) {
          delete node.color;
        }
      });
    },
    clearLinesListColor() {
      this.lineList.forEach(line => {
        if (line.hasOwnProperty('color')) {
          delete line.color;
        }
      });
    },
    handleLineMouseEnter(data) {
      const lineIndex = this.lineList.findIndex(line => line.path === data.lineData.path);
      this.lineList.splice(lineIndex, 1);
      this.lineList.push(data.lineData);
      this.$emit('line-mouseenter', data);
    },
  },
  mounted() {
    this.render();
    this.selectedList.fill(false, 0, this.nodeList.length);
  }
};
</script>

<style >
.pipeline {
  /* transform: rotate(90deg) */
}

.pipeline .weight0 {
  fill: #ddd;
  stroke: #ddd;
}

.pipeline .weight1 {
  fill: #949393;
  stroke: #949393;
}

.pipeline .weight2 {
  fill: #8cc04f;
  stroke: #8cc04f;
}

/* .pipeline .pipeline-node{
  transform: rotate(90deg)
} */
.pipeline-node-terminal {
  fill: #949393;
}
.pipeline-connector-skipped {
  stroke: #949393;
  stroke-opacity: 0.25;
}
.pipeline-small-label {
  font-size: 80%;
}
.pipeline-big-label.selected,
.pipeline-small-label.selected {
  font-weight: bold;
}
.pipeline-selection-highlight circle {
  fill: none;
  stroke: #4a90e2;
}
.pipeline-selection-highlight circle.white-highlight {
  stroke: white;
}

.pipeline-node-terminal {
  fill: #949393;
}
.svgResultStatus.no-background .circle-bg {
  opacity: 0;
}

.jdl-table td .svgResultStatus {
  vertical-align: middle;
}

.pipeline-big-label.selected,
.pipeline-small-label.selected {
  font-weight: bold;
}
</style>
