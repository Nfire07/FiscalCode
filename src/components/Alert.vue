<template>
  <Transition name="alert-slide">
    <div
      v-if="visible"
      class="alert"
      :class="`alert--${type}`"
      role="alert"
      :aria-live="type === 'error' ? 'assertive' : 'polite'"
    >
      <div class="alert__progress" v-if="timeout > 0">
        <div class="alert__progress-bar" :style="{ animationDuration: `${timeout}ms` }"></div>
      </div>

      <span class="alert__icon material-icons-round">{{ icons[type] }}</span>

      <div class="alert__body">
        <p v-if="title" class="alert__title">{{ title }}</p>
        <p class="alert__message">{{ message }}</p>
      </div>

      <button
        v-if="dismissible"
        class="alert__close"
        type="button"
        aria-label="Dismiss alert"
        @click="dismiss"
      >
        <span class="material-icons-round">close</span>
      </button>
    </div>
  </Transition>
</template>

<script>
export default {
  name: 'Alert',

  props: {
    type: {
      type: String,
      default: 'info',
      validator: (v) => ['success', 'error', 'warning', 'info'].includes(v),
    },
    title: {
      type: String,
      default: '',
    },
    message: {
      type: String,
      required: true,
    },
    dismissible: {
      type: Boolean,
      default: true,
    },
    timeout: {
      type: Number,
      default: 0,
    },
    modelValue: {
      type: Boolean,
      default: true,
    },
  },

  emits: ['update:modelValue', 'dismiss'],

  data() {
    return {
      visible: this.modelValue,
      timer: null,
      icons: {
        success: 'check_circle',
        error: 'error',
        warning: 'warning',
        info: 'info',
      },
    };
  },

  watch: {
    modelValue(val) {
      this.visible = val;
      if (val && this.timeout > 0) this.startTimer();
    },
  },

  mounted() {
    if (this.visible && this.timeout > 0) this.startTimer();
  },

  beforeUnmount() {
    clearTimeout(this.timer);
  },

  methods: {
    startTimer() {
      clearTimeout(this.timer);
      this.timer = setTimeout(() => this.dismiss(), this.timeout);
    },

    dismiss() {
      this.visible = false;
      this.$emit('update:modelValue', false);
      this.$emit('dismiss');
    },
  },
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/icon?family=Material+Icons+Round');
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:wght@400;500;600&display=swap');

.alert {
  position: relative;
  display: flex;
  align-items: flex-start;
  gap: 0.75rem;
  padding: 0.9rem 1rem 0.9rem 1rem;
  border-radius: 12px;
  border: 1px solid transparent;
  font-family: var(--font-family, 'DM Sans', sans-serif);
  overflow: hidden;
  width: 100%;
}

.alert--info {
  background: color-mix(in srgb, #3b82f6 10%, transparent);
  border-color: color-mix(in srgb, #3b82f6 30%, transparent);
  --alert-accent: #3b82f6;
}

.alert--success {
  background: color-mix(in srgb, #22c55e 10%, transparent);
  border-color: color-mix(in srgb, #22c55e 30%, transparent);
  --alert-accent: #22c55e;
}

.alert--warning {
  background: color-mix(in srgb, #f59e0b 10%, transparent);
  border-color: color-mix(in srgb, #f59e0b 30%, transparent);
  --alert-accent: #f59e0b;
}

.alert--error {
  background: color-mix(in srgb, var(--error) 12%, transparent);
  border-color: color-mix(in srgb, var(--error) 35%, transparent);
  --alert-accent: var(--error);
}

.alert__progress {
  position: absolute;
  bottom: 0;
  left: 0;
  right: 0;
  height: 2px;
  background: color-mix(in srgb, var(--alert-accent) 20%, transparent);
}

.alert__progress-bar {
  height: 100%;
  background: var(--alert-accent);
  width: 100%;
  transform-origin: left;
  animation: alert-progress linear forwards;
}

@keyframes alert-progress {
  from { transform: scaleX(1); }
  to   { transform: scaleX(0); }
}

.alert__icon {
  font-size: 1.25rem;
  color: var(--alert-accent);
  flex-shrink: 0;
  margin-top: 1px;
  user-select: none;
}

.alert__body {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 0.15rem;
}

.alert__title {
  margin: 0;
  font-size: 0.875rem;
  font-weight: 600;
  color: var(--foreground);
  line-height: 1.3;
}

.alert__message {
  margin: 0;
  font-size: 0.825rem;
  font-weight: 400;
  color: color-mix(in srgb, var(--foreground) 80%, transparent);
  line-height: 1.5;
}

.alert__close {
  background: none;
  border: none;
  cursor: pointer;
  padding: 0.1rem;
  border-radius: 6px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: color-mix(in srgb, var(--foreground) 50%, transparent);
  transition: background 0.15s, color 0.15s;
  flex-shrink: 0;
  margin-top: 1px;
}

.alert__close:hover {
  background: color-mix(in srgb, var(--alert-accent) 15%, transparent);
  color: var(--alert-accent);
}

.alert__close .material-icons-round {
  font-size: 1rem;
  user-select: none;
}

.alert-slide-enter-active {
  transition: all 0.25s cubic-bezier(0.34, 1.56, 0.64, 1);
}

.alert-slide-leave-active {
  transition: all 0.18s ease-in;
}

.alert-slide-enter-from {
  opacity: 0;
  transform: translateY(-8px) scale(0.97);
}

.alert-slide-leave-to {
  opacity: 0;
  transform: translateY(-4px) scale(0.98);
}
</style>