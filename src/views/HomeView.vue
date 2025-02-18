<script setup lang="ts">
import { ref, onMounted } from 'vue';
import Aggrid from '@/components/Aggrid.vue';
import type { ColDef } from 'ag-grid-community';
import CustomButton from '@/components/CustomButton.vue';

interface MyRow {
  make: string;
  model: string;
  price: number;
  button: string;
}

// Make rows a reactive variable
const rows = ref<MyRow[] | null>(null);

const loadingState = ref<boolean>(false);

const customColumns: ColDef<MyRow>[] = [
  { field: 'make', headerName: 'Make', sortable: true, filter: true },
  { field: 'model', headerName: 'Model', sortable: true, filter: true },
  {
    field: 'price',
    headerName: 'Price',
    sortable: true,
    filter: true,
    cellRenderer: (params: any) => `$${params.value.toLocaleString()}`
  },
  {
    field: 'button',
    headerName: 'Click Me!',
    cellRenderer: CustomButton,
    cellRendererParams: (params: any) => ({
      text: params.value
    })
  }
];

onMounted(() => {
  loadingState.value = true
  setTimeout(() => {
    rows.value = [
      { make: 'Toyota', model: 'Celica', price: 35000, button: 'test' },
      { make: 'Ford', model: 'Mondeo', price: 32000, button: 'test2' },
      { make: 'Porsche', model: 'Boxster', price: 72000, button: 'test3' }
    ];
    loadingState.value = false
  }, 3000);
});
</script>

<template>
  <main>
    <div class="flex items-center justify-center h-screen bg-gray-100">
      <div class="h-96 w-[50rem]">
        <!-- Pass rows as rowData. While rows is null, the grid will display the loading overlay -->
        <Aggrid :columns="customColumns" :row-data="rows" :isLoading="loadingState"/>
      </div>
    </div>
  </main>
</template>
