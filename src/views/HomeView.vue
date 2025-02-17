<script setup lang="ts">
import Aggrid from '@/components/Aggrid.vue';
import type { ColDef } from 'ag-grid-community';
import CustomButton from '@/components/CustomButton.vue';

interface MyRow {
  make: string;
  model: string;
  price: number;
  button: string;
}

const rows: MyRow[] = [
  { make: 'Toyota', model: 'Celica', price: 35000, button: 'test' },
  { make: 'Ford', model: 'Mondeo', price: 32000, button: 'test2' },
  { make: 'Porsche', model: 'Boxster', price: 72000, button: 'test3' }
];

// Here we define custom columns. Note that for the "Click Me!" column,
// we use cellRendererSelector to return our custom component and pass the cell's value.
const customColumns: (ColDef<MyRow> & { cellRendererSelector?: any })[] = [
  { field: 'make', headerName: 'Make', sortable: true, filter: true },
  { field: 'model', headerName: 'Model', sortable: true, filter: true },
  {
    field: 'price',
    headerName: 'Price',
    sortable: true,
    filter: true,
    cellRenderer: (params: any) => `$${params.value.toLocaleString()}`
  },
  // VUE AG GRID INTEGRATION CELLRENDERER VE TUREVLERİNDE OTOMATİK OLARAK PROPLARI "PARAMS"
  //  İÇİNE SARIYOR O YÜZDEN PROPLARI YAZARKEN PARAM OBJESİ İÇİNE EKLEMEMİZ LAZIM
  {
    field: 'button',
    headerName: 'Click Me!',
    cellRenderer: CustomButton,
    cellRendererParams: (params: any) => ({
      text: params.value
    })
  }
];
</script>

<template>
  <main>
    <div class="flex items-center justify-center h-screen bg-gray-100">
      <div class="h-96 w-[50rem]">
        <Aggrid :columns="customColumns" :row-data="rows" />
      </div>
    </div>
  </main>
</template>
