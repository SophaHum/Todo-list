<template>
  <div class="min-h-screen bg-gray-100 py-8">
    <div class="max-w-2xl mx-auto bg-white p-6 rounded-lg shadow-lg">
      <h1 class="text-3xl font-bold text-center mb-8">Todo List</h1>
      <TodoForm @add-todo="addTodo" />
      <TodoList :todos="todos" @toggle-todo="toggleTodo" @delete-todo="deleteTodo" />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { supabase } from './supabase';
import TodoForm from './components/TodoForm.vue';
import TodoList from './components/TodoList.vue';

interface Todo {
  id: number;
  task: string;
  is_completed: boolean;
  created_at: string;
}

const todos = ref<Todo[]>([]);

// Fetch todos from Supabase
const fetchTodos = async () => {
  try {
    const { data, error } = await supabase
      .from('todos')
      .select('*')
      .order('created_at', { ascending: false });

    if (error) throw error;
    todos.value = data || [];
  } catch (error) {
    console.error('Error fetching todos:', error);
  }
};

// Add a new todo
const addTodo = async (task: string) => {
  try {
    const { error } = await supabase.from('todos').insert([{ task }]);
    if (error) throw error;
    await fetchTodos();
  } catch (error) {
    console.error('Error adding todo:', error);
  }
};

// Toggle todo completion
const toggleTodo = async (id: number, isCompleted: boolean) => {
  try {
    const { error } = await supabase
      .from('todos')
      .update({ is_completed: !isCompleted })
      .eq('id', id);
    if (error) throw error;
    await fetchTodos();
  } catch (error) {
    console.error('Error toggling todo:', error);
  }
};

// Delete a todo
const deleteTodo = async (id: number) => {
  try {
    const { error } = await supabase.from('todos').delete().eq('id', id);
    if (error) throw error;
    await fetchTodos();
  } catch (error) {
    console.error('Error deleting todo:', error);
  }
};

onMounted(() => {
  fetchTodos();
});
</script>
