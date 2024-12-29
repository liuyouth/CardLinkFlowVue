<template>
  <div class="modal" :class="{ show: visible }" @click="closeModal">
    <div class="modal-content" @click.stop>
      <span class="close-button" @click="closeModal">&times;</span>
      <img :src="imageUrl" alt="Large preview">
    </div>
  </div>
</template>

<script>
export default {
  name: 'ImageModal',
  
  props: {
    visible: {
      type: Boolean,
      default: false
    },
    imageUrl: {
      type: String,
      default: ''
    }
  },

  setup(props, { emit }) {
    const closeModal = () => {
      emit('close')
    }

    return {
      closeModal
    }
  }
}
</script>

<style scoped>
.modal {
  display: none;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.9);
  z-index: 2000;
  backdrop-filter: blur(5px);
}

.modal.show {
  display: flex;
}

.modal-content {
  position: relative;
  margin: auto;
  padding: 20px;
  width: 90%;
  height: 90%;
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content img {
  max-width: 100%;
  max-height: 100%;
  object-fit: contain;
}

.close-button {
  position: absolute;
  top: 15px;
  right: 15px;
  color: #fff;
  font-size: 30px;
  font-weight: bold;
  cursor: pointer;
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(0, 0, 0, 0.5);
  border-radius: 50%;
}

.close-button:hover {
  background: rgba(255, 255, 255, 0.1);
}
</style> 