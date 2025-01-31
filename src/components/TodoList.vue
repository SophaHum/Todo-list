<template>
  <ul class="space-y-3">
    <li v-for="todo in todos"
        :key="todo.id"
        class="flex items-center gap-3 p-4 backdrop-blur-sm bg-white/40 rounded-xl shadow-lg border border-white/20 cursor-move hover:bg-white/50 transition-all duration-300"
        draggable="true"
        @dragstart="dragStart($event, todo)"
        @dragover.prevent
        @dragenter.prevent
        @drop="drop($event, todo)"
        :class="{ 'opacity-50': isDragging && draggedTodo?.id === todo.id }">
      <input
        type="checkbox"
        :checked="todo.is_completed"
        @change="$emit('toggle-todo', todo.id, todo.is_completed)"
        class="w-5 h-5 accent-purple-500 rounded focus:ring-purple-500"
      >
      <div v-if="editingId === todo.id" class="flex-1 flex gap-2">
        <input
          v-model="editedTask"
          @keyup.enter="saveEdit(todo.id)"
          class="flex-1 px-3 py-1 border rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
          type="text"
        >
        <button
          @click="saveEdit(todo.id)"
          class="px-3 py-1 text-sm text-white bg-green-500 rounded-md hover:bg-green-600"
        >
          Save
        </button>
        <button
          @click="cancelEdit"
          class="px-3 py-1 text-sm text-white bg-gray-500 rounded-md hover:bg-gray-600"
        >
          Cancel
        </button>
      </div>
      <template v-else>
        <span :class="{ 'line-through': todo.is_completed }" class="flex-1 text-gray-800">{{ todo.task }}</span>
        <button
          @click="startEdit(todo)"
          class="px-4 py-2 text-sm text-white bg-gradient-to-r from-blue-500 to-purple-500 rounded-lg hover:opacity-90 transition-opacity"
        >
          Edit
        </button>
        <button
          @click="$emit('delete-todo', todo.id)"
          class="px-4 py-2 text-sm text-white bg-gradient-to-r from-red-500 to-pink-500 rounded-lg hover:opacity-90 transition-opacity"
        >
          Delete
        </button>
      </template>
    </li>
  </ul>
</template>

<script setup lang="ts">
import { ref } from 'vue';

interface Todo {
  id: number;
  task: string;
  is_completed: boolean;
}

defineProps<{
  todos: Todo[]
}>();

const draggedTodo = ref<Todo | null>(null);
const isDragging = ref(false);

const emit = defineEmits<{
  'toggle-todo': [id: number, isCompleted: boolean]
  'edit-todo': [id: number, task: string]
  'delete-todo': [id: number]
  'reorder-todo': [fromId: number, toId: number]
}>();

const editingId = ref<number | null>(null);
const editedTask = ref('');

const startEdit = (todo: Todo) => {
  editingId.value = todo.id;
  editedTask.value = todo.task;
};

const saveEdit = (id: number) => {
  emit('edit-todo', id, editedTask.value);
  editingId.value = null;
};

const cancelEdit = () => {
  editingId.value = null;
  editedTask.value = '';
};

const dragStart = (event: DragEvent, todo: Todo) => {
  if (event.dataTransfer) {
    draggedTodo.value = todo;
    isDragging.value = true;
    event.dataTransfer.effectAllowed = 'move';
  }
};

const drop = (event: DragEvent, targetTodo: Todo) => {
  event.preventDefault();
  if (draggedTodo.value && draggedTodo.value.id !== targetTodo.id) {
    emit('reorder-todo', draggedTodo.value.id, targetTodo.id);
  }
  isDragging.value = false;
  draggedTodo.value = null;
};
</script>
