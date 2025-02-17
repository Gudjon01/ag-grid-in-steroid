<script setup lang="ts">
import { ref, defineProps } from 'vue';

// Define the prop that will be passed from AG Grid (params.value)
const props = defineProps<{ 
  params: {
    text: any;  // or use string/number if you know the type
    // You can also include other properties provided by AG Grid if needed.
  }
}>();

// Reactive flag to control popover visibility.
const showPopover = ref(false);

// Reactive object to hold inline style for the popover.
const popoverStyle = ref({ top: '0px', left: '0px' });

// Reference to the button element.
const buttonRef = ref<HTMLElement | null>(null);

// Toggle the popover and calculate its position.
const handleClick = () => {
  if (buttonRef.value) {
    const rect = buttonRef.value.getBoundingClientRect();
    // Position the popover just below the button.
    popoverStyle.value = {
      top: `${rect.bottom + window.scrollY + 5}px`,
      left: `${rect.left + window.scrollX}px`
    };
  }
  showPopover.value = !showPopover.value;
};
</script>

<template>
  <!-- Button container -->
  <div class="relative inline-block">
    <button 
      ref="buttonRef"
      class="bg-blue-500 text-white px-3 py-1 rounded hover:bg-blue-600 transition"
      @click="handleClick"
    >
      Click Me!
    </button>
  </div>
  
  <!-- Teleport the popover to the document body -->
  <teleport to="body">
    <div 
      v-if="showPopover"
      :style="{
        position: 'absolute',
        top: popoverStyle.top,
        left: popoverStyle.left,
        zIndex: 1000
      }"
      class="w-48 p-3 bg-white border border-gray-300 rounded shadow-lg"
    >
      Popover value: {{ props.params.text }}
    </div>
  </teleport>
</template>

<style scoped>
/* Add any component-specific styles if needed */
</style>
