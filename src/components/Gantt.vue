<template>
  <div class="gantt-wrapper">
    <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet" />

    <div class="gantt-toolbar">
      <div class="toolbar-left">
        <button class="nav-btn" @click="shiftRange(-7)" aria-label="Previous week">
          <span class="material-icons">chevron_left</span>
        </button>
        <button class="today-btn" @click="jumpToToday">Today</button>
        <button class="nav-btn" @click="shiftRange(7)" aria-label="Next week">
          <span class="material-icons">chevron_right</span>
        </button>
        <span class="range-label">{{ rangeLabel }}</span>
      </div>
      <div class="toolbar-right">
        <div class="view-toggle">
          <button
            v-for="v in views"
            :key="v.key"
            class="view-btn"
            :class="{ 'view-btn--active': activeView === v.key }"
            @click="setView(v.key)"
          >
            {{ v.label }}
          </button>
        </div>
      </div>
    </div>

    <div class="gantt-scroll-area" ref="scrollArea">
      <div class="gantt-inner" :style="{ minWidth: ganttMinWidth + 'px' }">

        <div class="gantt-head">
          <div class="gantt-label-col"></div>
          <div class="gantt-grid-header">
            <div
              v-for="(col, i) in columns"
              :key="i"
              class="gantt-col-header"
              :class="{ 'col--today': col.isToday, 'col--weekend': col.isWeekend }"
              :style="{ width: colWidth + 'px' }"
            >
              <span class="col-day-name">{{ col.dayName }}</span>
              <span class="col-day-num">{{ col.dayNum }}</span>
            </div>
          </div>
        </div>

        <div class="gantt-month-row">
          <div class="gantt-label-col"></div>
          <div class="gantt-grid-months">
            <div
              v-for="(segment, i) in monthSegments"
              :key="i"
              class="gantt-month-segment"
              :style="{ width: segment.width + 'px' }"
            >
              {{ segment.label }}
            </div>
          </div>
        </div>

        <div
          v-for="task in tasks"
          :key="task.id"
          class="gantt-row"
          :class="{ 'gantt-row--hovered': hoveredTask === task.id }"
          @mouseenter="hoveredTask = task.id"
          @mouseleave="hoveredTask = null"
        >
          <div class="gantt-label-col">
            <div class="task-label">
              <span
                class="task-dot"
                :style="{ background: task.color || 'var(--primary)' }"
              ></span>
              <span class="task-name">{{ task.name }}</span>
            </div>
          </div>
          <div class="gantt-bar-area">
            <div
              v-for="(col, i) in columns"
              :key="i"
              class="gantt-cell"
              :class="{ 'cell--today': col.isToday, 'cell--weekend': col.isWeekend }"
              :style="{ width: colWidth + 'px' }"
            ></div>

            <div
              v-if="getBarData(task)"
              class="gantt-bar"
              :style="getBarStyle(task)"
              @mouseenter="showTooltip(task, $event)"
              @mouseleave="hideTooltip"
            >
              <span class="bar-label">{{ task.name }}</span>
              <div
                v-if="task.progress !== undefined"
                class="bar-progress"
                :style="{ width: task.progress + '%' }"
              ></div>
            </div>
          </div>
        </div>

        <div class="gantt-today-line" :style="todayLineStyle" v-if="todayLineStyle"></div>
      </div>
    </div>

    <Transition name="tooltip-fade">
      <div
        v-if="tooltip.visible"
        class="gantt-tooltip"
        :style="{ top: tooltip.y + 'px', left: tooltip.x + 'px' }"
      >
        <p class="tooltip-title">{{ tooltip.task.name }}</p>
        <p class="tooltip-row">
          <span class="material-icons">calendar_today</span>
          {{ formatDate(tooltip.task.start) }} → {{ formatDate(tooltip.task.end) }}
        </p>
        <p class="tooltip-row" v-if="tooltip.task.progress !== undefined">
          <span class="material-icons">trending_up</span>
          {{ tooltip.task.progress }}% complete
        </p>
        <p class="tooltip-row" v-if="tooltip.task.assignee">
          <span class="material-icons">person</span>
          {{ tooltip.task.assignee }}
        </p>
      </div>
    </Transition>
  </div>
