<template>
  <ul>
    <li
      v-for="todo in todos"
      :key="todo.id"
      class="flex items-center justify-between p-4 border-b border-gray-200"
    >
      <div class="flex items-center gap-3">
        <input
          type="checkbox"
          :checked="todo.is_completed"
          @change="toggleTodo(todo.id, todo.is_completed)"
          class="w-5 h-5"
        />
        <span :class="{ 'line-through': todo.is_completed }">{{ todo.task }}</span>
      </div>
      <button
        @click="deleteTodo(todo.id)"
        class="text-red-500 hover:text-red-700"
      >
        Delete
      </button>
    </li>
  </ul>
</template>

<script setup lang="ts">
interface Todo {
  id: number;
  task: string;
  is_completed: boolean;
}

defineProps<{
  todos: Todo[];
}>();

const emit = defineEmits(['toggle-todo', 'delete-todo']);

const toggleTodo = (id: number, isCompleted: boolean) => {
  emit('toggle-todo', id, isCompleted);
};

const deleteTodo = (id: number) => {
  emit('delete-todo', id);
};
</script>
