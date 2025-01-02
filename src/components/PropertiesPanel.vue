<template>
  <div class="properties-panel" :class="{ show: selectedElement }">
    <div class="panel-header">
      <span class="panel-title">
        <i class="icon" :class="getPanelIcon"></i>
        {{ getPanelTitle }}
      </span>
      <div class="panel-actions">
        <button class="action-btn" @click="resetProperties" title="重置属性">
          <i class="icon reset-icon"></i>
        </button>
        <button class="close-button" @click="closePanel">×</button>
      </div>
    </div>
    
    <div class="panel-content">
      <!-- 节点属性 -->
      <div v-if="isNodeSelected" class="property-section node-properties">
        <div class="section-header">
          <i class="icon node-icon"></i>
          <h3>节点属性</h3>
        </div>
        
        <div class="property-group">
          <div class="property-item">
            <label>节点标识</label>
            <div class="input-with-copy">
              <input
                type="text"
                :value="selectedElement.id"
                class="property-input"
                readonly
              >
              <button class="copy-btn" @click="copyToClipboard(selectedElement.id)">
                复制
              </button>
            </div>
          </div>

          <div class="property-item">
            <label>标签</label>
            <div class="input-with-validation">
              <input
                type="text"
                v-model="nodeLabel"
                class="property-input"
                @input="validateAndUpdateNode('label')"
                :class="{ 'error': validationErrors.label }"
              >
              <span class="validation-error" v-if="validationErrors.label">
                {{ validationErrors.label }}
              </span>
            </div>
          </div>

          <div class="property-item">
            <label>类型</label>
            <select
              v-model="nodeType"
              class="property-input select-input"
              @change="updateNodeProperty('type')"
            >
              <option v-for="type in nodeTypes" 
                      :key="type.value" 
                      :value="type.value">
                {{ type.label }}
              </option>
            </select>
          </div>

          <div class="property-item" v-if="showUrlInput">
            <label>资源URL</label>
            <div class="input-with-validation">
              <input
                type="text"
                v-model="nodeUrl"
                class="property-input"
                @input="validateAndUpdateNode('url')"
                :class="{ 'error': validationErrors.url }"
              >
              <button class="test-btn" @click="testUrl" v-if="nodeUrl">
                测试链接
              </button>
              <span class="validation-error" v-if="validationErrors.url">
                {{ validationErrors.url }}
              </span>
            </div>
          </div>
        </div>

        <!-- 节点预览 -->
        <div class="preview-section">
          <label>节点预览</label>
          <div class="preview-box" :class="[nodeType, { active: isPreviewActive }]">
            <span class="preview-label">{{ nodeLabel || '未命名节点' }}</span>
            <div class="node-type-indicator" :class="nodeType"></div>
          </div>
        </div>
      </div>

      <!-- 连接线属性 -->
      <div v-if="isEdgeSelected" class="property-section edge-properties">
        <div class="section-header">
          <i class="icon edge-icon"></i>
          <h3>连接线属性</h3>
        </div>
        
        <div class="property-group">
          <div class="property-item">
            <label>连线类型</label>
            <select
              v-model="edgeType"
              class="property-input select-input"
              @change="updateEdgeProperty('type')"
            >
              <option v-for="type in edgeTypes" 
                      :key="type.value" 
                      :value="type.value">
                {{ type.label }}
              </option>
            </select>
          </div>

          <div class="property-item">
            <label>动画速度 ({{ edgeSpeed }}x)</label>
            <div class="slider-container">
              <input
                type="range"
                v-model="edgeSpeed"
                min="1"
                max="50"
                class="property-input slider"
                @input="updateEdgeProperty('speed')"
              >
              <div class="slider-values">
                <span>慢</span>
                <span>快</span>
              </div>
            </div>
          </div>
        </div>

        <!-- 连线预览 -->
        <div class="preview-section">
          <label>连线预览</label>
          <div class="edge-preview">
            <div class="preview-line" 
                 :class="[edgeStyle, edgeType]"
                 :style="{ animationDuration: `${50/edgeSpeed}s` }">
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, watch } from 'vue'
import { useToast } from '@/composables/useToast'

