<template>
  <div
    class="flow-node"
    :class="{
      'dragging': isDragging,
      'selected': isSelected,
      'resizing': isResizing
    }"
    :style="{
      left: `${node.position.x}px`,
      top: `${node.position.y}px`,
      width: node.width ? `${node.width}px` : 'auto',
      height: node.height ? `${node.height}px` : 'auto'
    }"
    @mousedown="startDrag"
    @click="selectNode"
  >
    <!-- AI按钮 -->
    <button class="ai-button" @click.stop="handleAI">
      <svg class="ai-icon" viewBox="0 0 24 24" width="14" height="14">
        <path fill="currentColor" d="M12,2A10,10 0 0,1 22,12A10,10 0 0,1 12,22A10,10 0 0,1 2,12A10,10 0 0,1 12,2M12,4A8,8 0 0,0 4,12A8,8 0 0,0 12,20A8,8 0 0,0 20,12A8,8 0 0,0 12,4M12,6A6,6 0 0,1 18,12A6,6 0 0,1 12,18A6,6 0 0,1 6,12A6,6 0 0,1 12,6M12,8A4,4 0 0,0 8,12A4,4 0 0,0 12,16A4,4 0 0,0 16,12A4,4 0 0,0 12,8Z"/>
      </svg>
      AI
    </button>

    <div class="node-content">
      <textarea
        v-model="content"
        class="node-textarea"
        @input="updateContent"
        ref="textarea"
        :style="{
          height: node.height ? `${node.height - 24}px` : 'auto'
        }"
      ></textarea>
    </div>

    <!-- 连接点 -->
    <div
      v-for="port in ports"
      :key="port.position"
      class="node-port"
      :class="[port.position]"
      @mousedown.stop="startConnection(port)"
      @mouseup.stop="endConnection(port)"
    ></div>

    <!-- 大小调整手柄 -->
    <div 
      class="resize-handle"
      @mousedown.stop="startResize"
    ></div>
  </div>
</template>

<script>
import { ref, computed, watch } from 'vue'

export default {
  name: 'FlowNode',
  
  props: {
    node: {
      type: Object,
      required: true
    },
    isSelected: {
      type: Boolean,
      default: false
    }
  },

  setup(props, { emit }) {
    const isDragging = ref(false)
    const isResizing = ref(false)
    const content = ref(props.node.content || '')
    const textarea = ref(null)
    let startPos = { x: 0, y: 0 }
    let startSize = { width: 0, height: 0 }

    // 监听节点内容变化
    watch(() => props.node.content, (newContent) => {
      if (newContent !== content.value) {
        content.value = newContent
      }
    })

    // 更新内容
    const updateContent = () => {
      emit('update', {
        id: props.node.id,
        content: content.value
      })
    }

    // 定义连接点
    const ports = computed(() => [
      { position: 'right', type: 'output' },
      { position: 'left', type: 'input' }
    ])

    // 开始拖动
    const startDrag = (event) => {
      // 如果是在文本框中点击，不启动拖动
      if (event.target === textarea.value) {
        return
      }
      
      isDragging.value = true
      startPos = {
        x: event.clientX - props.node.position.x,
        y: event.clientY - props.node.position.y
      }

      const handleMouseMove = (e) => {
        const newPosition = {
          x: e.clientX - startPos.x,
          y: e.clientY - startPos.y
        }
        emit('update:position', props.node.id, newPosition)
      }

      const handleMouseUp = () => {
        isDragging.value = false
        document.removeEventListener('mousemove', handleMouseMove)
        document.removeEventListener('mouseup', handleMouseUp)
      }

      document.addEventListener('mousemove', handleMouseMove)
      document.addEventListener('mouseup', handleMouseUp)
    }

    // 选择节点
    const selectNode = (event) => {
      // 如果是在文本框中点击，不触发选择
      if (event.target !== textarea.value) {
        emit('select', props.node)
      }
    }

    // 开始连接
    const startConnection = (port) => {
      emit('connection:start', {
        nodeId: props.node.id,
        portPosition: port.position,
        portType: port.type
      })
    }

    // 结束连接
    const endConnection = (port) => {
      emit('connection:end', {
        nodeId: props.node.id,
        portPosition: port.position,
        portType: port.type
      })
    }

    // 开始调整大小
    const startResize = (event) => {
      event.stopPropagation()
      isResizing.value = true
      startPos = {
        x: event.clientX,
        y: event.clientY
      }
      startSize = {
        width: props.node.width || 200,
        height: props.node.height || 100
      }

      const handleMouseMove = (e) => {
        const deltaX = e.clientX - startPos.x
        const deltaY = e.clientY - startPos.y
        const newWidth = Math.max(150, startSize.width + deltaX)
        const newHeight = Math.max(80, startSize.height + deltaY)
        emit('update', {
          id: props.node.id,
          width: newWidth,
          height: newHeight
        })
      }

      const handleMouseUp = () => {
        isResizing.value = false
        document.removeEventListener('mousemove', handleMouseMove)
        document.removeEventListener('mouseup', handleMouseUp)
      }

      document.addEventListener('mousemove', handleMouseMove)
      document.addEventListener('mouseup', handleMouseUp)
    }

    // 处理AI按钮点击
    const handleAI = (event) => {
      event.stopPropagation()
      emit('ai-action', {
        id: props.node.id,
        content: content.value
      })
    }

    return {
      isDragging,
      isResizing,
      content,
      textarea,
      ports,
      startDrag,
      selectNode,
      updateContent,
      startConnection,
      endConnection,
      startResize,
      handleAI
    }
  }
}
</script>

