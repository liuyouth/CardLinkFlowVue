<template>
  <div class="properties-panel" :class="{ show: selectedElement }">
    <div class="panel-header">
      <span class="panel-title">属性设置</span>
      <button class="close-button" @click="closePanel">×</button>
    </div>
    
    <div class="panel-content">
      <!-- 节点属性 -->
      <div v-if="isNodeSelected" class="property-section node-properties">
        <h3>节点属性</h3>
        <div class="property-item">
          <label>节点标识</label>
          <input
            type="text"
            :value="selectedElement.id"
            class="property-input"
            readonly
          >
        </div>
        <div class="property-item">
          <label>标签</label>
          <input
            type="text"
            v-model="nodeLabel"
            class="property-input"
            @input="updateNodeProperty('label')"
          >
        </div>
        <div class="property-item">
          <label>类型</label>
          <select
            v-model="nodeType"
            class="property-input"
            @change="updateNodeProperty('type')"
          >
            <option value="resource">资源</option>
            <option value="ai-model">AI模型</option>
            <option value="result">结果</option>
          </select>
        </div>
        <div class="property-item">
          <label>资源URL</label>
          <input
            type="text"
            v-model="nodeUrl"
            class="property-input"
            @input="updateNodeProperty('url')"
          >
        </div>
      </div>

      <!-- 连接线属性 -->
      <div v-if="isEdgeSelected" class="property-section edge-properties">
        <h3>连接线属性</h3>
        <div class="property-item">
          <label>连线标识</label>
          <input
            type="text"
            :value="selectedElement.id"
            class="property-input"
            readonly
          >
        </div>
        <div class="property-item">
          <label>连线类型</label>
          <select
            v-model="edgeType"
            class="property-input"
            @change="updateEdgeProperty('type')"
          >
            <option value="default">默认连线</option>
            <option value="resource">资源连线</option>
            <option value="result">结果连线</option>
          </select>
        </div>
        <div class="property-item">
          <label>动画速度</label>
          <input
            type="range"
            v-model="edgeSpeed"
            min="1"
            max="50"
            class="property-input"
            @input="updateEdgeProperty('speed')"
          >
          <span class="speed-value">{{ edgeSpeed }}x</span>
        </div>
        <div class="property-item">
          <label>线条样式</label>
          <select
            v-model="edgeStyle"
            class="property-input"
            @change="updateEdgeProperty('style')"
          >
            <option value="solid">实线</option>
            <option value="dashed">虚线</option>
            <option value="dotted">点线</option>
          </select>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, watch } from 'vue'

export default {
  name: 'PropertiesPanel',
  
  props: {
    selectedElement: {
      type: Object,
      default: null
    }
  },

  setup(props, { emit }) {
    // 节点属性
    const nodeLabel = ref('')
    const nodeType = ref('')
    const nodeUrl = ref('')

    // 连接线属性
    const edgeType = ref('default')
    const edgeSpeed = ref(1)
    const edgeStyle = ref('solid')

    // 计算属性
    const isNodeSelected = computed(() => {
      return props.selectedElement?.type === 'node'
    })

    const isEdgeSelected = computed(() => {
      return props.selectedElement?.type === 'edge'
    })

    // 监听选中元素变化
    watch(() => props.selectedElement, (newVal) => {
      if (newVal?.type === 'node') {
        nodeLabel.value = newVal.label || ''
        nodeType.value = newVal.nodeType || 'resource'
        nodeUrl.value = newVal.url || ''
      } else if (newVal?.type === 'edge') {
        edgeType.value = newVal.edgeType || 'default'
        edgeSpeed.value = newVal.speed || 1
        edgeStyle.value = newVal.style || 'solid'
      }
    })

    // 更新节点属性
    const updateNodeProperty = (property) => {
      const update = {
        type: 'node',
        id: props.selectedElement.id,
        changes: {}
      }

      switch (property) {
        case 'label':
          update.changes.label = nodeLabel.value
          break
        case 'type':
          update.changes.nodeType = nodeType.value
          break
        case 'url':
          update.changes.url = nodeUrl.value
          break
      }

      emit('update', update)
    }

    // 更新连接线属性
    const updateEdgeProperty = (property) => {
      const update = {
        type: 'edge',
        id: props.selectedElement.id,
        changes: {}
      }

      switch (property) {
        case 'type':
          update.changes.edgeType = edgeType.value
          break
        case 'speed':
          update.changes.speed = parseInt(edgeSpeed.value)
          break
        case 'style':
          update.changes.style = edgeStyle.value
          break
      }

      emit('update', update)
    }

    // 关闭面板
    const closePanel = () => {
      emit('update:selectedElement', null)
    }

    return {
      nodeLabel,
      nodeType,
      nodeUrl,
      edgeType,
      edgeSpeed,
      edgeStyle,
      isNodeSelected,
      isEdgeSelected,
      updateNodeProperty,
      updateEdgeProperty,
      closePanel
    }
  }
}
</script>

<style scoped>
.properties-panel {
  position: fixed;
  right: 0;
  top: 0;
  width: 300px;
  height: 100vh;
  background: #2D2D2D;
  border-left: 1px solid rgba(255, 255, 255, 0.1);
  box-shadow: -2px 0 10px rgba(0, 0, 0, 0.2);
  z-index: 1000;
  transform: translateX(100%);
  transition: transform 0.3s ease;
}

.properties-panel.show {
  transform: translateX(0);
}

.panel-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.1);
}

.panel-title {
  font-size: 16px;
  font-weight: 500;
  color: rgba(255, 255, 255, 0.9);
}

.close-button {
  background: none;
  border: none;
  color: rgba(255, 255, 255, 0.6);
  font-size: 20px;
  cursor: pointer;
  padding: 4px 8px;
}

.panel-content {
  padding: 16px;
  overflow-y: auto;
  height: calc(100% - 60px);
}

.property-section {
  margin-bottom: 24px;
}

.property-section h3 {
  font-size: 14px;
  color: rgba(255, 255, 255, 0.7);
  margin-bottom: 16px;
}

.property-item {
  margin-bottom: 16px;
}

.property-item label {
  display: block;
  font-size: 12px;
  color: rgba(255, 255, 255, 0.5);
  margin-bottom: 8px;
}

.property-input {
  width: 100%;
  padding: 8px;
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 4px;
  color: #fff;
  font-size: 13px;
}

.property-input:focus {
  border-color: rgba(82, 255, 168, 0.5);
  outline: none;
}

.property-input[type="range"] {
  -webkit-appearance: none;
  height: 4px;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 2px;
}

.property-input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none;
  width: 16px;
  height: 16px;
  background: rgba(82, 255, 168, 0.8);
  border-radius: 50%;
  cursor: pointer;
}

.speed-value {
  font-size: 12px;
  color: rgba(255, 255, 255, 0.5);
  margin-left: 8px;
}

.property-input[readonly] {
  background: rgba(255, 255, 255, 0.02);
  cursor: not-allowed;
}
</style> 