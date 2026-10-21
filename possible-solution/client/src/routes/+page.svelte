<script lang="ts">
  import { browser } from '$app/environment';
  import { onMount } from 'svelte';

  type Todo = {
    id: number;
    text: string;
    completed: boolean;
  };

  const STORAGE_KEY = 'todo-list-items';

  let newTodo = '';
  let todos: Todo[] = [];

  function saveTodos() {
    if (!browser) return;
    localStorage.setItem(STORAGE_KEY, JSON.stringify(todos));
  }

  function loadTodos() {
    if (!browser) return;

    const saved = localStorage.getItem(STORAGE_KEY);
    if (!saved) {
      todos = [
        { id: 1, text: 'Welcome to your todo list', completed: false },
        { id: 2, text: 'Click the checkbox to complete tasks', completed: true }
      ];
      saveTodos();
      return;
    }

    try {
      const parsed = JSON.parse(saved) as Todo[];
      todos = Array.isArray(parsed) ? parsed : [];
    } catch {
      todos = [];
    }
  }

  onMount(() => {
    loadTodos();
  });

  $: if (browser) {
    saveTodos();
  }

  function addTodo() {
    const text = newTodo.trim();
    if (!text) return;

    todos = [
      ...todos,
      {
        id: Date.now(),
        text,
        completed: false
      }
    ];

    newTodo = '';
  }

  function toggleTodo(id: number) {
    todos = todos.map((todo) =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    );
  }

  function deleteTodo(id: number) {
    todos = todos.filter((todo) => todo.id !== id);
  }

  function clearCompleted() {
    todos = todos.filter((todo) => !todo.completed);
  }

  $: remainingCount = todos.filter((todo) => !todo.completed).length;
  $: completedCount = todos.filter((todo) => todo.completed).length;
</script>

<svelte:head>
  <title>Todo List</title>
</svelte:head>

<div class="page">
  <div class="todo-app">
    <header>
      <p class="eyebrow">Tasks</p>
      <h1>My To-Do List</h1>
    </header>

    <form class="add-todo" on:submit|preventDefault={addTodo}>
      <input
        bind:value={newTodo}
        type="text"
        placeholder="Add a task..."
        aria-label="Add a new task"
      />
      <button type="submit" disabled={!newTodo.trim()}>Add</button>
    </form>

    <div class="summary">
      <span>{remainingCount} remaining</span>
      <span>{completedCount} completed</span>
      <button type="button" class="text-button" on:click={clearCompleted} disabled={completedCount === 0}>
        Clear completed
      </button>
    </div>

    {#if todos.length === 0}
      <div class="empty-state">
        <p>No tasks yet.</p>
        <p>Add one above to get started.</p>
      </div>
    {:else}
      <ul class="todo-list">
        {#each todos as todo (todo.id)}
          <li class:completed={todo.completed}>
            <label class="todo-item">
              <input type="checkbox" checked={todo.completed} on:change={() => toggleTodo(todo.id)} />
              <span>{todo.text}</span>
            </label>
            <button type="button" class="delete-button" on:click={() => deleteTodo(todo.id)} aria-label="Delete task">
              ×
            </button>
          </li>
        {/each}
      </ul>
    {/if}
  </div>
</div>

<style>
  :global(body) {
    margin: 0;
    font-family: 'Segoe UI', sans-serif;
    background: linear-gradient(135deg, #f3f4f6 0%, #e0f2fe 100%);
    color: #0f172a;
  }

  * {
    box-sizing: border-box;
  }

  .page {
    min-height: 100vh;
    display: grid;
    place-items: center;
    padding: 24px;
  }

  .todo-app {
    width: min(100%, 560px);
    background: rgba(255, 255, 255, 0.9);
    border: 1px solid rgba(148, 163, 184, 0.3);
    border-radius: 20px;
    box-shadow: 0 20px 50px rgba(15, 23, 42, 0.08);
    padding: 24px;
  }

  header {
    margin-bottom: 20px;
  }

  .eyebrow {
    margin: 0 0 8px;
    text-transform: uppercase;
    letter-spacing: 0.12em;
    font-size: 0.72rem;
    color: #3b82f6;
    font-weight: 700;
  }

  h1 {
    margin: 0;
    font-size: clamp(2rem, 4vw, 2.6rem);
  }

  .add-todo {
    display: flex;
    gap: 12px;
    margin-bottom: 18px;
  }

  .add-todo input {
    flex: 1;
    border: 1px solid #cbd5e1;
    border-radius: 12px;
    padding: 12px 14px;
    font-size: 1rem;
    outline: none;
    transition: border-color 0.2s ease, box-shadow 0.2s ease;
  }

  .add-todo input:focus {
    border-color: #3b82f6;
    box-shadow: 0 0 0 3px rgba(59, 130, 246, 0.15);
  }

  button {
    border: none;
    border-radius: 12px;
    font-weight: 600;
    cursor: pointer;
    transition: transform 0.15s ease, opacity 0.15s ease;
  }

  button:hover:not(:disabled) {
    transform: translateY(-1px);
  }

  .add-todo button {
    background: #2563eb;
    color: white;
    padding: 0 18px;
    min-width: 80px;
  }

  .add-todo button:disabled {
    opacity: 0.55;
    cursor: not-allowed;
  }

  .summary {
    display: flex;
    align-items: center;
    justify-content: space-between;
    flex-wrap: wrap;
    gap: 10px;
    color: #475569;
    font-size: 0.95rem;
    margin-bottom: 18px;
  }

  .text-button {
    background: transparent;
    color: #2563eb;
    padding: 0;
    font-size: 0.9rem;
  }

  .text-button:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }

  .todo-list {
    list-style: none;
    padding: 0;
    margin: 0;
    display: flex;
    flex-direction: column;
    gap: 10px;
  }

  .todo-list li {
    display: flex;
    align-items: center;
    justify-content: space-between;
    gap: 12px;
    border: 1px solid #e2e8f0;
    border-radius: 12px;
    padding: 10px 12px;
    background: #f8fafc;
  }

  .todo-item {
    display: flex;
    align-items: center;
    gap: 12px;
    flex: 1;
    min-width: 0;
  }

  .todo-item input {
    width: 18px;
    height: 18px;
    accent-color: #22c55e;
    cursor: pointer;
  }

  .todo-item span {
    word-break: break-word;
  }

  .todo-list li.completed .todo-item span {
    text-decoration: line-through;
    color: #64748b;
  }

  .delete-button {
    width: 30px;
    height: 30px;
    border-radius: 8px;
    background: #fee2e2;
    color: #b91c1c;
    font-size: 1.3rem;
    line-height: 1;
  }

  .empty-state {
    text-align: center;
    padding: 28px 16px 10px;
    color: #64748b;
  }

  .empty-state p {
    margin: 0 0 8px;
  }
</style>
