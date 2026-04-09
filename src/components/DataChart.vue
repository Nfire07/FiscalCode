<template>
  <div class="datachart" :style="containerStyle">
    <p v-if="title" class="datachart__title">{{ title }}</p>
    <p v-if="subtitle" class="datachart__subtitle">{{ subtitle }}</p>
    <div class="datachart__wrapper" :style="wrapperStyle">
      <canvas ref="canvas" />
    </div>
    <p v-if="error" class="datachart__error">{{ error }}</p>
  </div>
</template>

<script>
const CHART_JS_CDN = 'https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.min.js'

const SUPPORTED_TYPES = ['line', 'bar', 'pie', 'doughnut', 'radar', 'polarArea', 'bubble', 'scatter']

function loadScript(src) {
  return new Promise((resolve, reject) => {
    if (window.Chart) return resolve(window.Chart)
    const script = document.createElement('script')
    script.src = src
    script.onload = () => resolve(window.Chart)
    script.onerror = () => reject(new Error('Failed to load Chart.js'))
    document.head.appendChild(script)
  })
}

function getCssVar(name) {
  return getComputedStyle(document.documentElement).getPropertyValue(name).trim()
}

export default {
  name: 'DataChart',

  props: {
    type: {
      type: String,
      default: 'line',
      validator: (v) => SUPPORTED_TYPES.includes(v),
    },
    labels: {
      type: Array,
      default: () => [],
    },
    datasets: {
      type: Array,
      default: () => [],
    },
    title: {
      type: String,
      default: null,
    },
    subtitle: {
      type: String,
      default: null,
    },
    width: {
      type: String,
      default: '100%',
    },
    height: {
      type: String,
      default: '320px',
    },
    colors: {
      type: Array,
      default: () => [],
    },
    fill: {
      type: Boolean,
      default: false,
    },
    tension: {
      type: Number,
      default: 0.4,
    },
    borderWidth: {
      type: Number,
      default: 2,
    },
    pointRadius: {
      type: Number,
      default: 4,
    },
    showLegend: {
      type: Boolean,
      default: true,
    },
    legendPosition: {
      type: String,
      default: 'bottom',
    },
    showGrid: {
      type: Boolean,
      default: true,
    },
    showTooltip: {
      type: Boolean,
      default: true,
    },
    stacked: {
      type: Boolean,
      default: false,
    },
    horizontal: {
      type: Boolean,
      default: false,
    },
    beginAtZero: {
      type: Boolean,
      default: true,
    },
    xLabel: {
      type: String,
      default: null,
    },
    yLabel: {
      type: String,
      default: null,
    },
    options: {
      type: Object,
      default: () => ({}),
    },
    animate: {
      type: Boolean,
      default: true,
    },
    animationDuration: {
      type: Number,
      default: 800,
    },
    padding: {
      type: String,
      default: null,
    },
    background: {
      type: String,
      default: null,
    },
  },

  data() {
    return {
      chart: null,
      error: null,
    }
  },

  computed: {
    containerStyle() {
      const style = {}
      if (this.width) style.width = this.width
      if (this.padding) style.padding = this.padding
      if (this.background) style.background = this.background
      return style
    },

    wrapperStyle() {
      return { height: this.height, position: 'relative' }
    },

    resolvedColors() {
      if (this.colors.length) return this.colors
      const primary = getCssVar('--primary') || '#ED202F'
      const foreground = getCssVar('--foreground') || '#1a1a1a'
      const base = [
        primary,
        foreground,
        '#4e9af1',
        '#f4a261',
        '#2a9d8f',
        '#e9c46a',
        '#9b5de5',
        '#f15bb5',
      ]
      return base
    },

    isCircular() {
      return ['pie', 'doughnut', 'polarArea'].includes(this.type)
    },

    resolvedDatasets() {
      return this.datasets.map((ds, i) => {
        const color = this.resolvedColors[i % this.resolvedColors.length]
        const alpha = (hex) => {
          const r = parseInt(hex.slice(1, 3), 16)
          const g = parseInt(hex.slice(3, 5), 16)
          const b = parseInt(hex.slice(5, 7), 16)
          return `rgba(${r},${g},${b},0.15)`
        }
        const safeAlpha = color.startsWith('#') ? alpha(color) : color

        const base = {
          borderColor: this.isCircular ? getCssVar('--foreground') : color,
          backgroundColor: this.isCircular
            ? this.resolvedColors.map((c) => c)
            : this.fill
              ? safeAlpha
              : color,
          borderWidth: ds.borderWidth ?? this.borderWidth,
          ...ds,
        }

        if (this.type === 'line') {
          base.tension = ds.tension ?? this.tension
          base.fill = ds.fill ?? this.fill
          base.pointRadius = ds.pointRadius ?? this.pointRadius
          base.pointHoverRadius = (ds.pointRadius ?? this.pointRadius) + 3
          base.pointBackgroundColor = color
        }

        if (this.type === 'bar' && this.horizontal) {
          base.indexAxis = 'y'
        }

        return base
      })
    },

    chartData() {
      return {
        labels: this.labels,
        datasets: this.resolvedDatasets,
      }
    },

    chartOptions() {
      const foreground = getCssVar('--foreground') || '#1a1a1a'
      const gridColor = `${foreground}18`
      const fontFamily = getCssVar('--font-family') || "'DM Sans', sans-serif"

      const base = {
        responsive: true,
        maintainAspectRatio: false,
        animation: this.animate
          ? { duration: this.animationDuration, easing: 'easeOutQuart' }
          : false,
        plugins: {
          legend: {
            display: this.showLegend,
            position: this.legendPosition,
            labels: {
              color: foreground,
              font: { family: fontFamily, size: 12 },
              usePointStyle: true,
              pointStyleWidth: 10,
              padding: 16,
            },
          },
          tooltip: {
            enabled: this.showTooltip,
            backgroundColor: foreground,
            titleColor: getCssVar('--background') || '#dfdfdf',
            bodyColor: getCssVar('--background') || '#dfdfdf',
            borderColor: getCssVar('--primary') || '#ED202F',
            borderWidth: 1,
            padding: 10,
            titleFont: { family: fontFamily, weight: '600' },
            bodyFont: { family: fontFamily },
            cornerRadius: 6,
          },
        },
      }

      if (!this.isCircular) {
        const indexAxis = this.horizontal ? 'y' : 'x'
        base.indexAxis = indexAxis

        base.scales = {
          x: {
            stacked: this.stacked,
            display: true,
            grid: {
              display: this.showGrid,
              color: gridColor,
            },
            ticks: {
              color: foreground,
              font: { family: fontFamily, size: 11 },
            },
            title: this.xLabel
              ? { display: true, text: this.xLabel, color: foreground, font: { family: fontFamily } }
              : { display: false },
            border: { color: gridColor },
          },
          y: {
            stacked: this.stacked,
            display: true,
            beginAtZero: this.beginAtZero,
            grid: {
              display: this.showGrid,
              color: gridColor,
            },
            ticks: {
              color: foreground,
              font: { family: fontFamily, size: 11 },
            },
            title: this.yLabel
              ? { display: true, text: this.yLabel, color: foreground, font: { family: fontFamily } }
              : { display: false },
            border: { color: gridColor },
          },
        }
      }

      return this.deepMerge(base, this.options)
    },
  },

  watch: {
    chartData: {
      deep: true,
      handler() {
        this.updateChart()
      },
    },
    type() {
      this.destroyChart()
      this.initChart()
    },
    chartOptions: {
      deep: true,
      handler() {
        this.updateChart()
      },
    },
  },

  async mounted() {
    try {
      await loadScript(CHART_JS_CDN)
      this.initChart()
    } catch (e) {
      this.error = 'Failed to load chart library.'
    }
  },

  beforeUnmount() {
    this.destroyChart()
  },

  methods: {
    initChart() {
      if (!window.Chart || !this.$refs.canvas) return
      const ctx = this.$refs.canvas.getContext('2d')
      this.chart = new window.Chart(ctx, {
        type: this.type,
        data: this.chartData,
        options: this.chartOptions,
      })
    },

    updateChart() {
      if (!this.chart) return
      this.chart.data = this.chartData
      this.chart.options = this.chartOptions
      this.chart.update()
    },

    destroyChart() {
      if (this.chart) {
        this.chart.destroy()
        this.chart = null
      }
    },

    deepMerge(target, source) {
      const output = { ...target }
      for (const key in source) {
        if (
          source[key] &&
          typeof source[key] === 'object' &&
          !Array.isArray(source[key])
        ) {
          output[key] = this.deepMerge(target[key] || {}, source[key])
        } else {
          output[key] = source[key]
        }
      }
      return output
    },
  },
}
</script>

<style scoped>
.datachart {
  box-sizing: border-box;
  font-family: var(--font-family, 'DM Sans', sans-serif);
  color: var(--foreground, #1a1a1a);
  width: 100%;
}

.datachart__title {
  margin: 0 0 0.25rem;
  font-size: 1rem;
  font-weight: 600;
  color: var(--foreground);
  letter-spacing: -0.01em;
}

.datachart__subtitle {
  margin: 0 0 1rem;
  font-size: 0.8rem;
  color: var(--foreground);
  opacity: 0.55;
}

.datachart__wrapper {
  width: 100%;
}

.datachart__error {
  margin: 0.5rem 0 0;
  font-size: 0.8rem;
  color: var(--error);
}
</style>