<template>
  <div class="data-table-wrapper">
    <link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet" />

    <div class="table-toolbar">
      <div class="search-wrapper">
        <span class="material-icons search-icon">search</span>
        <input
          v-model="globalSearch"
          type="text"
          class="search-input"
          :placeholder="searchPlaceholder"
        />
        <button v-if="globalSearch" class="search-clear" @click="globalSearch = ''">
          <span class="material-icons">close</span>
        </button>
      </div>

      <div class="toolbar-right">
        <div class="column-toggle-wrapper" v-if="allowColumnToggle">
          <button class="toolbar-btn" @click="columnMenuOpen = !columnMenuOpen">
            <span class="material-icons">view_column</span>
            <span>Columns</span>
          </button>
          <Transition name="dropdown-fade">
            <div v-if="columnMenuOpen" class="column-menu" v-click-outside="() => columnMenuOpen = false">
              <p class="column-menu-title">Toggle Columns</p>
              <label
                v-for="col in columns"
                :key="col.key"
                class="column-menu-item"
              >
                <input
                  type="checkbox"
                  :checked="visibleColumns.includes(col.key)"
                  @change="toggleColumn(col.key)"
                  class="column-checkbox"
                />
                <span class="column-menu-label">{{ col.label }}</span>
              </label>
            </div>
          </Transition>
        </div>

        <div class="rows-per-page" v-if="allowPagination">
          <span class="rows-label">Rows</span>
          <select v-model="rowsPerPage" class="rows-select">
            <option v-for="opt in rowsPerPageOptions" :key="opt" :value="opt">{{ opt }}</option>
          </select>
        </div>
      </div>
    </div>

    <div class="table-container">
      <table class="data-table">
        <thead>
          <tr>
            <th
              v-for="col in activeColumns"
              :key="col.key"
              class="th"
              :class="{ sortable: col.sortable !== false, 'th--sorted': sortKey === col.key }"
              :style="col.width ? { width: col.width } : {}"
              @click="col.sortable !== false && setSort(col.key)"
            >
              <div class="th-inner">
                <span class="th-label">{{ col.label }}</span>
                <span
                  v-if="col.sortable !== false"
                  class="material-icons sort-icon"
                  :class="{
                    'sort-icon--active': sortKey === col.key,
                    'sort-icon--desc': sortKey === col.key && sortDir === 'desc'
                  }"
                >
                  {{ sortKey === col.key ? (sortDir === 'asc' ? 'arrow_upward' : 'arrow_downward') : 'unfold_more' }}
                </span>
              </div>
              <div class="th-filter" v-if="allowColumnFilters" @click.stop>
                <input
                  v-model="columnFilters[col.key]"
                  type="text"
                  class="col-filter-input"
                  :placeholder="'Filter ' + col.label"
                />
              </div>
            </th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-if="paginatedRows.length === 0"
          >
            <td :colspan="activeColumns.length" class="empty-cell">
              <div class="empty-state">
                <span class="material-icons empty-icon">search_off</span>
                <p>No results found</p>
              </div>
            </td>
          </tr>
          <tr
            v-for="(row, rowIndex) in paginatedRows"
            :key="rowIndex"
            class="tr"
            :class="{ 'tr--even': rowIndex % 2 === 0 }"
            @click="$emit('row-click', row)"
          >
            <td
              v-for="col in activeColumns"
              :key="col.key"
              class="td"
              :class="col.align ? 'td--' + col.align : ''"
            >
              <slot :name="'cell-' + col.key" :value="row[col.key]" :row="row">
                <span
                  v-if="col.badge"
                  class="badge"
                  :style="getBadgeStyle(row[col.key], col)"
                >
                  {{ row[col.key] }}
                </span>
                <span v-else class="cell-text">{{ formatCell(row[col.key], col) }}</span>
              </slot>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <div class="table-footer" v-if="allowPagination">
      <span class="footer-info">
        {{ paginationInfo }}
      </span>
      <div class="pagination">
        <button
          class="page-btn"
          :disabled="currentPage === 1"
          @click="currentPage = 1"
          aria-label="First page"
        >
          <span class="material-icons">first_page</span>
        </button>
        <button
          class="page-btn"
          :disabled="currentPage === 1"
          @click="currentPage--"
          aria-label="Previous page"
        >
          <span class="material-icons">chevron_left</span>
        </button>
        <button
          v-for="page in visiblePages"
          :key="page"
          class="page-btn page-num"
          :class="{ 'page-btn--active': page === currentPage, 'page-btn--ellipsis': page === '...' }"
          :disabled="page === '...'"
          @click="page !== '...' && (currentPage = page)"
        >
          {{ page }}
        </button>
        <button
          class="page-btn"
          :disabled="currentPage === totalPages"
          @click="currentPage++"
          aria-label="Next page"
        >
          <span class="material-icons">chevron_right</span>
        </button>
        <button
          class="page-btn"
          :disabled="currentPage === totalPages"
          @click="currentPage = totalPages"
          aria-label="Last page"
        >
          <span class="material-icons">last_page</span>
        </button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "DataTable",

  directives: {
    clickOutside: {
      mounted(el, binding) {
        el._clickOutsideHandler = (e) => {
          if (!el.contains(e.target)) binding.value(e);
        };
        document.addEventListener("click", el._clickOutsideHandler);
      },
      unmounted(el) {
        document.removeEventListener("click", el._clickOutsideHandler);
      },
    },
  },

  props: {
    columns: {
      type: Array,
      required: true,
    },
    rows: {
      type: Array,
      default: () => [],
    },
    searchPlaceholder: {
      type: String,
      default: "Search...",
    },
    allowColumnFilters: {
      type: Boolean,
      default: true,
    },
    allowColumnToggle: {
      type: Boolean,
      default: true,
    },
    allowPagination: {
      type: Boolean,
      default: true,
    },
    rowsPerPageOptions: {
      type: Array,
      default: () => [10, 25, 50, 100],
    },
    defaultRowsPerPage: {
      type: Number,
      default: 10,
    },
  },

  emits: ["row-click"],

  data() {
    return {
      globalSearch: "",
      columnFilters: {},
      sortKey: null,
      sortDir: "asc",
      currentPage: 1,
      rowsPerPage: this.defaultRowsPerPage,
      visibleColumns: this.columns.map((c) => c.key),
      columnMenuOpen: false,
    };
  },

  computed: {
    activeColumns() {
      return this.columns.filter((c) => this.visibleColumns.includes(c.key));
    },

    filteredRows() {
      let result = [...this.rows];

      if (this.globalSearch.trim()) {
        const q = this.globalSearch.toLowerCase();
        result = result.filter((row) =>
          this.columns.some((col) =>
            String(row[col.key] ?? "").toLowerCase().includes(q)
          )
        );
      }

      this.columns.forEach((col) => {
        const f = (this.columnFilters[col.key] || "").trim().toLowerCase();
        if (f) {
          result = result.filter((row) =>
            String(row[col.key] ?? "").toLowerCase().includes(f)
          );
        }
      });

      if (this.sortKey) {
        result.sort((a, b) => {
          const av = a[this.sortKey] ?? "";
          const bv = b[this.sortKey] ?? "";
          const cmp = typeof av === "number" && typeof bv === "number"
            ? av - bv
            : String(av).localeCompare(String(bv));
          return this.sortDir === "asc" ? cmp : -cmp;
        });
      }

      return result;
    },

    totalPages() {
      return Math.max(1, Math.ceil(this.filteredRows.length / this.rowsPerPage));
    },

    paginatedRows() {
      if (!this.allowPagination) return this.filteredRows;
      const start = (this.currentPage - 1) * this.rowsPerPage;
      return this.filteredRows.slice(start, start + this.rowsPerPage);
    },

    paginationInfo() {
      const total = this.filteredRows.length;
      if (total === 0) return "No results";
      const start = (this.currentPage - 1) * this.rowsPerPage + 1;
      const end = Math.min(this.currentPage * this.rowsPerPage, total);
      return `${start}–${end} of ${total} rows`;
    },

    visiblePages() {
      const pages = [];
      const total = this.totalPages;
      const cur = this.currentPage;
      if (total <= 7) {
        for (let i = 1; i <= total; i++) pages.push(i);
      } else {
        pages.push(1);
        if (cur > 3) pages.push("...");
        for (let i = Math.max(2, cur - 1); i <= Math.min(total - 1, cur + 1); i++) pages.push(i);
        if (cur < total - 2) pages.push("...");
        pages.push(total);
      }
      return pages;
    },
  },

  watch: {
    globalSearch() {
      this.currentPage = 1;
    },
    columnFilters: {
      deep: true,
      handler() {
        this.currentPage = 1;
      },
    },
    rowsPerPage() {
      this.currentPage = 1;
    },
  },

  methods: {
    setSort(key) {
      if (this.sortKey === key) {
        this.sortDir = this.sortDir === "asc" ? "desc" : "asc";
      } else {
        this.sortKey = key;
        this.sortDir = "asc";
      }
    },

    toggleColumn(key) {
      if (this.visibleColumns.includes(key)) {
        if (this.visibleColumns.length > 1) {
          this.visibleColumns = this.visibleColumns.filter((k) => k !== key);
        }
      } else {
        this.visibleColumns = this.columns.map((c) => c.key).filter(
          (k) => k === key || this.visibleColumns.includes(k)
        );
      }
    },

    formatCell(value, col) {
      if (value === null || value === undefined) return "—";
      if (col.format) return col.format(value);
      return value;
    },

    getBadgeStyle(value, col) {
      if (col.badgeMap && col.badgeMap[value]) {
        return {
          background: col.badgeMap[value].bg || "color-mix(in srgb, var(--foreground) 10%, transparent)",
          color: col.badgeMap[value].color || "var(--foreground)",
        };
      }
      return {};
    },
  },
};
</script>

