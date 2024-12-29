<template>
  <div class="main-content" id="flow-canvas" @mousemove="handleMouseMove">
    <!-- 鼠标位置显示面板 -->
    <div class="mouse-position-panel" v-if="mousePosition.x || mousePosition.y">
      <div>X: {{ Math.round(mousePosition.x) }}</div>
      <div>Y: {{ Math.round(mousePosition.y) }}</div>
    </div>

    <!-- 输入框容器 -->
    <div class="input-container">
      <input
        type="text"
        class="resource-input"
        placeholder="输入文本并按回车"
        v-model="nodeContent"
        @keyup.enter="addNode"
      />
      <button class="add-node-button" title="添加空白节点" @click="addEmptyNode">
        +
      </button>
    </div>

    <!-- 节点列表 -->
    <flow-node
      v-for="node in nodes"
      :key="node.id"
      :node="node"
      :is-selected="selectedElement?.id === node.id"
      @update:position="updateNodePosition"
      @select="selectNode"
      @update="handleNodeUpdate"
      @connection:start="startConnection"
      @connection:end="endConnection"
      @ai-action="handleAIAction"
    />

    <!-- 连接线 -->
    <flow-edge
      v-for="edge in edges"
      :key="edge.id"
      :edge="edge"
      :is-selected="selectedElement?.id === edge.id"
      @select="selectEdge"
    />

    <!-- 预览连线 -->
    <flow-edge
      v-if="previewEdge"
      :edge="previewEdge"
      :is-preview="true"
    />
  </div>
</template>

<script>
import FlowNode from './FlowNode.vue'
import FlowEdge from './FlowEdge.vue'
import { ref, reactive, onMounted } from 'vue'

// 初始节点数据
const initialNodes = []

// 初始连接线数据
const initialEdges = []

