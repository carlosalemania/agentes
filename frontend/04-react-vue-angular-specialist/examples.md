# React/Vue/Angular Specialist - Examples

---

## Example 1: React Performance Optimization

```jsx
import { useState, useMemo, useCallback, memo } from 'react';

// Memoized child component
const TodoItem = memo(({ todo, onToggle, onDelete }) => {
  console.log('TodoItem render:', todo.id);

  return (
    <li>
      <input
        type="checkbox"
        checked={todo.completed}
        onChange={() => onToggle(todo.id)}
      />
      <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
        {todo.text}
      </span>
      <button onClick={() => onDelete(todo.id)}>Delete</button>
    </li>
  );
});

function TodoList() {
  const [todos, setTodos] = useState([
    { id: 1, text: 'Learn React', completed: false },
    { id: 2, text: 'Build app', completed: false }
  ]);
  const [filter, setFilter] = useState('all');

  // Memoized filtered list
  const filteredTodos = useMemo(() => {
    if (filter === 'completed') return todos.filter(t => t.completed);
    if (filter === 'active') return todos.filter(t => !t.completed);
    return todos;
  }, [todos, filter]);

  // Memoized callbacks
  const handleToggle = useCallback((id) => {
    setTodos(prev => prev.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  }, []);

  const handleDelete = useCallback((id) => {
    setTodos(prev => prev.filter(todo => todo.id !== id));
  }, []);

  return (
    <div>
      <select value={filter} onChange={e => setFilter(e.target.value)}>
        <option value="all">All</option>
        <option value="active">Active</option>
        <option value="completed">Completed</option>
      </select>

      <ul>
        {filteredTodos.map(todo => (
          <TodoItem
            key={todo.id}
            todo={todo}
            onToggle={handleToggle}
            onDelete={handleDelete}
          />
        ))}
      </ul>
    </div>
  );
}
```

## Example 2: Vue 3 Composable

```vue
<!-- composables/useApi.js -->
<script>
import { ref, watchEffect } from 'vue';

export function useApi(url) {
  const data = ref(null);
  const error = ref(null);
  const loading = ref(false);

  const fetchData = async () => {
    loading.value = true;
    error.value = null;

    try {
      const response = await fetch(url.value);
      if (!response.ok) throw new Error('Failed to fetch');
      data.value = await response.json();
    } catch (err) {
      error.value = err.message;
    } finally {
      loading.value = false;
    }
  };

  watchEffect(() => {
    if (url.value) fetchData();
  });

  return { data, error, loading, refetch: fetchData };
}
</script>

<!-- UserProfile.vue -->
<script setup>
import { ref, computed } from 'vue';
import { useApi } from './composables/useApi';

const userId = ref(1);
const apiUrl = computed(() => `/api/users/${userId.value}`);

const { data: user, error, loading } = useApi(apiUrl);
</script>

<template>
  <div>
    <div v-if="loading">Loading...</div>
    <div v-else-if="error">Error: {{ error }}</div>
    <div v-else-if="user">
      <h1>{{ user.name }}</h1>
      <p>{{ user.email }}</p>
    </div>
  </div>
</template>
```

---

**Versión:** 1.0.0