<style scoped>
.data-table-wrapper {
  width: 100%;
  font-family: var(--font-family);
  color: var(--foreground);
  display: flex;
  flex-direction: column;
  gap: 0;
  border-radius: 12px;
  overflow: hidden;
  border: 1.5px solid color-mix(in srgb, var(--foreground) 12%, transparent);
}

.table-toolbar {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 12px;
  padding: 14px 16px;
  background: var(--background);
  border-bottom: 1.5px solid color-mix(in srgb, var(--foreground) 10%, transparent);
  flex-wrap: wrap;
}

.search-wrapper {
  position: relative;
  display: flex;
  align-items: center;
  flex: 1;
  min-width: 180px;
  max-width: 340px;
}

.search-icon {
  position: absolute;
  left: 10px;
  font-size: 18px;
  color: color-mix(in srgb, var(--foreground) 40%, transparent);
  pointer-events: none;
}

.search-input {
  width: 100%;
  padding: 8px 36px 8px 36px;
  border-radius: 8px;
  border: 1.5px solid color-mix(in srgb, var(--foreground) 18%, transparent);
  background: color-mix(in srgb, var(--foreground) 5%, transparent);
  color: var(--foreground);
  font-family: var(--font-family);
  font-size: 0.875rem;
  outline: none;
  transition: border-color 0.2s, box-shadow 0.2s;
}

