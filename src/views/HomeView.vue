<script setup lang="ts">
import { ref, onMounted } from 'vue';
import Aggrid from '@/components/Aggrid.vue';
import type { ColDef } from 'ag-grid-community';
import CustomButton from '@/components/CustomButton.vue';

interface Columns {
  make: string;
  model: string;
  price: number;
  button: string;
}

// A full data set for simulation purposes.
const allData: Columns[] = [
  { make: 'Toyota', model: 'Celica', price: 35000, button: 'test' },
  { make: 'Ford', model: 'Mondeo', price: 32000, button: 'test2' },
  { make: 'Porsche', model: 'Boxster', price: 72000, button: 'test3' },
  { make: 'Honda', model: 'Civic', price: 21000, button: 'test4' },
  { make: 'Chevrolet', model: 'Impala', price: 41000, button: 'test5' },
  { make: 'BMW', model: 'M3', price: 65000, button: 'test6' },
  { make: 'Toyota', model: 'Celica', price: 35000, button: 'test' },
  { make: 'Ford', model: 'Mondeo', price: 32000, button: 'test2' },
  { make: 'Porsche', model: 'Boxster', price: 72000, button: 'test3' },
  { make: 'Honda', model: 'Civic', price: 21000, button: 'test4' },
  { make: 'Chevrolet', model: 'Impala', price: 41000, button: 'test5' },
  { make: 'BMW', model: 'M3', price: 65000, button: 'test6' },
  { make: 'Toyota', model: 'Celica', price: 35000, button: 'test' },
  { make: 'Ford', model: 'Mondeo', price: 32000, button: 'test2' },
  { make: 'Porsche', model: 'Boxster', price: 72000, button: 'test3' },
  { make: 'Honda', model: 'Civic', price: 21000, button: 'test4' },
  { make: 'Chevrolet', model: 'Impala', price: 41000, button: 'test5' },
  { make: 'BMW', model: 'M3', price: 65000, button: 'test6' },
  { make: 'Toyota', model: 'Celica', price: 35000, button: 'test' },
  { make: 'Ford', model: 'Mondeo', price: 32000, button: 'test2' },
  { make: 'Porsche', model: 'Boxster', price: 72000, button: 'test3' },
  { make: 'Honda', model: 'Civic', price: 21000, button: 'test4' },
  { make: 'Chevrolet', model: 'Impala', price: 41000, button: 'test5' },
  { make: 'BMW', model: 'M3', price: 65000, button: 'test6' },
  { make: 'Toyota', model: 'Celica', price: 35000, button: 'test' },
  { make: 'Ford', model: 'Mondeo', price: 32000, button: 'test2' },
  { make: 'Porsche', model: 'Boxster', price: 72000, button: 'test3' },
  { make: 'Honda', model: 'Civic', price: 21000, button: 'test4' },
  { make: 'Chevrolet', model: 'Impala', price: 41000, button: 'test5' },
  { make: 'BMW', model: 'M3', price: 65000, button: 'test6' },
  { make: 'Toyota', model: 'Celica', price: 35000, button: 'test' },
  { make: 'Ford', model: 'Mondeo', price: 32000, button: 'test2' },
  { make: 'Porsche', model: 'Boxster', price: 72000, button: 'test3' },
  { make: 'Honda', model: 'Civic', price: 21000, button: 'test4' },
  { make: 'Chevrolet', model: 'Impala', price: 41000, button: 'test5' },
  { make: 'BMW', model: 'M3', price: 65000, button: 'test6' },
  { make: 'Toyota', model: 'Celica', price: 35000, button: 'test' },
  { make: 'Ford', model: 'Mondeo', price: 32000, button: 'test2' },
  { make: 'Porsche', model: 'Boxster', price: 72000, button: 'test3' },
  { make: 'Honda', model: 'Civic', price: 21000, button: 'test4' },
  { make: 'Chevrolet', model: 'Impala', price: 41000, button: 'test5' },
  { make: 'BMW', model: 'M3', price: 65000, button: 'test6' },
  { make: 'Toyota', model: 'Celica', price: 35000, button: 'test' },
  { make: 'Ford', model: 'Mondeo', price: 32000, button: 'test2' },
  { make: 'Porsche', model: 'Boxster', price: 72000, button: 'test3' },
  { make: 'Honda', model: 'Civic', price: 21000, button: 'test4' },
  { make: 'Chevrolet', model: 'Impala', price: 41000, button: 'test5' },
  { make: 'BMW', model: 'M3', price: 65000, button: 'test6' },
  { make: 'Toyota', model: 'Celica', price: 35000, button: 'test' },
  { make: 'Ford', model: 'Mondeo', price: 32000, button: 'test2' },
  { make: 'Porsche', model: 'Boxster', price: 72000, button: 'test3' },
  { make: 'Honda', model: 'Civic', price: 21000, button: 'test4' },
  { make: 'Chevrolet', model: 'Impala', price: 41000, button: 'test5' },
  { make: 'BMW', model: 'M3', price: 65000, button: 'test6' },
  { make: 'Toyota', model: 'Celica', price: 35000, button: 'test' },
  { make: 'Ford', model: 'Mondeo', price: 32000, button: 'test2' },
  { make: 'Porsche', model: 'Boxster', price: 72000, button: 'test3' },
  { make: 'Honda', model: 'Civic', price: 21000, button: 'test4' },
  { make: 'Chevrolet', model: 'Impala', price: 41000, button: 'test5' },
  { make: 'BMW', model: 'M3', price: 65000, button: 'test6' },
  { make: 'Toyota', model: 'Celica', price: 35000, button: 'test' },
  { make: 'Ford', model: 'Mondeo', price: 32000, button: 'test2' },
  { make: 'Porsche', model: 'Boxster', price: 72000, button: 'test3' },
  { make: 'Honda', model: 'Civic', price: 21000, button: 'test4' },
  { make: 'Chevrolet', model: 'Impala', price: 41000, button: 'test5' },
  { make: 'BMW', model: 'M3', price: 65000, button: 'test6' },
  // ...add as many rows as you like for simulation
];

