<template>
  <div
    ref="tableWrapper"
    :class="[
      {
        'el-table--fit': fit,
        'el-table--striped': stripe,
        'el-table--border': border || isGroup,
        'el-table--hidden': isHidden,
        'el-table--group': isGroup,
        'el-table--fluid-height': maxHeight,
        'el-table--scrollable-x': layout.scrollX.value,
        'el-table--scrollable-y': layout.scrollY.value,
        'el-table--enable-row-hover': !store.states.isComplex.value,
        'el-table--enable-row-transition':
          (store.states.data.value || []).length !== 0 &&
          (store.states.data.value || []).length < 100,
      },
      tableSize ? `el-table--${tableSize}` : '',
      className,
      'el-table',
    ]"
    :style="style"
    @mouseleave="handleMouseLeave()"
  >
    <div class="el-table__inner-wrapper">
      <div ref="hiddenColumns" class="hidden-columns">
        <slot></slot>
      </div>
      <div
        v-if="showHeader"
        ref="headerWrapper"
        v-mousewheel="handleHeaderFooterMousewheel"
        class="el-table__header-wrapper"
      >
        <table-header
          ref="tableHeader"
          :border="border"
          :default-sort="defaultSort"
          :store="store"
          :style="tableBodyStyles"
          @set-drag-visible="setDragVisible"
        />
      </div>
      <div ref="bodyWrapper" :style="bodyHeight" class="el-table__body-wrapper">
        <table-body
          :context="context"
          :highlight="highlightCurrentRow"
          :row-class-name="rowClassName"
          :tooltip-effect="tooltipEffect"
          :row-style="rowStyle"
          :store="store"
          :stripe="stripe"
          :style="{
            width: bodyWidth,
          }"
        />
        <div
          v-if="isEmpty"
          ref="emptyBlock"
          :style="emptyBlockStyle"
          class="el-table__empty-block"
        >
          <span class="el-table__empty-text">
            <slot name="empty">{{ computedEmptyText }}</slot>
          </span>
        </div>
        <div
          v-if="$slots.append"
          ref="appendWrapper"
          class="el-table__append-wrapper"
        >
          <slot name="append"></slot>
        </div>
      </div>
      <div v-if="border || isGroup" class="el-table__border-left-patch"></div>
      <div
        v-if="layout.scrollX.value && layout.height.value"
        class="el-table__border-bottom-patch"
        :style="borderBottomPatchStyles"
      ></div>
    </div>
    <div
      v-if="showSummary"
      v-show="!isEmpty"
      ref="footerWrapper"
      v-mousewheel="handleHeaderFooterMousewheel"
      class="el-table__footer-wrapper"
    >
      <table-footer
        :border="border"
        :default-sort="defaultSort"
        :store="store"
        :style="tableBodyStyles"
        :sum-text="computedSumText"
        :summary-method="summaryMethod"
      />
    </div>
    <div
      v-show="resizeProxyVisible"
      ref="resizeProxy"
      class="el-table__column-resize-proxy"
    ></div>
  </div>
</template>

<script lang="ts">
import { defineComponent, getCurrentInstance, computed } from 'vue'
import debounce from 'lodash/debounce'
import { Mousewheel } from '@element-plus/directives'
import { useLocale } from '@element-plus/hooks'
import { createStore } from './store/helper'
import TableLayout from './table-layout'
import TableHeader from './table-header'
import TableBody from './table-body'
import TableFooter from './table-footer'
import useUtils from './table/utils-helper'
import useStyle from './table/style-helper'
import defaultProps from './table/defaults'

import type { Table } from './table/defaults'

let tableIdSeed = 1
export default defineComponent({
  name: 'ElTable',
  directives: {
    Mousewheel,
  },
  components: {
    TableHeader,
    TableBody,
    TableFooter,
  },
  props: defaultProps,
  emits: [
    'select',
    'select-all',
    'selection-change',
    'cell-mouse-enter',
    'cell-mouse-leave',
    'cell-contextmenu',
    'cell-click',
    'cell-dblclick',
    'row-click',
    'row-contextmenu',
    'row-dblclick',
    'header-click',
    'header-contextmenu',
    'sort-change',
    'filter-change',
    'current-change',
    'header-dragend',
    'expand-change',
  ],
  setup(props) {
    type Row = typeof props.data[number]
    const { t } = useLocale()
    const table = getCurrentInstance() as Table<Row>
    const store = createStore<Row>(table, props)
    table.store = store
    const layout = new TableLayout<Row>({
      store: table.store,
      table,
      fit: props.fit,
      showHeader: props.showHeader,
    })
    table.layout = layout

    const isEmpty = computed(() => (store.states.data.value || []).length === 0)

    /**
     * open functions
     */
    const {
      setCurrentRow,
      toggleRowSelection,
      clearSelection,
      clearFilter,
      toggleAllSelection,
      toggleRowExpansion,
      clearSort,
      sort,
    } = useUtils<Row>(store)
    const {
      isHidden,
      renderExpanded,
      setDragVisible,
      isGroup,
      handleMouseLeave,
      handleHeaderFooterMousewheel,
      tableSize,
      bodyHeight,
      emptyBlockStyle,
      handleFixedMousewheel,
      fixedHeight,
      fixedBodyHeight,
      resizeProxyVisible,
      bodyWidth,
      resizeState,
      doLayout,
      tableBodyStyles,
      borderBottomPatchStyles,
    } = useStyle<Row>(props, layout, store, table)

    const debouncedUpdateLayout = debounce(doLayout, 50)

    const tableId = `el-table_${tableIdSeed++}`
    table.tableId = tableId
    table.state = {
      isGroup,
      resizeState,
      doLayout,
      debouncedUpdateLayout,
    }
    const computedSumText = computed(
      () => props.sumText || t('el.table.sumText')
    )

    const computedEmptyText = computed(() => {
      return props.emptyText || t('el.table.emptyText')
    })

    return {
      layout,
      store,
      handleHeaderFooterMousewheel,
      handleMouseLeave,
      tableId,
      tableSize,
      isHidden,
      isEmpty,
      renderExpanded,
      resizeProxyVisible,
      resizeState,
      isGroup,
      bodyWidth,
      bodyHeight,
      tableBodyStyles,
      emptyBlockStyle,
      borderBottomPatchStyles,
      debouncedUpdateLayout,
      handleFixedMousewheel,
      fixedHeight,
      fixedBodyHeight,
      setCurrentRow,
      toggleRowSelection,
      clearSelection,
      clearFilter,
      toggleAllSelection,
      toggleRowExpansion,
      clearSort,
      doLayout,
      sort,
      t,
      setDragVisible,
      context: table,
      computedSumText,
      computedEmptyText,
    }
  },
})
</script>