<style scoped>
.flow-node {
  position: absolute;
  background: rgba(45, 45, 45, 0.95);
  border-radius: 4px;
  min-width: 150px;
  min-height: 80px;
  border: 1px solid rgba(255, 255, 255, 0.1);
  cursor: move;
  user-select: none;
  transition: all 300ms ease;
  backdrop-filter: blur(10px);
  font-size: 13px;
  line-height: 1.6;
  color: rgba(255, 255, 255, 0.9);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
}

.flow-node:hover {
  box-shadow: 0 0 0 1px rgba(82, 255, 168, 0.3);
}

.flow-node.selected {
  box-shadow: 0 0 0 1px rgba(82, 255, 168, 0.5);
}

.node-content {
    margin-top: 6px;
  height: 100%;
}

.node-textarea {
  width: 100%;
  height: 100%;
  background: transparent;
  border: none;
  color: rgba(255, 255, 255, 0.9);
  font-size: 13px;
  line-height: 1.6;
  resize: none;
  outline: none;
  font-family: inherit;
  cursor: text;
  padding: 0;
}

.node-textarea:focus {
  outline: none;
}

.node-port {
  position: absolute;
  width: 8px;
  height: 8px;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 50%;
  transition: all 0.2s ease;
  cursor: pointer;
  border: 1px solid rgba(255, 255, 255, 0.3);
}

.node-port:hover {
  background: rgba(82, 255, 168, 0.5);
  transform: scale(1.2);
}

.node-port.right {
  right: -4px;
  top: 50%;
  transform: translateY(-50%);
}

.node-port.left {
  left: -4px;
  top: 50%;
  transform: translateY(-50%);
}

.resize-handle {
  position: absolute;
  right: 0;
  bottom: 0;
  width: 12px;
  height: 12px;
  background: transparent;
  cursor: se-resize;
  transition: all 0.2s ease;
  opacity: 0;
  z-index: 10;
}

.resize-handle::after {
  content: '';
  position: absolute;
  right: 2px;
  bottom: 2px;
  width: 6px;
  height: 6px;
  border-right: 2px solid rgba(255, 255, 255, 0.3);
  border-bottom: 2px solid rgba(255, 255, 255, 0.3);
  transition: all 0.2s ease;
}

.flow-node:hover .resize-handle {
  opacity: 1;
}

.resize-handle:hover::after,
.flow-node.resizing .resize-handle::after {
  border-color: rgba(82, 255, 168, 0.8);
}

.flow-node.resizing {
  transition: none;
}

.ai-button {
  position: absolute;
  top: -20px;
  right: 8px;
  background: rgba(82, 255, 168, 0.15);
  border: 1px solid rgba(82, 255, 168, 0.3);
  border-radius: 4px;
  color: rgba(82, 255, 168, 0.9);
  font-size: 12px;
  padding: 2px 8px;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 4px;
  transition: all 0.2s ease;
  opacity: 0;
  z-index: 100;
}

.flow-node:hover .ai-button {
  opacity: 1;
  top: -24px;
}

.ai-button:hover {
  background: rgba(82, 255, 168, 0.25);
  border-color: rgba(82, 255, 168, 0.5);
}

.ai-icon {
  width: 14px;
  height: 14px;
}
</style> 