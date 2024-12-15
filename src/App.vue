<template>
  <main class="app">
    <!-- Greeting Section -->
    <section class="greeting">
      <h2 class="title">
        What's up,
        <input type="text" placeholder="Name here" v-model="name" />
      </h2>
    </section>

    <!-- Search and Input Fields Side by Side -->
    <section class="search-input">
      <div class="search-container">
        <label for="search">Search Todos:</label>
        <input
          type="text"
          id="search"
          v-model="searchQuery"
          placeholder="Search todos"
        />
      </div>

      <div class="input-container">
        <label for="input">Add New Todo:</label>
        <input
          type="text"
          id="input"
          v-model="input_content"
          placeholder="Type something here"
        />
        <button @click="addTodo" class="add-button">Add</button>
      </div>
    </section>

    <!-- Recurrence Section -->
    <section class="recurrence">
      <h3>Set Recurrence</h3>
      <div>
        <label>
          <input
            type="radio"
            name="recurrence"
            value="none"
            v-model="recurrence"
          />
          None
        </label>
        <label>
          <input
            type="radio"
            name="recurrence"
            value="weekly"
            v-model="recurrence"
          />
          Weekly
        </label>
        <label>
          <input
            type="radio"
            name="recurrence"
            value="monthly"
            v-model="recurrence"
          />
          Monthly
        </label>
      </div>
    </section>

    <!-- Create Todo Section -->
    <section class="create-todo">
      <h3>SET CATEGORY AND PRIORITY</h3>
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

      <h4>Set Priority</h4>
      <div class="options">
        <label>
          <input
            type="radio"
            name="priority"
            value="high"
            v-model="input_priority"
          />
          High
        </label>
        <label>
          <input
            type="radio"
            name="priority"
            value="medium"
            v-model="input_priority"
          />
          Medium
        </label>
        <label>
          <input
            type="radio"
            name="priority"
            value="low"
            v-model="input_priority"
          />
          Low
        </label>
      </div>
    </section>

    <!-- Todo List Section -->
    <section class="todo-list">
      <h3>TODO LIST</h3>
      <VueDraggableNext
        v-bind:model-value="filteredTodos"
        @update:model-value="updateTodos"
        :item-key="'id'"
        class="list"
      >
        <template #item="{ element }">
          <div
            :key="element.id"
            :class="`todo-item ${element.done ? 'done' : ''}`"
            :style="{ backgroundColor: getPriorityColor(element.priority) }"
          >
            <p>{{ element.content }}</p>

            <!-- Display Recurrence and Next Due Date -->
            <div v-if="element.recurrence !== 'none'" class="recurrence-info">
              <span v-if="element.recurrence">Recurs: {{ element.recurrence }}</span>
              <span v-if="element.nextDueDate">Next Due: {{ element.nextDueDate }}</span>
            </div>

            <div class="button-container">
              <button class="edit-button" @click="editTodo(element.id)">
                Edit
              </button>
              <button class="delete-button" @click="deleteTodo(element.id)">
                Delete
              </button>
            </div>
          </div>
        </template>
      </VueDraggableNext>
    </section>
  </main>
</template>

<script setup>
import { ref, onMounted, computed, watch } from 'vue';
import VueDraggableNext from 'vuedraggable';

const todos = ref([]);
const name = ref('');
const input_content = ref('');
const input_category = ref(null);
const input_priority = ref('medium'); // Priority field
const searchQuery = ref('');
const recurrence = ref('none'); // For recurring tasks

// Load todos from localStorage
onMounted(() => {
  const savedTodos = localStorage.getItem('todos');
  if (savedTodos) {
    todos.value = JSON.parse(savedTodos);
  }
  name.value = localStorage.getItem('name') || '';
});

// Watch for changes in todos and sync them with localStorage
watch(
  todos,
  (newTodos) => {
    localStorage.setItem('todos', JSON.stringify(newTodos));
  },
  { deep: true }
);