</template>

<script>
export default {
  name: "Gantt",

  props: {
    tasks: {
      type: Array,
      default: () => [
        { id: 1,  name: "Discovery & Research",    start: new Date(2026, 2, 23), end: new Date(2026, 2, 28), progress: 100, assignee: "Anna M.",   color: "#22c98b" },
        { id: 2,  name: "UX Wireframes",           start: new Date(2026, 2, 26), end: new Date(2026, 3, 3),  progress: 90,  assignee: "Laura B.",  color: "#4f8ef7" },
        { id: 3,  name: "UI Design System",        start: new Date(2026, 3, 1),  end: new Date(2026, 3, 8),  progress: 60,  assignee: "Laura B.",  color: "#4f8ef7" },
        { id: 4,  name: "Frontend Setup",          start: new Date(2026, 3, 2),  end: new Date(2026, 3, 5),  progress: 100, assignee: "Marco T.",  color: "#ED202F" },
        { id: 5,  name: "Component Library",       start: new Date(2026, 3, 4),  end: new Date(2026, 3, 11), progress: 55,  assignee: "Marco T.",  color: "#ED202F" },
        { id: 6,  name: "API Integration",         start: new Date(2026, 3, 6),  end: new Date(2026, 3, 14), progress: 30,  assignee: "Carlo V.",  color: "#f59e0b" },
        { id: 7,  name: "Auth & Permissions",      start: new Date(2026, 3, 7),  end: new Date(2026, 3, 10), progress: 70,  assignee: "Carlo V.",  color: "#f59e0b" },
        { id: 8,  name: "Dashboard Views",         start: new Date(2026, 3, 9),  end: new Date(2026, 3, 17), progress: 20,  assignee: "Marco T.",  color: "#ED202F" },
        { id: 9,  name: "QA & Testing",            start: new Date(2026, 3, 14), end: new Date(2026, 3, 21), progress: 0,   assignee: "Anna M.",   color: "#a855f7" },
        { id: 10, name: "Staging Deploy",          start: new Date(2026, 3, 20), end: new Date(2026, 3, 22), progress: 0,   assignee: "Carlo V.",  color: "#f59e0b" },
        { id: 11, name: "Client Review",           start: new Date(2026, 3, 22), end: new Date(2026, 3, 24), progress: 0,   assignee: "Anna M.",   color: "#22c98b" },
        { id: 12, name: "Bug Fixes",               start: new Date(2026, 3, 24), end: new Date(2026, 3, 28), progress: 0,   assignee: "Marco T.",  color: "#ED202F" },
        { id: 13, name: "Production Deploy",       start: new Date(2026, 3, 29), end: new Date(2026, 3, 30), progress: 0,   assignee: "Carlo V.",  color: "#f59e0b" },
      ],
    },
    initialStartDate: {
      type: Date,
      default: null,
    },
    colWidth: {
      type: Number,
      default: 44,
    },
  },

  data() {
    const start = this.initialStartDate
      ? new Date(this.initialStartDate)
      : (() => { const d = new Date(); d.setDate(d.getDate() - 3); return d; })();
    start.setHours(0, 0, 0, 0);

    return {
      viewStart: start,
      activeView: "2w",
      hoveredTask: null,
      tooltip: { visible: false, task: null, x: 0, y: 0 },
      views: [
        { key: "1w", label: "1W", days: 7 },
        { key: "2w", label: "2W", days: 14 },
        { key: "1m", label: "1M", days: 30 },
        { key: "3m", label: "3M", days: 90 },
      ],
    };
  },

  computed: {
    viewDays() {
      return this.views.find((v) => v.key === this.activeView)?.days || 14;
    },

    viewEnd() {
      const d = new Date(this.viewStart);
      d.setDate(d.getDate() + this.viewDays - 1);
      return d;
    },

    columns() {
      const cols = [];
      const today = new Date();
      today.setHours(0, 0, 0, 0);
      let cur = new Date(this.viewStart);
      while (cur <= this.viewEnd) {
        const dow = cur.getDay();
        cols.push({
          date: new Date(cur),
          dayName: cur.toLocaleDateString("en", { weekday: "short" }),
          dayNum: cur.getDate(),
          isToday: cur.toDateString() === today.toDateString(),
          isWeekend: dow === 0 || dow === 6,
        });
        cur.setDate(cur.getDate() + 1);
      }
      return cols;
    },

    monthSegments() {
      const segments = [];
      let i = 0;
      while (i < this.columns.length) {
        const col = this.columns[i];
        const month = col.date.getMonth();
        const year = col.date.getFullYear();
        let count = 0;
        while (i < this.columns.length && this.columns[i].date.getMonth() === month) {
          count++;
          i++;
        }
        segments.push({
          label: new Date(year, month, 1).toLocaleDateString("en", { month: "long", year: "numeric" }),
          width: count * this.colWidth,
        });
      }
      return segments;
    },

    ganttMinWidth() {
      return 160 + this.columns.length * this.colWidth;
    },

    rangeLabel() {
      return (
        this.viewStart.toLocaleDateString("en", { day: "numeric", month: "short" }) +
        " – " +
        this.viewEnd.toLocaleDateString("en", { day: "numeric", month: "short", year: "numeric" })
      );
    },

    todayLineStyle() {
      const today = new Date();
      today.setHours(0, 0, 0, 0);
      if (today < this.viewStart || today > this.viewEnd) return null;
      const idx = this.columns.findIndex((c) => c.isToday);
      if (idx === -1) return null;
      const left = 160 + idx * this.colWidth + this.colWidth / 2;
      return { left: left + "px" };
    },
  },

  methods: {
    shiftRange(days) {
      const d = new Date(this.viewStart);
      d.setDate(d.getDate() + days);
      this.viewStart = d;
    },

    jumpToToday() {
      const d = new Date();
      d.setHours(0, 0, 0, 0);
      d.setDate(d.getDate() - 3);
      this.viewStart = d;
    },

    setView(key) {
      this.activeView = key;
    },

    daysBetween(a, b) {
      return Math.round((b - a) / (1000 * 60 * 60 * 24));
    },

    getBarData(task) {
      const taskStart = new Date(task.start);
      const taskEnd = new Date(task.end);
      taskStart.setHours(0, 0, 0, 0);
      taskEnd.setHours(0, 0, 0, 0);
      if (taskEnd < this.viewStart || taskStart > this.viewEnd) return null;
      return true;
    },

    getBarStyle(task) {
      const taskStart = new Date(task.start);
      const taskEnd = new Date(task.end);
      taskStart.setHours(0, 0, 0, 0);
      taskEnd.setHours(0, 0, 0, 0);

      const clampedStart = taskStart < this.viewStart ? this.viewStart : taskStart;
      const clampedEnd = taskEnd > this.viewEnd ? this.viewEnd : taskEnd;

      const startOffset = this.daysBetween(this.viewStart, clampedStart);
      const duration = this.daysBetween(clampedStart, clampedEnd) + 1;

      return {
        left: startOffset * this.colWidth + "px",
        width: duration * this.colWidth - 6 + "px",
        background: task.color || "var(--primary)",
      };
    },

    formatDate(date) {
      return new Date(date).toLocaleDateString("en", { day: "numeric", month: "short", year: "numeric" });
    },

    showTooltip(task, event) {
      const rect = this.$el.getBoundingClientRect();
      this.tooltip = {
        visible: true,
        task,
        x: event.clientX - rect.left + 12,
        y: event.clientY - rect.top - 10,
      };
    },

    hideTooltip() {
      this.tooltip.visible = false;
    },
  },
};
</script>