.search-input:focus {
  border-color: var(--primary);
  box-shadow: 0 0 0 3px color-mix(in srgb, var(--primary) 12%, transparent);
}

.search-input::placeholder {
  color: color-mix(in srgb, var(--foreground) 35%, transparent);
}

.search-clear {
  position: absolute;
  right: 8px;
  background: none;
  border: none;
  cursor: pointer;
  color: color-mix(in srgb, var(--foreground) 45%, transparent);
  display: flex;
  align-items: center;
  padding: 2px;
  border-radius: 50%;
  transition: color 0.15s;
}

.search-clear:hover {
  color: var(--foreground);
}

.search-clear .material-icons {
  font-size: 16px;
}

.toolbar-right {
  display: flex;
  align-items: center;
  gap: 10px;
}

.column-toggle-wrapper {
  position: relative;
}

.toolbar-btn {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 7px 12px;
  border-radius: 8px;
  border: 1.5px solid color-mix(in srgb, var(--foreground) 18%, transparent);
  background: color-mix(in srgb, var(--foreground) 5%, transparent);
  color: var(--foreground);
  font-family: var(--font-family);
  font-size: 0.8rem;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.15s, border-color 0.15s;
}

.toolbar-btn:hover {
  background: color-mix(in srgb, var(--foreground) 10%, transparent);
  border-color: color-mix(in srgb, var(--foreground) 28%, transparent);
}