// Add a new todo
const addTodo = () => {
  if (!input_content.value.trim()) return; // Prevent empty todos
  const newTodo = {
    id: Date.now(),
    content: input_content.value,
    category: input_category.value,
    priority: input_priority.value, // Add priority field
    done: false,
    createdAt: new Date(),
    recurrence: recurrence.value, // Add recurrence type
    nextDueDate: null, // Set initially to null
  };
  updateNextDueDate(newTodo);
  todos.value.push(newTodo);
  input_content.value = ''; // Reset input
  input_category.value = null; // Reset category
  input_priority.value = 'medium'; // Reset priority
  recurrence.value = 'none'; // Reset recurrence
};

// Update next due date for recurring tasks
const updateNextDueDate = (todo) => {
  if (todo.recurrence === 'weekly') {
    const nextMonday = new Date();
    nextMonday.setDate(nextMonday.getDate() + (1 + 7 - nextMonday.getDay()) % 7);
    todo.nextDueDate = nextMonday.toLocaleDateString();
  } else if (todo.recurrence === 'monthly') {
    const nextMonth = new Date();
    nextMonth.setMonth(nextMonth.getMonth() + 1);
    nextMonth.setDate(1); // Set to first day of next month
    todo.nextDueDate = nextMonth.toLocaleDateString();
  } else {
    todo.nextDueDate = null; // No recurrence
  }
};

// Edit a todo
const editTodo = (todoId) => {
  const todo = todos.value.find((t) => t.id === todoId);
  if (todo) {
    const newContent = prompt('Edit todo content:', todo.content);
    if (newContent !== null) {
      todo.content = newContent;
      updateNextDueDate(todo); // Recalculate next due date
    }
  }
};

// Delete a todo
const deleteTodo = (todoId) => {
  todos.value = todos.value.filter((todo) => todo.id !== todoId);
};

// Sync name with localStorage
watch(name, (newVal) => {
  localStorage.setItem('name', newVal);
});

// Update todos after drag-and-drop
const updateTodos = (newTodos) => {
  todos.value = newTodos;
};

// Filter todos based on search query
const filteredTodos = computed(() => {
  return todos.value.filter((todo) =>
    todo.content.toLowerCase().includes(searchQuery.value.toLowerCase())
  );
});

// Assign priority colors
const getPriorityColor = (priority) => {
  switch (priority) {
    case 'high':
      return '#FF4D4D'; // High priority
    case 'medium':
      return '#FFD700'; // Medium priority
    case 'low':
      return '#90EE90'; // Low priority
    default:
      return '#ffffff'; // Default
  }
};
</script>

<style scoped>
/* General styles */
.app {
  font-family: Arial, sans-serif;
  max-width: 800px;
  margin: 0 auto;
}

section {
  margin-bottom: 20px;
}

/* Search and input fields side by side */
.search-input {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.search-container,
.input-container {
  width: 48%;
}

.search-container input,
.input-container input {
  padding: 8px;
  width: 100%;
  margin: 5px 0;
}

button {
  margin-left: 10px;
  border: none;
  padding: 12px 20px; /* Larger button */
  border-radius: 5px;
  cursor: pointer;
  font-size: 16px;
}

button:hover {
  opacity: 0.9;
}

.add-button {
  background-color: #4caf50;
  color: white;
}

.edit-button {
  background-color: #4caf50;
  color: white;
}

.delete-button {
  background-color: #f44336;
  color: white;
}

.todo-item {
  margin-bottom: 10px;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.button-container {
  display: flex;
  gap: 10px;
}

.done {
  text-decoration: line-through;
}

.list {
  margin-top: 20px;
}

.recurrence-info {
  font-size: 12px;
  color: #666;
  margin-top: 5px;
}

.recurrence-info span {
  display: block;
}

h3 {
  margin-bottom: 10px;
}
</style>
