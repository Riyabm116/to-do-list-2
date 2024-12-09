<script setup>
import { ref, onMounted, computed, watch } from 'vue';

const todos = ref([]);
const name = ref('');
const input_content = ref('');
const input_category = ref(null);

// Load todos from localStorage
onMounted(() => {
  // Parse todos from localStorage and set them into the todos array
  const savedTodos = localStorage.getItem('todos');
  if (savedTodos) {
    todos.value = JSON.parse(savedTodos);
  }
  name.value = localStorage.getItem('name') || '';
});

// Watch for changes in todos and sync them with localStorage
watch(todos, (newTodos) => {
  localStorage.setItem('todos', JSON.stringify(newTodos));
}, { deep: true }); // deep watch to handle nested array changes

// Sort todos in computed property without side effects
const todos_asc = computed(() => {
  return [...todos.value].sort((a, b) => b.createdAt - a.createdAt);
});

// Add a new todo
const addTodo = () => {
  const newTodo = {
    id: Date.now(),  // Unique id for each todo
    content: input_content.value,
    category: input_category.value,
    done: false,
    createdAt: new Date()
  };
  todos.value.push(newTodo);
  input_content.value = '';  // Reset input field
  input_category.value = null;  // Reset category
};

// Edit a todo
const editTodo = (todoId, newContent) => {
  const todo = todos.value.find(t => t.id === todoId);
  if (todo) {
    todo.content = newContent;
  }
};

// Delete a todo
const deleteTodo = (todoId) => {
  todos.value = todos.value.filter(todo => todo.id !== todoId);
};

// Sync the name with localStorage
watch(name, (newVal) => {
  localStorage.setItem('name', newVal);
});
</script>

<template>
  <main class="app">
    <section class="greeting">
      <h2 class="title">
        What's up,
        <input type="text" placeholder="Name here" v-model="name" />
      </h2>
    </section>

    <section class="create-todo">
      <h3>CREATE A TODO</h3>
      <form @submit.prevent="addTodo">
        <h4>What's on your todo list?</h4>
        <input
          type="text"
          placeholder="e.g. make a video"
          v-model="input_content"
        />

        <h4>Pick a category</h4>
        <div class="options">
          <label>
            <input
              type="radio"
              name="category"
              value="work"
              v-model="input_category"
            />
            <span class="bubble work"></span>
            <div>Work</div>
          </label>

          <label>
            <input
              type="radio"
              name="category"
              value="personal"
              v-model="input_category"
            />
            <span class="bubble personal"></span>
            <div>Personal</div>
          </label>
        </div>

        <input type="submit" value="Add todo" />
      </form>
    </section>

    <section class="todo-list">
      <h3>TODO LIST</h3>
      <div class="list">
        <div
          v-for="todo in todos_asc"
          :key="todo.id"
          :class="`todo-item ${todo.done ? 'done' : ''}`"
        >
          <p>{{ todo.content }}</p>
          <button @click="editTodo(todo.id, 'Updated content')">Edit</button>
          <button @click="deleteTodo(todo.id)">Delete</button>
        </div>
      </div>
    </section>
  </main>
</template>