// Reactive variable for current page data.
const rows = ref<Columns[] | null>(null);

// A reactive boolean to simulate loading state.
const loadingState = ref<boolean>(false);

// Define the columns.
const customColumns: ColDef<Columns>[] = [
  { field: 'make', headerName: 'Make' },
  { field: 'model', headerName: 'Model' },
  {
    field: 'price',
    headerName: 'Price',
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

// The number of rows per page you want (simulate server-side pagination).
const pageSize = 10;

// Simulate a server call that fetches data for a specific page.
const fetchServerData = (page: number, rowCount: number) => {
  loadingState.value = true;
  console.log(`Simulating server call for page ${page} with ${rowCount} rows`);
  setTimeout(() => {
    // Calculate start and end index.
    const start = page * rowCount;
    const end = start + rowCount;
    // In a real scenario, the server returns only the data for the requested page.
    rows.value = allData.slice(start, end);
    loadingState.value = false;
  }, 1500); // simulate a 1.5 second network delay
};

// On mount, fetch the first page.
onMounted(() => {
  fetchServerData(0, pageSize);
});
</script>

<template>
  <main>
    <div class="flex items-center justify-center h-screen bg-gray-100">
      <div class="h-96 w-[50rem]">
        <!-- Pass isLoading and is-server-side props.
             The Aggrid component will emit a "pageChange" event when a pagination button is clicked. -->
        <Aggrid 
          :columns="customColumns" 
          :row-data="rows" 
          :isLoading="loadingState" 
          is-server-side
          :row-count="pageSize"
          :total-row-count="allData.length"
          @page-change="(payload) => { 
              console.log('pageChange event:', payload); 
              fetchServerData(payload.page, payload.rowCount);
          }"
        />
      </div>
    </div>
  </main>
</template>
