<template>
  <div class="data-loader">

    <div v-if="state === 'loading'" class="data-loader__loading">
      <template v-if="skeleton">
        <div
          v-for="i in skeletonRows"
          :key="i"
          class="skeleton-row"
          :style="{ animationDelay: `${(i - 1) * 80}ms` }"
        >
          <div class="skeleton-block skeleton-block--avatar"></div>
          <div class="skeleton-lines">
            <div class="skeleton-block skeleton-block--line" style="width: 65%"></div>
            <div class="skeleton-block skeleton-block--line" style="width: 40%"></div>
          </div>
        </div>
      </template>
      <template v-else>
        <div class="spinner-wrapper">
          <div class="spinner"></div>
          <p v-if="loadingText" class="state-text">{{ loadingText }}</p>
        </div>
      </template>
    </div>

    <div v-else-if="state === 'error'" class="data-loader__error">
      <span class="state-icon material-icons-round">cloud_off</span>
      <p class="state-title">{{ errorTitle }}</p>
      <p class="state-message">{{ errorMessage || fetchError }}</p>
      <button
        v-if="retryable"
        type="button"
        class="retry-btn"
        @click="load"
      >
        <span class="material-icons-round">refresh</span>
        {{ retryText }}
      </button>
    </div>

    <div v-else-if="state === 'empty'" class="data-loader__empty">
      <span class="state-icon material-icons-round">{{ emptyIcon }}</span>
      <p class="state-title">{{ emptyTitle }}</p>
      <p class="state-message">{{ emptyMessage }}</p>
    </div>

    <div v-else-if="state === 'success'" class="data-loader__content">
      <slot :data="data" />
    </div>

  </div>
</template>

<script>
export default {
  name: 'DataLoader',

  props: {
    url: {
      type: String,
      required: true,
    },
    fetchOptions: {
      type: Object,
      default: () => ({}),
    },
    transformer: {
      type: Function,
      default: null,
    },
    isEmpty: {
      type: Function,
      default: null,
    },
    skeleton: {
      type: Boolean,
      default: false,
    },
    skeletonRows: {
      type: Number,
      default: 4,
    },
    loadingText: {
      type: String,
      default: 'Loading...',
    },
    retryable: {
      type: Boolean,
      default: true,
    },
    retryText: {
      type: String,
      default: 'Try again',
    },
    errorTitle: {
      type: String,
      default: 'Something went wrong',
    },
    errorMessage: {
      type: String,
      default: '',
    },
    emptyIcon: {
      type: String,
      default: 'inbox',
    },
    emptyTitle: {
      type: String,
      default: 'No data found',
    },
    emptyMessage: {
      type: String,
      default: 'There is nothing to display here yet.',
    },
    immediate: {
      type: Boolean,
      default: true,
    },
  },

  emits: ['success', 'error', 'empty'],

  data() {
    return {
      state: 'loading',
      data: null,
      fetchError: '',
    };
  },

  mounted() {
    if (this.immediate) this.load();
  },

  methods: {
    async load() {
      this.state = 'loading';
      this.fetchError = '';

      try {
        const response = await fetch(this.url, this.fetchOptions);

        if (!response.ok) {
          throw new Error(`Request failed with status ${response.status}`);
        }

        let result = await response.json();

        if (this.transformer) {
          result = this.transformer(result);
        }

        const empty = this.isEmpty
          ? this.isEmpty(result)
          : Array.isArray(result)
            ? result.length === 0
            : result === null || result === undefined;

        if (empty) {
          this.data = result;
          this.state = 'empty';
          this.$emit('empty', result);
        } else {
          this.data = result;
          this.state = 'success';
          this.$emit('success', result);
        }
      } catch (err) {
        this.fetchError = err.message || 'Unknown error';
        this.state = 'error';
        this.$emit('error', this.fetchError);
      }
    },
  },
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/icon?family=Material+Icons+Round');
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:wght@400;500;600&display=swap');

.data-loader {
  font-family: var(--font-family);
  width: 100%;
}

.data-loader__loading,
.data-loader__error,
.data-loader__empty {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 3rem 1.5rem;
  gap: 0.6rem;
  text-align: center;
}

.skeleton-row {
  display: flex;
  align-items: center;
  gap: 0.85rem;
  padding: 0.65rem 0;
  width: 100%;
  animation: skeleton-fade-in 0.3s ease both;
}

@keyframes skeleton-fade-in {
  from { opacity: 0; transform: translateY(6px); }
  to   { opacity: 1; transform: translateY(0); }
}

.skeleton-lines {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 0.45rem;
}

.skeleton-block {
  background: linear-gradient(
    90deg,
    color-mix(in srgb, var(--foreground) 8%, transparent) 25%,
    color-mix(in srgb, var(--foreground) 14%, transparent) 50%,
    color-mix(in srgb, var(--foreground) 8%, transparent) 75%
  );
  background-size: 200% 100%;
  animation: skeleton-shimmer 1.4s infinite ease-in-out;
  border-radius: 6px;
}

.skeleton-block--avatar {
  width: 40px;
  height: 40px;
  border-radius: 10px;
  flex-shrink: 0;
}

.skeleton-block--line {
  height: 12px;
  border-radius: 4px;
}

@keyframes skeleton-shimmer {
  0%   { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}

.spinner-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
}

.spinner {
  width: 36px;
  height: 36px;
  border: 3px solid color-mix(in srgb, var(--primary) 20%, transparent);
  border-top-color: var(--primary);
  border-radius: 50%;
  animation: spin 0.75s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}

.state-icon {
  font-size: 2.5rem;
  user-select: none;
}

.data-loader__error .state-icon {
  color: var(--error);
}

.data-loader__empty .state-icon {
  color: color-mix(in srgb, var(--foreground) 30%, transparent);
}

.state-title {
  margin: 0;
  font-size: 0.95rem;
  font-weight: 600;
  color: var(--foreground);
}

.state-message {
  margin: 0;
  font-size: 0.825rem;
  color: color-mix(in srgb, var(--foreground) 55%, transparent);
  max-width: 280px;
  line-height: 1.5;
}

.state-text {
  margin: 0;
  font-size: 0.825rem;
  color: color-mix(in srgb, var(--foreground) 55%, transparent);
}

.retry-btn {
  display: inline-flex;
  align-items: center;
  gap: 0.4rem;
  margin-top: 0.4rem;
  padding: 0.55rem 1.1rem;
  background: color-mix(in srgb, var(--primary) 10%, transparent);
  border: 1px solid color-mix(in srgb, var(--primary) 35%, transparent);
  border-radius: 8px;
  color: var(--primary);
  font-family: var(--font-family, 'DM Sans', sans-serif);
  font-size: 0.85rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.18s, border-color 0.18s, transform 0.15s;
}

.retry-btn:hover {
  background: color-mix(in srgb, var(--primary) 18%, transparent);
  border-color: var(--primary);
}

.retry-btn:active {
  transform: scale(0.97);
}

.retry-btn .material-icons-round {
  font-size: 1rem;
  user-select: none;
}

.data-loader__content {
  width: 100%;
}
</style>