<script setup lang="ts">
import { ref, shallowRef, defineProps, defineEmits, withDefaults, watch, computed } from 'vue';
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

// Extend the props interface to include totalRowCount
interface AgGridProps<T> {
  columns: ColDef<T>[];
  rowData: T[] | null;
  isLoading: boolean;
  horizontalAutoFit?: boolean;
  verticalAutoFit?: boolean;
  rowCount?: number;         // default rows per page
  isServerSide?: boolean;    // default false
  totalRowCount?: number;    // total rows in the dataset (for server-side)
}

// Provide default values for isServerSide and rowCount.
const props = withDefaults(defineProps<AgGridProps<any>>(), {
  isServerSide: false,
  rowCount: 10
});

// Define our custom event. We emit a 'pageChange' event with page number and rowCount.
const emit = defineEmits<{
  (event: 'pageChange', payload: { page: number, rowCount: number }): void;
}>();

// --- Dynamic Row Height Logic ---
let minRowHeight = 25;
let currentRowHeight: number;

// Shallow ref to hold the grid API.
const gridApi = shallowRef<GridApi | null>(null);

// We'll also store a computed row count for pagination when verticalAutoFit is true.
const computedRowCount = ref(props.rowCount);

// Function to update row heights.
const updateRowHeight = (params: { api: GridApi }) => {
  const bodyViewport = document.querySelector(".ag-body-viewport");
  if (!bodyViewport) return;
  const gridHeight = bodyViewport.clientHeight;
  // Instead of using rendered row count, we rely on minRowHeight for a consistent value.
  // (Assuming minRowHeight comes from the theme.)
  currentRowHeight = minRowHeight;
  params.api.resetRowHeights();
};

// Provide row height via callback.
const getRowHeight = ref((params: RowHeightParams) => currentRowHeight);

// --- Grid Events ---
function onFirstDataRendered(params: FirstDataRenderedEvent) {
  if (props.verticalAutoFit) updateRowHeight(params);
  if (props.horizontalAutoFit) params.api.sizeColumnsToFit();
}

function onGridSizeChanged(params: GridSizeChangedEvent) {
  const bodyViewport = document.querySelector(".ag-body-viewport");
  if (!bodyViewport) return;
  const gridWidth = bodyViewport.clientWidth;
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
  if (props.horizontalAutoFit) {
    window.setTimeout(() => params.api.sizeColumnsToFit(), 10);
  }
  if (props.verticalAutoFit) {
    // Calculate new row count using the grid viewport height and minRowHeight.
    const gridHeight = bodyViewport.clientHeight;
    let newRowCount = Math.floor(gridHeight / minRowHeight);
    if (newRowCount < 1) newRowCount = 1;
    computedRowCount.value = newRowCount;
    if (gridApi.value && !props.isServerSide) {
      gridApi.value.setGridOption('paginationPageSize', newRowCount);
    }
  }
};

const updatePaginationState = () => {
  if (gridApi.value && !props.isServerSide) {
    currentPage.value = gridApi.value.paginationGetCurrentPage();
    totalPages.value = gridApi.value.paginationGetTotalPages();
  }
};

const onGridReady = (params: GridReadyEvent) => {
  gridApi.value = params.api;
  // Get minRowHeight from the theme.
  minRowHeight = params.api.getSizesForCurrentTheme().rowHeight;
  currentRowHeight = minRowHeight;

  // For client-side pagination, enable pagination and set page size.
  if (!props.isServerSide) {
    gridApi.value.setGridOption('pagination', true);
    gridApi.value.setGridOption('paginationPageSize', props.rowCount);
    gridApi.value.setGridOption('suppressPaginationPanel', true);
    gridApi.value.addEventListener('paginationChanged', updatePaginationState);
    updatePaginationState();
  }

  // Set loading state.
  gridApi.value.setGridOption('loading', props.isLoading);
};

// --- Reactive Pagination State (for client-side only) ---
const currentPage = ref(0);
const totalPages = ref(0);

// --- Computed Total Pages for Server-Side ---
const serverTotalPages = computed(() => {
  if (props.isServerSide && props.totalRowCount != null) {
    return Math.ceil(props.totalRowCount / computedRowCount.value);
  }
  return 0;
});

// A unified computed property for display total pages.
const displayTotalPages = computed(() => {
  return props.isServerSide ? serverTotalPages.value : totalPages.value;
});

