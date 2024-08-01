<!-- src/lib/components/TodoList.svelte -->
<script lang="ts">
    import { Button } from "$lib/components/ui/button";
    import { Input } from "$lib/components/ui/input";
    import { Checkbox } from "$lib/components/ui/checkbox";
    import { onMount } from "svelte";
    import { browser } from '$app/environment';

    let todos: { id: number; text: string; completed: boolean }[] = [];
    let newTodoText = "";

    function saveTodos() {
        if (browser) {
            localStorage.setItem('todos', JSON.stringify(todos));
        }
    }

    function loadTodos() {
        if (browser) {
            const storedTodos = localStorage.getItem('todos');
            if (storedTodos) {
                todos = JSON.parse(storedTodos);
            }
        }
    }

    function addTodo() {
        if (newTodoText.trim()) {
            todos = [...todos, { id: Date.now(), text: newTodoText.trim(), completed: false }];
            newTodoText = "";
            saveTodos();
        }
    }

    function deleteTodo(id: number) {
        todos = todos.filter(todo => todo.id !== id);
        saveTodos();
    }

    function toggleTodo(id: number) {
        todos = todos.map(todo => 
            todo.id === id ? { ...todo, completed: !todo.completed } : todo
        );
        saveTodos();
    }

    onMount(() => {
        loadTodos();
    });
</script>

<div class="space-y-4">
    <h2 class="text-2xl font-bold mb-4">Todo List</h2>
    
    <form on:submit|preventDefault={addTodo} class="flex space-x-2">
        <Input 
            type="text" 
            bind:value={newTodoText} 
            placeholder="Add a new task" 
            class="flex-grow"
        />
        <Button type="submit">Add</Button>
    </form>

    <ul class="space-y-2">
        {#each todos as todo (todo.id)}
            <li class="flex items-center space-x-2 bg-gray-100 p-2 rounded">
                <Checkbox 
                    checked={todo.completed} 
                    on:change={() => toggleTodo(todo.id)}
                />
                <span class={todo.completed ? 'line-through' : ''}>{todo.text}</span>
                <Button 
                    variant="destructive" 
                    size="sm" 
                    on:click={() => deleteTodo(todo.id)}
                >
                    Delete
                </Button>
            </li>
        {/each}
    </ul>
</div>