<template>
  <form @submit.prevent="handleSubmit" class="mb-6">
    <div class="flex gap-2">
      <input
        v-model="task"
        type="text"
        placeholder="Add a new task"
        class="flex-1 p-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
      />
      <button
        type="submit"
        class="px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600"
      >
        Add
      </button>
    </div>
  </form>
</template>

<script setup lang="ts">
import { ref } from 'vue';

const task = ref('');

const emit = defineEmits(['add-todo']);

const handleSubmit = () => {
  if (task.value.trim()) {
    emit('add-todo', task.value.trim());
    task.value = '';
  }
};
</script>