<style scoped>
.gantt-wrapper {
  width: 100%;
  font-family: var(--font-family, 'DM Sans', sans-serif);
  color: var(--foreground, #1a1a1a);
  border-radius: 12px;
  overflow: hidden;
  border: 1.5px solid color-mix(in srgb, var(--foreground, #1a1a1a) 12%, transparent);
  position: relative;
}

.gantt-toolbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 12px 16px;
  background: var(--background);
  border-bottom: 1.5px solid color-mix(in srgb, var(--foreground, #1a1a1a) 10%, transparent);
  gap: 12px;
  flex-wrap: wrap;
}

.toolbar-left {
  display: flex;
  align-items: center;
  gap: 8px;
}

.toolbar-right {
  display: flex;
  align-items: center;
  gap: 10px;
}

.nav-btn {
  width: 34px;
  height: 34px;
  border-radius: 8px;
  border: 1.5px solid color-mix(in srgb, var(--foreground, #1a1a1a) 18%, transparent);
  background: color-mix(in srgb, var(--foreground, #1a1a1a) 5%, transparent);
  color: var(--foreground, #1a1a1a);
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: background 0.15s;
}

.nav-btn:hover {
  background: color-mix(in srgb, var(--foreground, #1a1a1a) 12%, transparent);
}

.nav-btn .material-icons {
  font-size: 20px;
}

.today-btn {
  padding: 6px 14px;
  border-radius: 8px;
  border: 1.5px solid color-mix(in srgb, var(--foreground, #1a1a1a) 18%, transparent);
  background: color-mix(in srgb, var(--foreground, #1a1a1a) 5%, transparent);
  color: var(--foreground, #1a1a1a);
  font-family: var(--font-family, 'DM Sans', sans-serif);
  font-size: 0.8rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.15s;
}

.today-btn:hover {
  background: color-mix(in srgb, var(--foreground, #1a1a1a) 12%, transparent);
}

.range-label {
  font-size: 0.82rem;
  font-weight: 600;
  color: color-mix(in srgb, var(--foreground, #1a1a1a) 60%, transparent);
  white-space: nowrap;
}

.view-toggle {
  display: flex;
  border-radius: 8px;
  overflow: hidden;
  border: 1.5px solid color-mix(in srgb, var(--foreground, #1a1a1a) 18%, transparent);
}

.view-btn {
  padding: 6px 13px;
  border: none;
  border-right: 1.5px solid color-mix(in srgb, var(--foreground, #1a1a1a) 18%, transparent);
  background: color-mix(in srgb, var(--foreground, #1a1a1a) 4%, transparent);
  color: var(--foreground, #1a1a1a);
  font-family: var(--font-family, 'DM Sans', sans-serif);
  font-size: 0.78rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.15s, color 0.15s;
}

.view-btn:last-child {
  border-right: none;
}

.view-btn:hover:not(.view-btn--active) {
  background: color-mix(in srgb, var(--foreground, #1a1a1a) 10%, transparent);
}

.view-btn--active {
  background: var(--primary, #ED202F);
  color: var(--foreground);
}

.gantt-scroll-area {
  overflow-x: auto;
  background: var(--background, #dfdfdf);
}

.gantt-inner {
  position: relative;
}

.gantt-head {
  display: flex;
  position: sticky;
  top: 0;
  z-index: 10;
  background: var(--background, #dfdfdf);
  border-bottom: 1.5px solid color-mix(in srgb, var(--foreground, #1a1a1a) 12%, transparent);
}

.gantt-month-row {
  display: flex;
  border-bottom: 1px solid color-mix(in srgb, var(--foreground, #1a1a1a) 8%, transparent);
  background: color-mix(in srgb, var(--foreground, #1a1a1a) 2%, transparent);
}

.gantt-grid-months {
  display: flex;
}

.gantt-month-segment {
  font-size: 0.7rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: color-mix(in srgb, var(--foreground, #1a1a1a) 50%, transparent);
  padding: 4px 8px;
  white-space: nowrap;
  overflow: hidden;
  border-right: 1px solid color-mix(in srgb, var(--foreground, #1a1a1a) 8%, transparent);
}

.gantt-label-col {
  width: 160px;
  min-width: 160px;
  flex-shrink: 0;
  padding: 0 12px;
  display: flex;
  align-items: center;
  border-right: 1.5px solid color-mix(in srgb, var(--foreground, #1a1a1a) 10%, transparent);
  background: var(--background, #dfdfdf);
}

.gantt-grid-header {
  display: flex;
}

.gantt-col-header {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 6px 0;
  gap: 1px;
  flex-shrink: 0;
  border-right: 1px solid color-mix(in srgb, var(--foreground, #1a1a1a) 6%, transparent);
}

.gantt-col-header.col--weekend {
  background: color-mix(in srgb, var(--foreground, #1a1a1a) 4%, transparent);
}

.gantt-col-header.col--today {
  background: color-mix(in srgb, var(--primary, #ED202F) 10%, transparent);
}

.col-day-name {
  font-size: 0.65rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.04em;
  color: color-mix(in srgb, var(--foreground, #1a1a1a) 45%, transparent);
}

.col-day-num {
  font-size: 0.85rem;
  font-weight: 700;
  color: var(--foreground, #1a1a1a);
}

.col--today .col-day-num {
  color: var(--primary, #ED202F);
}

.col--weekend .col-day-num {
  color: color-mix(in srgb, var(--foreground, #1a1a1a) 50%, transparent);
}

.gantt-row {
  display: flex;
  align-items: center;
  height: 52px;
  border-bottom: 1px solid color-mix(in srgb, var(--foreground, #1a1a1a) 7%, transparent);
  transition: background 0.12s;
}

.gantt-row--hovered {
  background: color-mix(in srgb, var(--foreground) 3%, transparent);
}

.gantt-row:last-child {
  border-bottom: none;
}

.task-label {
  display: flex;
  align-items: center;
  gap: 8px;
  width: 100%;
  overflow: hidden;
}

.task-dot {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  flex-shrink: 0;
}

.task-name {
  font-size: 0.82rem;
  font-weight: 600;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  color: var(--foreground, #1a1a1a);
}

.gantt-bar-area {
  flex: 1;
  position: relative;
  height: 100%;
  display: flex;
  align-items: center;
}

.gantt-cell {
  height: 100%;
  flex-shrink: 0;
  border-right: 1px solid color-mix(in srgb, var(--foreground, #1a1a1a) 5%, transparent);
}

.gantt-cell.cell--weekend {
  background: color-mix(in srgb, var(--foreground, #1a1a1a) 3%, transparent);
}

.gantt-cell.cell--today {
  background: color-mix(in srgb, var(--primary, #ED202F) 6%, transparent);
}

.gantt-bar {
  position: absolute;
  height: 28px;
  border-radius: 6px;
  display: flex;
  align-items: center;
  padding: 0 10px;
  cursor: pointer;
  overflow: hidden;
  transition: filter 0.15s, transform 0.12s;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.15);
}

.gantt-bar:hover {
  filter: brightness(1.1);
  transform: scaleY(1.06);
}

.bar-label {
  font-size: 0.72rem;
  font-weight: 700;
  color: var(--foreground);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  position: relative;
  z-index: 1;
}

.bar-progress {
  position: absolute;
  left: 0;
  top: 0;
  height: 100%;
  background: rgba(255, 255, 255, 0.22);
  border-radius: 6px 0 0 6px;
  pointer-events: none;
}

.gantt-today-line {
  position: absolute;
  top: 0;
  bottom: 0;
  width: 2px;
  background: var(--primary, #ED202F);
  opacity: 0.7;
  pointer-events: none;
  z-index: 5;
}

.gantt-today-line::before {
  content: "";
  position: absolute;
  top: 0;
  left: 50%;
  transform: translateX(-50%);
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: var(--primary, #ED202F);
}

.gantt-tooltip {
  position: absolute;
  z-index: 100;
  background: var(--background);
  color: var(--foreground);
  border-radius: 10px;
  padding: 10px 14px;
  min-width: 180px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.18);
  pointer-events: none;
}

.tooltip-title {
  font-size: 0.85rem;
  font-weight: 700;
  margin: 0 0 8px;
}

.tooltip-row {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 0.78rem;
  margin: 0 0 4px;
  opacity: 0.85;
}

.tooltip-row:last-child {
  margin-bottom: 0;
}

.tooltip-row .material-icons {
  font-size: 14px;
  opacity: 0.7;
}

.tooltip-fade-enter-active,
.tooltip-fade-leave-active {
  transition: opacity 0.15s ease, transform 0.15s ease;
}

.tooltip-fade-enter-from,
.tooltip-fade-leave-to {
  opacity: 0;
  transform: translateY(4px);
}
</style>