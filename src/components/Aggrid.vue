<script setup lang="ts">
import { ref, shallowRef, defineProps } from 'vue';
import { AgGridVue } from 'ag-grid-vue3';
import { 
  themeQuartz, 
  type FirstDataRenderedEvent, 
  type GridApi, 
  type GridReadyEvent, 
  type GridSizeChangedEvent, 
  type RowHeightParams,
  type ColDef
} from 'ag-grid-community';

// Define a generic props interface for the grid component.
interface AgGridProps<T> {
  // Columns are required and describe the data shape.
  columns: ColDef<T>[];
  // Row data is required and must be compatible with the columns.
  rowData: T[];
  verticalAutoFit?: boolean;
  horizontalAutoFit?: boolean;
}

// Define props without defaults; columns and rowData are required.
const props = defineProps<AgGridProps<any>>();

// --- Dynamic Row Height Logic ---
let minRowHeight = 25;
let currentRowHeight: number;

// Use a shallow ref for the grid API.
const gridApi = shallowRef<GridApi | null>(null);

// --- Column Definitions & Row Data ---
// (Now provided via props; no defaults here)

// Function to update row heights based on available vertical space.
const updateRowHeight = (params: { api: GridApi }) => {
  const bodyViewport = document.querySelector(".ag-body-viewport");
  if (!bodyViewport) return;

  const gridHeight = bodyViewport.clientHeight;
  const renderedRowCount = params.api.getDisplayedRowCount();

  if (renderedRowCount * minRowHeight >= gridHeight) {
    if (currentRowHeight !== minRowHeight) {
      currentRowHeight = minRowHeight;
      params.api.resetRowHeights();
    }
  } else {
    currentRowHeight = Math.floor(gridHeight / renderedRowCount);
    params.api.resetRowHeights();
  }
};

// Provide row height via a callback.
const getRowHeight = ref((params: RowHeightParams) => {
  return currentRowHeight;
});

// --- Grid Events ---
function onFirstDataRendered(params: FirstDataRenderedEvent) {
  if (props.horizontalAutoFit) {
    updateRowHeight(params);
  }
  if (props.verticalAutoFit) {
    params.api.sizeColumnsToFit();
  }
}

function onGridSizeChanged(params: GridSizeChangedEvent) {
  const gridWidth = document.querySelector(".ag-body-viewport")!.clientWidth;
  const columnsToShow: string[] = [];
  const columnsToHide: string[] = [];
  let totalColsWidth = 0;
  const allColumns = params.api.getColumns();
  if (allColumns && allColumns.length > 0) {
    for (let i = 0; i < allColumns.length; i++) {
      const column = allColumns[i];
      totalColsWidth += column.getMinWidth();
      if (totalColsWidth > gridWidth) {
        columnsToHide.push(column.getColId());
      } else {
        columnsToShow.push(column.getColId());
      }
    }
  }
  params.api.setColumnsVisible(columnsToShow, true);
  params.api.setColumnsVisible(columnsToHide, false);
  
  if (props.verticalAutoFit) {
    window.setTimeout(() => {
      params.api.sizeColumnsToFit();
    }, 10);
  }
  if (props.horizontalAutoFit) {
    updateRowHeight(params);
  }
}

const onGridReady = (params: GridReadyEvent) => {
  gridApi.value = params.api;
  // Set minRowHeight based on the current theme if available.
  minRowHeight = params.api.getSizesForCurrentTheme().rowHeight;
  currentRowHeight = minRowHeight;
};

// Optional grid options.
const gridOptions = {
  defaultColDef: {
    resizable: true,
    sortable: true,
    filter: true,
  }
};
</script>

<template>
  <!-- Apply the Quartz theme dynamically -->
  <ag-grid-vue 
    style="width: 100%; height: 100%" 
    :theme="themeQuartz" 
    :columnDefs="props.columns" 
    :rowData="props.rowData"
    :gridOptions="gridOptions" 
    :getRowHeight="getRowHeight"
    @grid-ready="onGridReady"
    @first-data-rendered="onFirstDataRendered"
    @grid-size-changed="onGridSizeChanged"
  />
</template>

<style scoped>
/* Add component-specific styles here if needed */
</style>