.toolbar-btn .material-icons {
  font-size: 18px;
}

.column-menu {
  position: absolute;
  right: 0;
  top: calc(100% + 6px);
  background: var(--background);
  border: 1.5px solid color-mix(in srgb, var(--foreground) 14%, transparent);
  border-radius: 10px;
  padding: 10px 0 8px;
  min-width: 180px;
  z-index: 100;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.12);
}

.column-menu-title {
  font-size: 0.7rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  color: color-mix(in srgb, var(--foreground) 45%, transparent);
  padding: 0 14px 8px;
  margin: 0;
  border-bottom: 1px solid color-mix(in srgb, var(--foreground) 10%, transparent);
}

.column-menu-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 7px 14px;
  cursor: pointer;
  transition: background 0.12s;
}

.column-menu-item:hover {
  background: color-mix(in srgb, var(--foreground) 6%, transparent);
}

.column-checkbox {
  accent-color: var(--primary);
  width: 15px;
  height: 15px;
  cursor: pointer;
}

.column-menu-label {
  font-size: 0.85rem;
  color: var(--foreground);
}

.rows-per-page {
  display: flex;
  align-items: center;
  gap: 7px;
}

.rows-label {
  font-size: 0.8rem;
  color: color-mix(in srgb, var(--foreground) 55%, transparent);
  font-weight: 500;
  white-space: nowrap;
}

.rows-select {
  padding: 6px 28px 6px 10px;
  border-radius: 8px;
  border: 1.5px solid color-mix(in srgb, var(--foreground) 18%, transparent);
  background: color-mix(in srgb, var(--foreground) 5%, transparent);
  color: var(--foreground);
  font-family: var(--font-family);
  font-size: 0.8rem;
  font-weight: 600;
  outline: none;
  cursor: pointer;
  appearance: none;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='14' height='14' viewBox='0 0 24 24' fill='%231a1a1a'%3E%3Cpath d='M7 10l5 5 5-5z'/%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: right 7px center;
}

.table-container {
  overflow-x: auto;
  background: var(--background);
}

.data-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 0.875rem;
}

.th {
  background: color-mix(in srgb, var(--foreground) 5%, transparent);
  padding: 0;
  border-bottom: 1.5px solid color-mix(in srgb, var(--foreground) 12%, transparent);
  white-space: nowrap;
  user-select: none;
}

.th.sortable {
  cursor: pointer;
}

.th.sortable:hover .th-inner {
  background: color-mix(in srgb, var(--foreground) 8%, transparent);
}

.th--sorted .th-inner {
  background: color-mix(in srgb, var(--primary) 8%, transparent);
}

.th-inner {
  display: flex;
  align-items: center;
  gap: 4px;
  padding: 11px 14px 8px;
  transition: background 0.15s;
}

.th-label {
  font-size: 0.75rem;
  font-weight: 700;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: color-mix(in srgb, var(--foreground) 70%, transparent);
}

.th--sorted .th-label {
  color: var(--primary);
}

.sort-icon {
  font-size: 14px;
  color: color-mix(in srgb, var(--foreground) 30%, transparent);
  transition: color 0.15s, transform 0.2s;
}

.sort-icon--active {
  color: var(--primary);
}

