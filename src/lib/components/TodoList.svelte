<script lang="ts">
    import { Button } from "$lib/components/ui/button";
    import { Input } from "$lib/components/ui/input";
    import { Checkbox } from "$lib/components/ui/checkbox";
    import { Plus, Trash2, CheckCircle2, Circle } from 'lucide-svelte';
    import { fly, fade } from 'svelte/transition';
    import { flip } from 'svelte/animate';
    import { onMount } from "svelte";
    import { browser } from '$app/environment';

    let todos: { id: number; text: string; completed: boolean }[] = [];
    let newTodoText = "";
    let filter: 'all' | 'active' | 'completed' = 'all';

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
            todos = [{ id: Date.now(), text: newTodoText.trim(), completed: false }, ...todos];
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

    function clearCompleted() {
        todos = todos.filter(todo => !todo.completed);
        saveTodos();
    }

    $: filteredTodos = filter === 'all' 
        ? todos 
        : filter === 'active' 
            ? todos.filter(t => !t.completed) 
            : todos.filter(t => t.completed);

    $: activeCount = todos.filter(t => !t.completed).length;
    $: completedCount = todos.length - activeCount;

    onMount(() => {
        loadTodos();
    });
</script>

<div class="bg-white dark:bg-gray-800 rounded-lg shadow-lg p-6">
    <h2 class="text-2xl font-bold mb-4 dark:text-white">Todo List</h2>
    
    <form on:submit|preventDefault={addTodo} class="flex space-x-2 mb-6">
        <Input 
            type="text" 
            bind:value={newTodoText} 
            placeholder="Add a new task" 
            class="flex-grow dark:bg-gray-700 dark:text-white dark:border-gray-600"
        />
        <Button type="submit" class="bg-blue-500 hover:bg-blue-600 dark:bg-blue-600 dark:hover:bg-blue-700">
            <Plus size={20} />
        </Button>
    </form>

    <div class="mb-4 flex justify-center space-x-2">
        <Button 
            variant={filter === 'all' ? 'default' : 'outline'} 
            on:click={() => filter = 'all'}
            class="text-sm"
        >
            All ({todos.length})
        </Button>
        <Button 
            variant={filter === 'active' ? 'default' : 'outline'} 
            on:click={() => filter = 'active'}
            class="text-sm"
        >
            Active ({activeCount})
        </Button>
        <Button 
            variant={filter === 'completed' ? 'default' : 'outline'} 
            on:click={() => filter = 'completed'}
            class="text-sm"
        >
            Completed ({completedCount})
        </Button>
    </div>

    <ul class="space-y-2 mb-4">
        {#each filteredTodos as todo (todo.id)}
            <li 
                animate:flip={{ duration: 300 }}
                in:fly={{ y: 20, duration: 300 }}
                out:fade={{ duration: 300 }}
                class="flex items-center space-x-2 bg-gray-100 dark:bg-gray-700 p-3 rounded-lg transition-all duration-300 hover:shadow-md"
            >
                <button 
                    on:click={() => toggleTodo(todo.id)} 
                    class="text-gray-500 hover:text-blue-500 dark:text-gray-400 dark:hover:text-blue-400 transition-colors duration-200"
                >
                    {#if todo.completed}
                        <CheckCircle2 size={20} />
                    {:else}
                        <Circle size={20} />
                    {/if}
                </button>
                <span class={`flex-grow ${todo.completed ? 'line-through text-gray-500 dark:text-gray-400' : 'dark:text-white'}`}>
                    {todo.text}
                </span>
                <Button 
                    variant="ghost" 
                    size="sm" 
                    on:click={() => deleteTodo(todo.id)}
                    class="text-red-500 hover:text-red-700 dark:text-red-400 dark:hover:text-red-300"
                >
                    <Trash2 size={16} />
                </Button>
            </li>
        {/each}
    </ul>

    {#if completedCount > 0}
        <div class="flex justify-end">
            <Button variant="outline" size="sm" on:click={clearCompleted}>
                Clear completed
            </Button>
        </div>
    {/if}
</div>

<style>
    /* Можете добавить дополнительные стили здесь, если необходимо */
</style>