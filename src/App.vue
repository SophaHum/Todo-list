<template>
  <div class="min-h-screen bg-gradient-to-br from-indigo-500 via-purple-500 to-pink-500 animate-gradient-xy py-8">
    <div class="max-w-2xl mx-auto backdrop-blur-xl bg-white/10 p-8 rounded-2xl shadow-2xl border border-white/20">
      <h1 class="text-4xl font-bold text-center mb-8 text-white drop-shadow-lg">Todo List</h1>
      <div class="bg-white/30 p-6 rounded-xl backdrop-blur-md shadow-lg">
        <TodoForm @add-todo="addTodo" />
        <TodoList
          :todos="todos"
          @toggle-todo="toggleTodo"
          @edit-todo="editTodo"
          @delete-todo="deleteTodo"
          @reorder-todo="reorderTodo"
        />
      </div>
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

//Edit a todo
const editTodo = async (id: number, task: string) => {
  try {
    const { error } = await supabase.from('todos').update({ task }).eq('id', id);
    if (error) throw error;
    await fetchTodos();
  } catch (error) {
    console.error('Error editing todo:', error);
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

// Reorder todos
const reorderTodo = async (fromId: number, toId: number) => {
  try {
    // Find the todos
    const fromTodo = todos.value.find(t => t.id === fromId);
    const toTodo = todos.value.find(t => t.id === toId);

    if (!fromTodo || !toTodo) return;

    // Swap their positions using created_at
    const { error } = await supabase
      .from('todos')
      .update({ created_at: toTodo.created_at })
      .eq('id', fromId);

    if (error) throw error;

    await supabase
      .from('todos')
      .update({ created_at: fromTodo.created_at })
      .eq('id', toId);

    await fetchTodos();
  } catch (error) {
    console.error('Error reordering todos:', error);
  }
};

onMounted(() => {
  fetchTodos();
});
</script>
