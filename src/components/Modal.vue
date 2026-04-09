<template>
  <Teleport to="body">
    <Transition name="modal">
      <div v-if="modelValue" class="modal-overlay" @click.self="close">
        <div class="modal-container" :style="containerStyle" role="dialog" aria-modal="true">
          <button class="modal-close" @click="close" aria-label="Close">
            <span class="material-icons">close</span>
          </button>
          <div class="modal-content">
            <slot />
          </div>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<script>
export default {
  name: 'Modal',

  props: {
    modelValue: {
      type: Boolean,
      default: false,
    },
    width: {
      type: String,
      default: '560px',
    },
    maxHeight: {
      type: String,
      default: '90vh',
    },
    closeOnOverlay: {
      type: Boolean,
      default: true,
    },
    closeOnEsc: {
      type: Boolean,
      default: true,
    },
    padding: {
      type: String,
      default: '2rem',
    },
  },

  emits: ['update:modelValue', 'close'],

  computed: {
    containerStyle() {
      return {
        width: this.width,
        maxHeight: this.maxHeight,
        padding: this.padding,
      }
    },
  },

  watch: {
    modelValue(val) {
      if (val) {
        document.body.style.overflow = 'hidden'
        if (this.closeOnEsc) {
          document.addEventListener('keydown', this._onKeydown)
        }
      } else {
        document.body.style.overflow = ''
        document.removeEventListener('keydown', this._onKeydown)
      }
    },
  },

  beforeUnmount() {
    document.body.style.overflow = ''
    document.removeEventListener('keydown', this._onKeydown)
  },

  methods: {
    close() {
      if (!this.closeOnOverlay) return
      this.$emit('update:modelValue', false)
      this.$emit('close')
    },

    forceClose() {
      this.$emit('update:modelValue', false)
      this.$emit('close')
    },

    _onKeydown(e) {
      if (e.key === 'Escape') this.forceClose()
    },
  },
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/icon?family=Material+Icons');

.modal-overlay {
  position: fixed;
  inset: 0;
  z-index: 1000;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(0, 0, 0, 0.45);
  backdrop-filter: blur(2px);
  -webkit-backdrop-filter: blur(2px);
}

.modal-container {
  position: relative;
  background: var(--background, #dfdfdf);
  color: var(--foreground, #1a1a1a);
  font-family: var(--font-family, 'DM Sans', sans-serif);
  border-radius: 12px;
  box-sizing: border-box;
  max-width: calc(100vw - 2rem);
  overflow-y: auto;
  box-shadow:
    0 4px 6px rgba(0, 0, 0, 0.04),
    0 16px 48px rgba(0, 0, 0, 0.12);
}

.modal-close {
  position: absolute;
  top: 1rem;
  right: 1rem;
  width: 32px;
  height: 32px;
  border: none;
  border-radius: 6px;
  background: transparent;
  color: var(--foreground, #1a1a1a);
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: background 0.15s ease, opacity 0.15s ease;
  opacity: 0.5;
  padding: 0;
  flex-shrink: 0;
}

.modal-close:hover {
  background: rgba(0, 0, 0, 0.08);
  opacity: 1;
}

.modal-close .material-icons {
  font-size: 20px;
  line-height: 1;
}

.modal-content {
  width: 100%;
}

.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.2s ease;
}

.modal-enter-active .modal-container,
.modal-leave-active .modal-container {
  transition: transform 0.2s ease, opacity 0.2s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

.modal-enter-from .modal-container,
.modal-leave-to .modal-container {
  transform: translateY(12px) scale(0.98);
  opacity: 0;
}
</style>