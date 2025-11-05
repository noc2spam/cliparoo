<template>
  <div v-if="props.show" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
    <div class="bg-white rounded-lg shadow-lg p-6 w-11/12 max-w-md">
      <h2 class="text-xl font-bold mb-4">{{ props.title || 'Confirm Action' }}</h2>
      <p class="mb-6">{{ props.message || 'Are you sure you want to proceed?' }}</p>
      <div class="flex justify-between space-x-4">
        <button @click="emit('close', false)" class="px-4 py-2 bg-[#f2f2f2] rounded hover:bg-[#d2d2d2]">
          Cancel
        </button>
        <button @click="emit('close', true)" class="px-4 py-2 bg-[#3c45bd] text-white rounded hover:bg-[#3c30bd]">
          Confirm
        </button>
      </div>
    </div>
  </div>
</template>
<script lang="ts" setup>
const props = defineProps<{ show: boolean, title?: string, message?: string }>();
const emit = defineEmits<{
  (e: 'close', confirmed: boolean): void;
}>();
</script>