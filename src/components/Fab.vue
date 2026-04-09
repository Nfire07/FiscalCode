<template>
  <div class="fab-container" :class="positionClass">
    <transition name="fab-actions">
      <div v-if="isOpen" class="fab-actions">
        <button
          v-for="action in actions"
          :key="action.icon"
          class="fab-action"
          :title="action.label"
          @click="handleAction(action)"
        >
          <span class="material-icons">{{ action.icon }}</span>
          <span v-if="showLabels" class="fab-action-label">{{ action.label }}</span>
        </button>
      </div>
    </transition>

    <button
      class="fab-main"
      :class="{ 'fab-main--open': isOpen }"
      :title="tooltip"
      @click="toggle"
    >
      <span class="material-icons fab-icon fab-icon--default">{{ icon }}</span>
      <span class="material-icons fab-icon fab-icon--close">close</span>
    </button>
  </div>
</template>

<script>
export default {
  name: 'Fab',

  props: {
    icon: {
      type: String,
      default: 'add'
    },
    tooltip: {
      type: String,
      default: 'Open menu'
    },
    position: {
      type: String,
      default: 'bottom-right',
      validator: (value) =>
        ['top-left', 'top-right', 'bottom-left', 'bottom-right'].includes(value)
    },
    actions: {
      type: Array,
      default: () => []
    },
    showLabels: {
      type: Boolean,
      default: true
    },
    offset: {
      type: Number,
      default: 24
    }
  },

  emits: ['action', 'open', 'close'],

  data() {
    return {
      isOpen: false
    }
  },

  computed: {
    positionClass() {
      return `fab-container--${this.position}`
    }
  },

  methods: {
    toggle() {
      this.isOpen = !this.isOpen
      this.$emit(this.isOpen ? 'open' : 'close')
    },
    handleAction(action) {
      this.$emit('action', action)
      if (action.handler) action.handler()
      this.isOpen = false
      this.$emit('close')
    }
  }
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/icon?family=Material+Icons');
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:wght@400;500;700&display=swap');

.fab-container {
  position: fixed;
  z-index: 1000;
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  gap: 12px;
  font-family: var(--font-family);
}

.fab-container--bottom-right {
  bottom: v-bind('offset + "px"');
  right: v-bind('offset + "px"');
  flex-direction: column-reverse;
  align-items: flex-end;
}

.fab-container--bottom-left {
  bottom: v-bind('offset + "px"');
  left: v-bind('offset + "px"');
  flex-direction: column-reverse;
  align-items: flex-start;
}

.fab-container--top-right {
  top: v-bind('offset + "px"');
  right: v-bind('offset + "px"');
  flex-direction: column;
  align-items: flex-end;
}

.fab-container--top-left {
  top: v-bind('offset + "px"');
  left: v-bind('offset + "px"');
  flex-direction: column;
  align-items: flex-start;
}

.fab-main {
  width: 56px;
  height: 56px;
  border-radius: 16px;
  border: none;
  background: var(--primary);
  color: var(--foreground);
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  transition:
    transform 0.2s cubic-bezier(0.34, 1.56, 0.64, 1),
    box-shadow 0.2s ease,
    border-radius 0.2s ease;
  position: relative;
  overflow: hidden;
  flex-shrink: 0;
}

.fab-main::before {
  content: '';
  position: absolute;
  inset: 0;
  background: rgba(255, 255, 255, 0);
  transition: background 0.15s ease;
}

.fab-main:hover::before {
  background: rgba(255, 255, 255, 0.12);
}

.fab-main:hover {
  box-shadow:
    0 4px 8px rgba(0, 0, 0, 0.18);
}


.fab-main--open {
  border-radius: 12px;
}

.fab-icon {
  position: absolute;
  font-size: 24px;
  transition: transform 0.3s cubic-bezier(0.34, 1.56, 0.64, 1), opacity 0.2s ease;
}

.fab-icon--default {
  transform: rotate(0deg);
  opacity: 1;
}

.fab-icon--close {
  transform: rotate(-90deg);
  opacity: 0;
}

.fab-main--open .fab-icon--default {
  transform: rotate(90deg);
  opacity: 0;
}

.fab-main--open .fab-icon--close {
  transform: rotate(0deg);
  opacity: 1;
}

.fab-actions {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.fab-container--bottom-right .fab-actions,
.fab-container--top-right .fab-actions {
  align-items: flex-end;
}

.fab-container--bottom-left .fab-actions,
.fab-container--top-left .fab-actions {
  align-items: flex-start;
}

.fab-action {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 0;
  border: none;
  background: none;
  cursor: pointer;
}

.fab-container--bottom-right .fab-action,
.fab-container--top-right .fab-action {
  flex-direction: row-reverse;
}

.fab-action .material-icons {
  width: 44px;
  height: 44px;
  border-radius: 12px;
  background: var(--primary);
  color: var(--foreground);
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
  transition:
    transform 0.18s cubic-bezier(0.34, 1.56, 0.64, 1),
    background 0.15s ease,
    box-shadow 0.15s ease;
  flex-shrink: 0;
}

.fab-action:hover .material-icons {
  background: var(--primary);
}

.fab-action-label {
  font-family: var(--font-family);
  font-size: 13px;
  font-weight: 500;
  color: var(--foreground);
  background: var(--background);
  padding: 4px 10px;
  border-radius: 6px;
  box-shadow: 0 1px 4px rgba(0, 0, 0, 0.12);
  white-space: nowrap;
  letter-spacing: 0.01em;
}

.fab-actions-enter-active {
  transition: none;
}

.fab-actions-enter-active .fab-action {
  animation: fab-pop 0.28s cubic-bezier(0.34, 1.56, 0.64, 1) both;
}

.fab-actions-enter-active .fab-action:nth-child(1) { animation-delay: 0ms; }
.fab-actions-enter-active .fab-action:nth-child(2) { animation-delay: 40ms; }
.fab-actions-enter-active .fab-action:nth-child(3) { animation-delay: 80ms; }
.fab-actions-enter-active .fab-action:nth-child(4) { animation-delay: 120ms; }
.fab-actions-enter-active .fab-action:nth-child(5) { animation-delay: 160ms; }

.fab-actions-leave-active {
  transition: opacity 0.15s ease, transform 0.15s ease;
}

.fab-actions-leave-to {
  opacity: 0;
  transform: scale(0.92) translateY(8px);
}

@keyframes fab-pop {
  from {
    opacity: 0;
    transform: scale(0.7) translateY(12px);
  }
  to {
    opacity: 1;
    transform: scale(1) translateY(0);
  }
}
</style>