export default {
  name: 'PropertiesPanel',
  
  props: {
    selectedElement: {
      type: Object,
      default: null
    }
  },

  setup(props, { emit }) {
    const { showToast } = useToast()
    const validationErrors = ref({})
    const isPreviewActive = ref(false)

    // 节点类型选项
    const nodeTypes = [
      { value: 'resource', label: '资源节点' },
      { value: 'ai-model', label: 'AI模型' },
      { value: 'result', label: '结果节点' }
    ]

    // 连线类型选项
    const edgeTypes = [
      { value: 'default', label: '默认连线' },
      { value: 'resource', label: '资源连线' },
      { value: 'result', label: '结果连线' }
    ]

    // 验证函数
    const validateNodeProperties = (property, value) => {
      const errors = {}
      
      switch (property) {
        case 'label':
          if (!value?.trim()) {
            errors.label = '标签不能为空'
          }
          break
        case 'url':
          if (value && !isValidUrl(value)) {
            errors.url = 'URL格式不正确'
          }
          break
      }

      return errors
    }

    // URL验证
    const isValidUrl = (url) => {
      try {
        new URL(url)
        return true
      } catch {
        return false
      }
    }

    // 测试URL连接
    const testUrl = async () => {
      try {
        const response = await fetch(nodeUrl.value)
        if (response.ok) {
          showToast('URL可以访问', 'success')
        } else {
          showToast('URL无法访问', 'error')
        }
      } catch {
        showToast('URL无法访问', 'error')
      }
    }

    // 重置属性
    const resetProperties = () => {
      if (window.confirm('确定要重置所有属性吗？')) {
        if (isNodeSelected.value) {
          nodeLabel.value = ''
          nodeType.value = 'resource'
          nodeUrl.value = ''
        } else {
          edgeType.value = 'default'
          edgeSpeed.value = 1
          edgeStyle.value = 'solid'
        }
        validationErrors.value = {}
      }
    }

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

    // 添加新的计算属性
    const getPanelTitle = computed(() => {
      if (props.selectedElement?.type === 'node') {
        return '节点属性设置'
      }
      return '连接线属性设置'
    })

    const getPanelIcon = computed(() => {
      return props.selectedElement?.type === 'node' ? 'node-icon' : 'edge-icon'
    })

    // 添加复制功能
    const copyToClipboard = async (text) => {
      try {
        await navigator.clipboard.writeText(text)
        // 这里可以添加一个提示消息
      } catch (err) {
        console.error('复制失败:', err)
      }
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
      closePanel,
      getPanelTitle,
      getPanelIcon,
      copyToClipboard,
      nodeTypes,
      edgeTypes,
      validationErrors,
      isPreviewActive,
      validateNodeProperties,
      testUrl,
      resetProperties,
      showUrlInput: computed(() => nodeType.value === 'resource')
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
  background: #1E1E1E;
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

.section-header {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
}

.icon {
  width: 16px;
  height: 16px;
  margin-right: 8px;
  background-size: contain;
}

.node-icon {
  background-image: url('../assets/node-icon.svg');
}

.edge-icon {
  background-image: url('../assets/edge-icon.svg');
}

.input-with-copy {
  display: flex;
  gap: 8px;
}

.copy-btn {
  padding: 4px 8px;
  background: rgba(82, 255, 168, 0.1);
  border: 1px solid rgba(82, 255, 168, 0.2);
  border-radius: 4px;
  color: rgba(82, 255, 168, 0.8);
  cursor: pointer;
  font-size: 12px;
}

.copy-btn:hover {
  background: rgba(82, 255, 168, 0.2);
}

.node-preview {
  margin-top: 20px;
  padding: 16px;
  background: rgba(255, 255, 255, 0.03);
  border-radius: 8px;
}

.preview-box {
  margin-top: 8px;
  padding: 12px;
  border-radius: 6px;
  background: rgba(82, 255, 168, 0.1);
  border: 1px solid rgba(82, 255, 168, 0.2);
  text-align: center;
}

.preview-box.ai-model {
  background: rgba(64, 156, 255, 0.1);
  border-color: rgba(64, 156, 255, 0.2);
}

.edge-preview {
  margin: 20px 0;
  padding: 20px;
  background: rgba(255, 255, 255, 0.03);
  border-radius: 8px;
}

.preview-line {
  height: 2px;
  background: currentColor;
  position: relative;
}

.preview-line.dashed {
  border-top: 2px dashed currentColor;
  background: none;
}

.preview-line.dotted {
  border-top: 2px dotted currentColor;
  background: none;
}

/* 添加过渡动画 */
.property-section {
  transition: all 0.3s ease;
}

.property-input {
  transition: border-color 0.2s ease;
}

/* 改进滚动条样式 */
.panel-content::-webkit-scrollbar {
  width: 6px;
}

.panel-content::-webkit-scrollbar-track {
  background: rgba(255, 255, 255, 0.02);
}

.panel-content::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 3px;
}

.panel-content::-webkit-scrollbar-thumb:hover {
  background: rgba(255, 255, 255, 0.2);
}

.panel-actions {
  display: flex;
  gap: 8px;
  align-items: center;
}

.action-btn {
  background: none;
  border: none;
  padding: 4px;
  color: rgba(255, 255, 255, 0.6);
  cursor: pointer;
  border-radius: 4px;
}

.action-btn:hover {
  background: rgba(255, 255, 255, 0.1);
}

.property-group {
  background: rgba(255, 255, 255, 0.03);
  border-radius: 8px;
  padding: 16px;
  margin-bottom: 20px;
}

.input-with-validation {
  position: relative;
}

.validation-error {
  color: #ff4d4f;
  font-size: 12px;
  margin-top: 4px;
  display: block;
}

.property-input.error {
  border-color: #ff4d4f;
}

.test-btn {
  position: absolute;
  right: 8px;
  top: 50%;
  transform: translateY(-50%);
  padding: 4px 8px;
  background: rgba(82, 255, 168, 0.1);
  border: none;
  border-radius: 4px;
  color: rgba(82, 255, 168, 0.8);
  cursor: pointer;
}

.slider-container {
  position: relative;
}

.slider-values {
  display: flex;
  justify-content: space-between;
  font-size: 12px;
  color: rgba(255, 255, 255, 0.5);
  margin-top: 4px;
}

.preview-section {
  background: rgba(255, 255, 255, 0.03);
  border-radius: 8px;
  padding: 16px;
}

.node-type-indicator {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  position: absolute;
  right: 8px;
  top: 50%;
  transform: translateY(-50%);
}

.node-type-indicator.resource {
  background: #52ffa8;
}

.node-type-indicator.ai-model {
  background: #409cff;
}

.node-type-indicator.result {
  background: #ff4d4f;
}

@keyframes flowAnimation {
  0% { background-position: 0% 0%; }
  100% { background-position: 100% 0%; }
}

.preview-line {
  background: linear-gradient(90deg, 
    rgba(255,255,255,0.1) 25%, 
    rgba(255,255,255,0.3) 50%, 
    rgba(255,255,255,0.1) 75%
  );
  background-size: 200% 100%;
  animation: flowAnimation 2s linear infinite;
}
</style> 