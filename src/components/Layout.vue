<template>
  <component
    :is="tag"
    class="layout"
    :style="computedStyle"
    :class="`layout--${resolvedMode}`"
  >
    <slot />
  </component>
</template>

<script>
const FR_PATTERN = /^(\d+(\.\d+)?fr\s*)+$/
const COLUMNS_PATTERN = /^\d+$/
const FR_WITH_OTHERS = /^[\d.]+fr(\s+[\d.]+fr|\s+\d+(px|rem|em|%))*(\s+[\d.]+fr|\s+\d+(px|rem|em|%))*$/

export default {
  name: 'Layout',

  props: {
    tag: {
      type: String,
      default: 'div',
    },
    type: {
      type: String,
      default: 'flex',
    },
    columns: {
      type: [String, Number],
      default: null,
    },
    rows: {
      type: [String, Number],
      default: null,
    },
    gap: {
      type: [String, Number],
      default: '1rem',
    },
    rowGap: {
      type: [String, Number],
      default: null,
    },
    columnGap: {
      type: [String, Number],
      default: null,
    },
    align: {
      type: String,
      default: null,
    },
    justify: {
      type: String,
      default: null,
    },
    direction: {
      type: String,
      default: null,
    },
    wrap: {
      type: Boolean,
      default: true,
    },
    inline: {
      type: Boolean,
      default: false,
    },
    minWidth: {
      type: String,
      default: null,
    },
    autoFill: {
      type: Boolean,
      default: false,
    },
    autoFit: {
      type: Boolean,
      default: false,
    },
    areas: {
      type: [String, Array],
      default: null,
    },
    padding: {
      type: String,
      default: null,
    },
    width: {
      type: String,
      default: null,
    },
    height: {
      type: String,
      default: null,
    },
    maxWidth: {
      type: String,
      default: null,
    },
    center: {
      type: Boolean,
      default: false,
    },
    margin:{
        type: String,
        defautl: "0px",
    },
  },

  computed: {
    resolved() {
      return this.parseTemplateType(this.type)
    },

    resolvedMode() {
      return this.resolved.mode
    },

    computedStyle() {
      const style = {}
      const { mode, columns: tplColumns, direction: tplDirection } = this.resolved
      const displayPrefix = this.inline ? 'inline-' : ''

      if (mode === 'grid') {
        style.display = `${displayPrefix}grid`

        const cols = this.columns
          ? typeof this.columns === 'number'
            ? `repeat(${this.columns}, 1fr)`
            : this.columns
          : tplColumns

        if (this.autoFill && this.minWidth) {
          style.gridTemplateColumns = `repeat(auto-fill, minmax(${this.minWidth}, 1fr))`
        } else if (this.autoFit && this.minWidth) {
          style.gridTemplateColumns = `repeat(auto-fit, minmax(${this.minWidth}, 1fr))`
        } else if (cols) {
          style.gridTemplateColumns = cols
        }

        if (this.rows) {
          style.gridTemplateRows =
            typeof this.rows === 'number' ? `repeat(${this.rows}, 1fr)` : this.rows
        }

        if (this.areas) {
          const areasValue = Array.isArray(this.areas)
            ? this.areas.map((row) => `"${row}"`).join(' ')
            : this.areas
          style.gridTemplateAreas = areasValue
        }
      } else {
        style.display = `${displayPrefix}flex`

        const dir = this.direction || tplDirection
        if (dir === 'col' || dir === 'column' || dir === 'stack') {
          style.flexDirection = 'column'
        } else if (dir === 'row') {
          style.flexDirection = 'row'
        } else if (dir) {
          style.flexDirection = dir
        }

        style.flexWrap = this.wrap ? 'wrap' : 'nowrap'
      }

      const gapValue = this.normalizeGap(this.gap)
      if (this.rowGap || this.columnGap) {
        if (this.rowGap) style.rowGap = this.normalizeGap(this.rowGap)
        if (this.columnGap) style.columnGap = this.normalizeGap(this.columnGap)
      } else if (gapValue) {
        style.gap = gapValue
      }

      if (this.align) style.alignItems = this.align
      if (this.justify) style.justifyContent = this.justify
      if (this.padding) style.padding = this.padding
      if (this.width) style.width = this.width
      if (this.height) style.height = this.height
      if (this.maxWidth) style.maxWidth = this.maxWidth

      if (this.center) {
        style.marginInline = 'auto'
        if (!this.maxWidth) style.maxWidth = '100%'
      }

      if (this.margin && this.margin !== '0') style.margin = this.margin

      return style
    },
  },

  methods: {
    normalizeGap(value) {
      if (!value && value !== 0) return null
      if (typeof value === 'number') return `${value}px`
      return value
    },

    parseTemplateType(type) {
      const trimmed = type.trim()

      if (FR_PATTERN.test(trimmed)) {
        return { mode: 'grid', columns: trimmed }
      }

      if (COLUMNS_PATTERN.test(trimmed)) {
        return { mode: 'grid', columns: `repeat(${trimmed}, 1fr)` }
      }

      if (FR_WITH_OTHERS.test(trimmed)) {
        return { mode: 'grid', columns: trimmed }
      }

      if (trimmed.includes('repeat(') || trimmed.includes('minmax(')) {
        return { mode: 'grid', columns: trimmed }
      }

      if (/^(flex|row|col|column|stack|wrap)$/.test(trimmed)) {
        return { mode: 'flex', direction: trimmed }
      }

      if (/^grid$/.test(trimmed)) {
        return { mode: 'grid', columns: null }
      }

      return { mode: 'flex', direction: null }
    },
  },
}
</script>

<style scoped>
.layout {
  box-sizing: border-box;
  font-family: var(--font-family, 'DM Sans', sans-serif);
  color: var(--foreground, #1a1a1a);
  background-color: transparent;
}

.layout--grid {
  align-content: start;
}

.layout--flex {
  align-content: flex-start;
}
</style>