export default {
  name: 'FlowCanvas',
  
  components: {
    FlowNode,
    FlowEdge
  },

  setup() {
    const nodes = reactive([])
    const edges = reactive([])
    const nodeContent = ref('')
    const selectedElement = ref(null)
    let connectionStart = null
    const previewEdge = ref(null)
    const mousePosition = reactive({ x: 0, y: 0 })

    // 初始化数据
    onMounted(() => {
      initialNodes.forEach(node => nodes.push(node))
      initialEdges.forEach(edge => edges.push(edge))
    })

    // 添加节点
    const addNode = () => {
      if (!nodeContent.value) return
      
      const node = {
        id: `node-${Date.now()}`,
        content: nodeContent.value,
        position: { x: 100, y: 100 },
        width: 200,
        height: 100
      }
      
      nodes.push(node)
      nodeContent.value = ''
    }

    // 添加空白节点
    const addEmptyNode = () => {
      const node = {
        id: `node-${Date.now()}`,
        content: '双击编辑文本',
        position: { x: 100, y: 100 },
        width: 200,
        height: 100
      }
      nodes.push(node)
    }

    // 更新节点位置
    const updateNodePosition = (nodeId, position) => {
      const node = nodes.find(n => n.id === nodeId)
      if (node) {
        node.position = position
        // 更新相关连接线
        edges.forEach(edge => {
          const portOffset = 6 // 连接点的半径
          if (edge.source.id === nodeId) {
            edge.source.x = position.x + (edge.source.position === 'right' ? node.width + portOffset : -portOffset)
            edge.source.y = position.y + node.height / 2
            console.log('更新连线起点:', {
              nodeId,
              position: edge.source.position,
              x: edge.source.x,
              y: edge.source.y
            })
          }
          if (edge.target.id === nodeId) {
            edge.target.x = position.x + (edge.target.position === 'right' ? node.width + portOffset : -portOffset)
            edge.target.y = position.y + node.height / 2
            console.log('更新连线终点:', {
              nodeId,
              position: edge.target.position,
              x: edge.target.x,
              y: edge.target.y
            })
          }
        })
      }
    }

    // 选择节点
    const selectNode = (node) => {
      selectedElement.value = { type: 'node', ...node }
    }

    // 选择连接线
    const selectEdge = (edge) => {
      selectedElement.value = { type: 'edge', ...edge }
    }

    // 开始连接
    const startConnection = ({ nodeId, portPosition }) => {
      const sourceNode = nodes.find(n => n.id === nodeId)
      if (sourceNode) {
        const portOffset = 6 // 连接点的半径
        const x = sourceNode.position.x + (portPosition === 'right' ? sourceNode.width + portOffset : -portOffset)
        const y = sourceNode.position.y + sourceNode.height / 2

        console.log('连接起点坐标:', {
          nodeId,
          portPosition,
          x,
          y,
          nodePosition: sourceNode.position,
          nodeWidth: sourceNode.width,
          nodeHeight: sourceNode.height
        })

        connectionStart = {
          id: nodeId,
          position: portPosition,
          x,
          y
        }
      }
    }

    // 结束连接
    const endConnection = ({ nodeId, portPosition }) => {
      if (!connectionStart || connectionStart.id === nodeId) {
        connectionStart = null
        previewEdge.value = null
        return
      }

      const targetNode = nodes.find(n => n.id === nodeId)
      if (targetNode) {
        const portOffset = 6 // 连接点的半径
        const x = targetNode.position.x + (portPosition === 'right' ? targetNode.width + portOffset : -portOffset)
        const y = targetNode.position.y + targetNode.height / 2

        console.log('连接终点坐标:', {
          nodeId,
          portPosition,
          x,
          y,
          nodePosition: targetNode.position,
          nodeWidth: targetNode.width,
          nodeHeight: targetNode.height
        })

        // 检查是否已存在相同的连接
        const existingEdge = edges.find(edge => 
          edge.source.id === connectionStart.id && 
          edge.target.id === nodeId &&
          edge.source.position === connectionStart.position &&
          edge.target.position === portPosition
        )

        if (existingEdge) {
          connectionStart = null
          previewEdge.value = null
          return
        }

        const edge = {
          id: `edge-${Date.now()}`,
          source: connectionStart,
          target: {
            id: nodeId,
            position: portPosition,
            x,
            y
          },
          type: 'default'
        }
        
        console.log('创建新连线:', edge)
        edges.push(edge)
      }
      connectionStart = null
      previewEdge.value = null
    }

    // 更新节点内容和大小
    const handleNodeUpdate = (update) => {
      const node = nodes.find(n => n.id === update.id)
      if (node) {
        if (update.content !== undefined) {
          node.content = update.content
        }
        if (update.width !== undefined) {
          node.width = update.width
        }
        if (update.height !== undefined) {
          node.height = update.height
        }
        // 更新相关连接线
        edges.forEach(edge => {
          if (edge.source.id === update.id) {
            edge.source.x = node.position.x + node.width
            edge.source.y = node.position.y + node.height / 2
          }
          if (edge.target.id === update.id) {
            edge.target.x = node.position.x
            edge.target.y = node.position.y + node.height / 2
          }
        })
      }
    }

    // 获取节点的所有输入节点
    const getInputNodes = (nodeId, result = []) => {
      // 找到所有以该节点为目标的边
      const inputEdges = edges.filter(edge => edge.target.id === nodeId)
      
      // 对于每个输入边
      inputEdges.forEach(edge => {
        const sourceNode = nodes.find(n => n.id === edge.source.id)
        if (sourceNode && !result.some(n => n.id === sourceNode.id)) {
          result.push(sourceNode)
          // 递归获取输入节点的输入节点
          getInputNodes(sourceNode.id, result)
        }
      })
      
      return result
    }

    // 处理AI按钮点击
    const handleAIAction = async (nodeData) => {
      const currentNode = nodes.find(n => n.id === nodeData.id)
      if (currentNode) {
        // 获取所有输入节点
        const inputNodes = getInputNodes(nodeData.id)
        
        // 构建完整的数据链
        const dataChain = {
          currentNode: {
            id: currentNode.id,
            content: currentNode.content
          },
          inputNodes: inputNodes.map(node => ({
            id: node.id,
            content: node.content
          }))
        }

        // 在控制台打印数据链
        console.log('节点数据链:')
        console.log('当前节点:', dataChain.currentNode)
        if (dataChain.inputNodes.length > 0) {
          console.log('输入节点链:')
          dataChain.inputNodes.forEach((node, index) => {
            console.log(`${index + 1}. 节点 ${node.id}:`, node.content)
          })
        } else {
          console.log('没有输入节点')
        }
      }
    }

    // 处理鼠标移动
    const handleMouseMove = (event) => {
      const canvas = document.getElementById('flow-canvas')
      if (!canvas) return

      const rect = canvas.getBoundingClientRect()
      const x = event.clientX - rect.left
      const y = event.clientY - rect.top

      // 更新鼠标位置
      mousePosition.x = x
      mousePosition.y = y
      
      // 如果正在创建连接，更新预览线
      if (connectionStart) {
        previewEdge.value = {
          id: 'preview',
          source: connectionStart,
          target: {
            x: mousePosition.x,
            y: mousePosition.y,
            position: connectionStart.position === 'right' ? 'left' : 'right'
          },
          type: 'preview'
        }
      }
    }

    return {
      nodes,
      edges,
      nodeContent,
      selectedElement,
      addNode,
      addEmptyNode,
      updateNodePosition,
      selectNode,
      selectEdge,
      startConnection,
      endConnection,
      handleNodeUpdate,
      handleAIAction,
      previewEdge,
      handleMouseMove,
      mousePosition,
    }
  }
}
</script>

<style scoped>
.main-content {
  position: relative;
  width: 100vw;
  height: 100vh;
  background-color: var(--background-dark);
}

.input-container {
  position: absolute;
  left: 20px;
  top: 20px;
  width: 240px;
  display: flex;
  gap: 8px;
  z-index: 1000;
}

.resource-input {
  flex: 1;
  width: auto;
  padding: 8px 12px;
  background: rgba(45, 45, 45, 0.95);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 4px;
  color: #fff;
  font-size: 13px;
  outline: none;
  transition: border-color 0.2s ease;
}

.resource-input:focus {
  border-color: rgba(82, 255, 168, 0.5);
}

.add-node-button {
  width: 32px;
  height: 32px;
  background: rgba(45, 45, 45, 0.95);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 4px;
  color: rgba(255, 255, 255, 0.7);
  font-size: 18px;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s ease;
  padding: 0;
  outline: none;
}

.add-node-button:hover {
  border-color: rgba(82, 255, 168, 0.5);
  color: rgba(82, 255, 168, 0.8);
}

.mouse-position-panel {
  position: fixed;
  right: 20px;
  top: 20px;
  background: rgba(45, 45, 45, 0.95);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 4px;
  padding: 8px 12px;
  color: rgba(255, 255, 255, 0.8);
  font-size: 12px;
  font-family: monospace;
  z-index: 1000;
  pointer-events: none;
  display: flex;
  flex-direction: column;
  gap: 4px;
}
</style> 