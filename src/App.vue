<script setup lang="ts">
import { ref, onMounted, computed, watch } from "vue";

// types

type Category = "personal" | "business";

interface TodoItem {
  createdAt: number;
  category: Category;
  title: string;
  done: boolean;
}

// refs

const todos = ref<TodoItem[]>([]);
const name = ref("");

const inputValue = ref("");
const inputCategory = ref(null);

const todosAsc = computed(() =>
  todos.value.sort((a, b) => b.createdAt - a.createdAt)
);

// code

const addTodo = () => {
  if (inputValue.value.trim() === "" || inputCategory.value === null) {
    return;
  }

  const todo: TodoItem = {
    createdAt: new Date().getTime(),
    title: inputValue.value,
    category: inputCategory.value,
    done: false,
  };

  inputValue.value = "";
  inputCategory.value = null;

  todos.value.push(todo);
};

const deleteTodo = (td: TodoItem) => {
  todos.value = todos.value.filter((t) => t !== td);
};

// watches

watch(name, (newValue) => {
  localStorage.setItem("name", newValue);
});

watch(
  todos,
  (newValue) => localStorage.setItem("todos", JSON.stringify(newValue)),
  { deep: true }
);

// mounted

onMounted(() => {
  name.value = localStorage.getItem("name") || "";
  todos.value = JSON.parse(localStorage.getItem("todos") || "[]") || [];
});
</script>

<template>
  <main class="app">
    <section class="greeting">
      <h2 class="title">
        Eae <input type="text" v-model="name" placeholder="Nome" />
      </h2>
    </section>

    <section class="create-todo">
      <h3>CRIAR UMA TAREFA</h3>
      <form @submit.prevent="addTodo">
        <h4>O que você vai fazer?</h4>
        <input
          type="text"
          placeholder="Ex.: ir à academia"
          v-model="inputValue"
        />

        <h4>Selecione um categoria</h4>
        <div class="options">
          <label>
            <input
              type="radio"
              name="category"
              value="business"
              v-model="inputCategory"
            />
            <span class="bubble business"></span>
            <div>Trabalho</div>
          </label>
          <label>
            <input
              type="radio"
              name="category"
              value="personal"
              v-model="inputCategory"
            />
            <span class="bubble personal"></span>
            <div>Pessoal</div>
          </label>
        </div>

        <input type="submit" value="Criar" />
      </form>
    </section>

    <section class="todos-list">
      <h3>TAREFAS</h3>
      <div class="list">
        <div
          v-for="todoItem in todosAsc"
          :class="`todo-item ${todoItem.done ? 'done' : ''}`"
        >
          <label>
            <input type="checkbox" v-model="todoItem.done" />
            <span :class="`bubble ${todoItem.category}`"></span>
          </label>

          <div class="todo-content">
            <input type="text" v-model="todoItem.title" />
          </div>

          <div class="actions">
            <button class="delete" @click="deleteTodo(todoItem)">Delete</button>
          </div>
        </div>
      </div>
    </section>
  </main>
</template>

<style scoped>
.greeting .title {
  display: flex;
}

.greeting .title input {
  margin-left: 0.5rem;
  flex: 1 1 0%;
  min-width: 0;
}

.greeting .title,
.greeting .title input {
  color: var(--dark);
  font-size: 1.5rem;
  font-weight: 700;
}

.create-todo input[type="text"] {
  display: block;
  width: 100%;
  font-size: 1.125rem;
  padding: 1rem 1.5rem;
  color: var(--dark);
  background-color: #fff;
  border-radius: 0.5rem;
  box-shadow: var(--shadow);
  margin-bottom: 1.5rem;
}

.create-todo .options {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  grid-gap: 1rem;
  margin-bottom: 1.5rem;
}

.create-todo .options label {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 1.5rem;
  background-color: #fff;
  border-radius: 0.5rem;
  box-shadow: var(--shadow);
  cursor: pointer;
}

input[type="radio"],
input[type="checkbox"] {
  display: none;
}

.bubble {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 20px;
  height: 20px;
  border-radius: 50%;
  border: 2px solid var(--business);
  box-shadow: var(--business-glow);
}

.bubble.personal {
  border-color: var(--personal);
  box-shadow: var(--personal-glow);
}

.bubble::after {
  content: "";
  display: block;
  opacity: 0;
  width: 0px;
  height: 0px;
  background-color: var(--business);
  box-shadow: var(--business-glow);
  border-radius: 50%;
  transition: 0.2s ease-in-out;
}

.bubble.personal::after {
  background-color: var(--personal);
  box-shadow: var(--personal-glow);
}

input:checked ~ .bubble::after {
  width: 10px;
  height: 10px;
  opacity: 1;
}

.create-todo .options label div {
  color: var(--dark);
  margin-top: 1rem;
}

.create-todo input[type="submit"] {
  display: block;
  width: 100%;
  font-size: 1.125rem;
  padding: 1rem 1.5rem;
  color: #fff;
  background-color: var(--primary);
  border-radius: 0.5rem;
  box-shadow: var(--primary-glow);
  cursor: pointer;
  transition: 0.2s ease-in-out;
}

.create-todo input[type="submit"]:hover {
  opacity: 0.75;
}

.todo-list .list {
  margin: 1rem 0;
}

.todo-item {
  display: flex;
  align-items: center;
  background-color: #fff;
  padding: 1rem;
  border-radius: 0.5rem;
  box-shadow: var(--shadow);
  margin-bottom: 1rem;
}

.todo-item label {
  display: block;
  margin-right: 1rem;
  cursor: pointer;
}

.todo-item .todo-content {
  flex: 1 1 0%;
}

.todo-item .todo-content input {
  color: var(--dark);
  font-size: 1.125rem;
}

.todo-item .actions {
  display: flex;
  align-items: center;
}

.todo-item .actions button {
  display: block;
  padding: 0.5rem;
  border-radius: 0.25rem;
  color: #fff;
  cursor: pointer;
  transition: 0.2s ease-in-out;
}

.todo-item .actions button:hover {
  opacity: 0.75;
}

.todo-item .actions .edit {
  margin-right: 0.5rem;
  background-color: var(--primary);
}

.todo-item .actions .delete {
  background-color: var(--danger);
}

.todo-item.done .todo-content input {
  text-decoration: line-through;
  color: var(--grey);
}
</style>
