<template>
  <svg 
    class="edge" 
    :class="{ 
      'selected': isSelected,
      'preview': isPreview 
    }" 
    :data-type="edge.type"
  >
    <path
      :d="pathData"
      class="edge-path"
      @click="selectEdge"
      @contextmenu.prevent="showContextMenu"
    />
    <text
      v-if="edge.label"
      :x="labelPosition.x"
      :y="labelPosition.y"
      class="condition-label"
    >
      {{ edge.label }}
    </text>
  </svg>
</template>

<script>
import { computed } from 'vue'

export default {
  name: 'FlowEdge',
  
  props: {
    edge: {
      type: Object,
      required: true
    },
    isSelected: {
      type: Boolean,
      default: false
    },
    isPreview: {
      type: Boolean,
      default: false
    }
  },

  setup(props, { emit }) {
    // 计算连接线路径
    const pathData = computed(() => {
      const { source, target } = props.edge
      
      // 确保坐标值存在且为数字
      const startX = Number(source.x) || 0
      const startY = Number(source.y) || 0
      const endX = Number(target.x) || 0
      const endY = Number(target.y) || 0
      
      // 计算控制点
      const dx = Math.abs(endX - startX)
      const offset = Math.min(dx * 0.5, 100) // 限制最大弯曲程度
      
      // 根据连接点的位置调整控制点
      const sourceOffset = source.position === 'right' ? offset : -offset
      const targetOffset = target.position === 'right' ? offset : -offset
      
      const controlPoint1X = startX + sourceOffset
      const controlPoint1Y = startY
      const controlPoint2X = endX + targetOffset
      const controlPoint2Y = endY
      
      return `M ${startX} ${startY} C ${controlPoint1X} ${controlPoint1Y}, ${controlPoint2X} ${controlPoint2Y}, ${endX} ${endY}`
    })

    // 计算标签位置
    const labelPosition = computed(() => {
      const { source, target } = props.edge
      return {
        x: source.x + (target.x - source.x) / 2,
        y: source.y + (target.y - source.y) / 2 - 10
      }
    })

    // 选择连接线
    const selectEdge = () => {
      emit('select', props.edge)
    }

    // 显示右键菜单
    const showContextMenu = (event) => {
      emit('context-menu', {
        edge: props.edge,
        position: {
          x: event.clientX,
          y: event.clientY
        }
      })
    }

    return {
      pathData,
      labelPosition,
      selectEdge,
      showContextMenu
    }
  }
}
</script>

<style scoped>
.edge {
  position: absolute;
  width: 100%;
  height: 100%;
  pointer-events: none;
  top: 0;
  left: 0;
}

.edge path {
  stroke: rgba(255, 255, 255, 0.15);
  stroke-width: 1.5px;
  fill: none;
  transition: all 0.3s ease;
  pointer-events: auto;
  cursor: pointer;
  stroke-dasharray: 4 4;
}

.edge.preview path {
  stroke: rgba(82, 255, 168, 0.3);
  stroke-width: 2px;
  stroke-dasharray: 4 4;
  pointer-events: none;
  animation: previewDash 1s linear infinite;
}

.edge[data-type="result"] path {
  stroke: rgba(82, 255, 168, 0.3);
  stroke-width: 2px;
  stroke-dasharray: none;
}

.edge.selected path {
  stroke: rgba(82, 255, 168, 0.5);
  stroke-width: 2px;
}

.condition-label {
  font-size: 12px;
  fill: rgba(255, 255, 255, 0.5);
  pointer-events: none;
}

@keyframes flowAnimation {
  from {
    stroke-dashoffset: 8;
  }
  to {
    stroke-dashoffset: 0;
  }
}

@keyframes previewDash {
  from {
    stroke-dashoffset: 8;
  }
  to {
    stroke-dashoffset: 0;
  }
}

.edge path {
  animation: flowAnimation 1s linear infinite;
}

.edge[data-type="result"] path {
  animation: none;
}
</style> 