<script setup lang="ts">
import { ref, shallowRef, defineProps, watch } from 'vue';
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

// Extend the props interface to include isLoading
interface AgGridProps<T> {
  // Columns are required and describe the data shape.
  columns: ColDef<T>[];
  // Row data is required and must be compatible with the columns.
  rowData: T[] | null;
  verticalAutoFit?: boolean;
  horizontalAutoFit?: boolean;
  isLoading: boolean;
}

// Define props; now isLoading is required.
const props = defineProps<AgGridProps<any>>();

// --- Dynamic Row Height Logic ---
let minRowHeight = 25;
let currentRowHeight: number;

// Use a shallow ref for the grid API.
const gridApi = shallowRef<GridApi | null>(null);

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
  minRowHeight = params.api.getSizesForCurrentTheme().rowHeight;
  currentRowHeight = minRowHeight;

  gridApi.value.setGridOption('loading', props.isLoading);
};

// --- Custom Templates for Overlays ---
const emptyStateTemplate = `
  <div class="flex flex-col items-center">
    <img class="h-48" src="/empty-table.svg" alt="empty-table" />
    <p class="text-xl">There is no data.</p>
  </div>
`;

const loadingTemplate = `
  <div class="flex flex-col items-center">
    <svg aria-hidden="true" class="w-8 h-8 text-gray-200 animate-spin fill-green-600" viewBox="0 0 100 101" fill="none" xmlns="http://www.w3.org/2000/svg">
        <path d="M100 50.5908C100 78.2051 77.6142 100.591 50 100.591C22.3858 100.591 0 78.2051 0 50.5908C0 22.9766 22.3858 0.59082 50 0.59082C77.6142 0.59082 100 22.9766 100 50.5908ZM9.08144 50.5908C9.08144 73.1895 27.4013 91.5094 50 91.5094C72.5987 91.5094 90.9186 73.1895 90.9186 50.5908C90.9186 27.9921 72.5987 9.67226 50 9.67226C27.4013 9.67226 9.08144 27.9921 9.08144 50.5908Z" fill="currentColor"/>
        <path d="M93.9676 39.0409C96.393 38.4038 97.8624 35.9116 97.0079 33.5539C95.2932 28.8227 92.871 24.3692 89.8167 20.348C85.8452 15.1192 80.8826 10.7238 75.2124 7.41289C69.5422 4.10194 63.2754 1.94025 56.7698 1.05124C51.7666 0.367541 46.6976 0.446843 41.7345 1.27873C39.2613 1.69328 37.813 4.19778 38.4501 6.62326C39.0873 9.04874 41.5694 10.4717 44.0505 10.1071C47.8511 9.54855 51.7191 9.52689 55.5402 10.0491C60.8642 10.7766 65.9928 12.5457 70.6331 15.2552C75.2735 17.9648 79.3347 21.5619 82.5849 25.841C84.9175 28.9121 86.7997 32.2913 88.1811 35.8758C89.083 38.2158 91.5421 39.6781 93.9676 39.0409Z" fill="currentFill"/>
    </svg>
    <p class="text-xl mt-2">Loading data...</p>
  </div>
`;

const gridOptions = {
  defaultColDef: {
    resizable: true,
    sortable: true,
    filter: true,
  },
  overlayNoRowsTemplate: emptyStateTemplate,
  overlayLoadingTemplate: loadingTemplate
};

// --- Watch for Changes to isLoading ---
watch(
  () => props.isLoading,
  (newVal) => {
    if (!gridApi.value) return;
    gridApi.value.setGridOption('loading', newVal);
  }
);
</script>

<template>
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