.th-filter {
  padding: 0 8px 8px;
}

.col-filter-input {
  width: 100%;
  padding: 5px 8px;
  border-radius: 6px;
  border: 1.5px solid color-mix(in srgb, var(--foreground) 14%, transparent);
  background: var(--background);
  color: var(--foreground);
  font-family: var(--font-family);
  font-size: 0.78rem;
  outline: none;
  box-sizing: border-box;
  transition: border-color 0.2s;
}

.col-filter-input:focus {
  border-color: var(--primary);
}

.col-filter-input::placeholder {
  color: color-mix(in srgb, var(--foreground) 28%, transparent);
  font-size: 0.75rem;
}

.tr {
  border-bottom: 1px solid color-mix(in srgb, var(--foreground) 7%, transparent);
  transition: background 0.12s;
  cursor: pointer;
}

.tr:last-child {
  border-bottom: none;
}

.tr:hover {
  background: color-mix(in srgb, var(--primary) 5%, transparent);
}

.tr--even {
  background: color-mix(in srgb, var(--foreground) 2%, transparent);
}

.tr--even:hover {
  background: color-mix(in srgb, var(--primary) 5%, transparent);
}

.td {
  padding: 12px 14px;
  color: var(--foreground);
  vertical-align: middle;
}

.td--center {
  text-align: center;
}

.td--right {
  text-align: right;
}

.cell-text {
  font-size: 0.875rem;
  line-height: 1.4;
}

.badge {
  display: inline-flex;
  align-items: center;
  padding: 3px 10px;
  border-radius: 100px;
  font-size: 0.75rem;
  font-weight: 600;
  background: color-mix(in srgb, var(--foreground) 10%, transparent);
  color: var(--foreground);
  white-space: nowrap;
}

.empty-cell {
  padding: 0;
}

.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
  padding: 48px 20px;
  color: color-mix(in srgb, var(--foreground) 35%, transparent);
}

.empty-icon {
  font-size: 36px;
}

.empty-state p {
  font-size: 0.9rem;
  margin: 0;
}

.table-footer {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 12px 16px;
  background: var(--background);
  border-top: 1.5px solid color-mix(in srgb, var(--foreground) 10%, transparent);
  flex-wrap: wrap;
  gap: 10px;
}

.footer-info {
  font-size: 0.8rem;
  color: color-mix(in srgb, var(--foreground) 55%, transparent);
  font-weight: 500;
}

.pagination {
  display: flex;
  align-items: center;
  gap: 4px;
}

.page-btn {
  min-width: 32px;
  height: 32px;
  border-radius: 7px;
  border: 1.5px solid color-mix(in srgb, var(--foreground) 14%, transparent);
  background: color-mix(in srgb, var(--foreground) 4%, transparent);
  color: var(--foreground);
  font-family: var(--font-family);
  font-size: 0.8rem;
  font-weight: 600;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  transition: background 0.15s, border-color 0.15s, color 0.15s;
  padding: 0 6px;
}

.page-btn .material-icons {
  font-size: 18px;
}

.page-btn:hover:not(:disabled) {
  background: color-mix(in srgb, var(--foreground) 10%, transparent);
  border-color: color-mix(in srgb, var(--foreground) 25%, transparent);
}

.page-btn:disabled {
  opacity: 0.35;
  cursor: not-allowed;
}

.page-btn--active {
  background: var(--primary);
  border-color: var(--primary);
  color: var(--foreground);
}

.page-btn--active:hover:not(:disabled) {
  background: var(--primary);
  opacity: 0.88;
}

.page-btn--ellipsis {
  border-color: transparent;
  background: transparent;
  cursor: default;
  letter-spacing: 0.05em;
}

.dropdown-fade-enter-active,
.dropdown-fade-leave-active {
  transition: opacity 0.15s ease, transform 0.15s ease;
}

.dropdown-fade-enter-from,
.dropdown-fade-leave-to {
  opacity: 0;
  transform: translateY(-6px);
}
</style>