// --- Custom Pagination Handlers ---
const firstPage = () => {
  if (props.isServerSide) {
    currentPage.value = 0;
    emit('pageChange', { page: 0, rowCount: computedRowCount.value });
  } else if (gridApi.value) {
    gridApi.value.paginationGoToPage(0);
    updatePaginationState();
    emit('pageChange', { page: 0, rowCount: computedRowCount.value });
  }
};

const previousPage = () => {
  if (currentPage.value > 0) {
    const newPage = currentPage.value - 1;
    if (props.isServerSide) {
      currentPage.value = newPage;
      emit('pageChange', { page: newPage, rowCount: computedRowCount.value });
    } else if (gridApi.value) {
      gridApi.value.paginationGoToPage(newPage);
      updatePaginationState();
      emit('pageChange', { page: newPage, rowCount: computedRowCount.value });
    }
  }
};

const nextPage = () => {
  if (props.isServerSide) {
    // Only allow next if current page is less than serverTotalPages - 1.
    if (currentPage.value < serverTotalPages.value - 1) {
      const newPage = currentPage.value + 1;
      currentPage.value = newPage;
      emit('pageChange', { page: newPage, rowCount: computedRowCount.value });
    }
  } else if (gridApi.value) {
    const curr = gridApi.value.paginationGetCurrentPage();
    const total = gridApi.value.paginationGetTotalPages();
    if (curr < total - 1) {
      const newPage = curr + 1;
      gridApi.value.paginationGoToPage(newPage);
      updatePaginationState();
      emit('pageChange', { page: newPage, rowCount: computedRowCount.value });
    }
  }
};

const lastPage = () => {
  if (props.isServerSide) {
    // For server-side, jump to the last page (if available).
    const last = serverTotalPages.value - 1;
    if (last >= 0) {
      currentPage.value = last;
      emit('pageChange', { page: last, rowCount: computedRowCount.value });
    }
  } else if (gridApi.value) {
    const total = gridApi.value.paginationGetTotalPages();
    const last = total - 1;
    gridApi.value.paginationGoToPage(last);
    updatePaginationState();
    emit('pageChange', { page: last, rowCount: computedRowCount.value });
  }
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
  overlayLoadingTemplate: loadingTemplate,
  pagination: true,
  paginationPageSize: props.rowCount, // initial value; will be updated if verticalAutoFit is true
  paginationPageSizeSelector: false,    // disable built-in page size selector
  suppressPaginationPanel: true,
};

watch(
  () => props.isLoading,
  (newVal) => {
    if (!gridApi.value) return;
    gridApi.value.setGridOption('loading', newVal);
    if (!newVal) {
      // Force hide the overlay if loading is turned off.
      gridApi.value.hideOverlay();
    }
  }
);
</script>

<template>
  <div class="size-full">
    <ag-grid-vue style="width: 100%; height: 100%;" :theme="themeQuartz" :columnDefs="props.columns"
      :rowData="props.rowData" :gridOptions="gridOptions" :getRowHeight="getRowHeight" @grid-ready="onGridReady"
      @first-data-rendered="onFirstDataRendered" @grid-size-changed="onGridSizeChanged" />

    <!-- Custom Pagination Controls -->
    <div class="cursor-pointer mt-4 flex items-center justify-center space-x-4">
      <button @click="firstPage" :disabled="currentPage === 0"
        class="px-3 py-1 bg-gray-300 rounded disabled:opacity-50">
        First
      </button>
      <button @click="previousPage" :disabled="currentPage === 0"
        class="px-3 py-1 bg-gray-300 rounded disabled:opacity-50">
        Previous
      </button>
      <span v-if="!props.isServerSide">
        Page {{ currentPage + 1 }} of {{ totalPages }}
      </span>
      <span v-else>
        Page {{ currentPage + 1 }} of {{ serverTotalPages }}
      </span>
      <button @click="nextPage"
        :disabled="props.isServerSide ? currentPage >= serverTotalPages - 1 : currentPage >= totalPages - 1"
        class="px-3 py-1 bg-gray-300 rounded disabled:opacity-50">
        Next
      </button>
      <button @click="lastPage" :disabled="props.isServerSide ? currentPage >= serverTotalPages - 1 : currentPage >= totalPages - 1"
        class="px-3 py-1 bg-gray-300 rounded disabled:opacity-50">
        Last
      </button>
    </div>
  </div>
</template>