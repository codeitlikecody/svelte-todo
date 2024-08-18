<script lang="ts">
	// import { text } from '@sveltejs/kit';
	import { Input } from '$lib/components/ui/input';
	import { Checkbox } from '$lib/components/ui/checkbox';
	import { Button } from '$lib/components/ui/button';

	import { ModeWatcher } from 'mode-watcher';

	import Sun from 'lucide-svelte/icons/sun';
	import Moon from 'lucide-svelte/icons/moon';

	import { resetMode, setMode } from 'mode-watcher';
	import * as DropdownMenu from '$lib/components/ui/dropdown-menu/index.js';

	type Todo = {
		text: string;
		done: boolean;
	};
	type Filters = 'all' | 'active' | 'completed';

	let todos = $state<Todo[]>([]);
	let filter = $state<Filters>('all');
	let filteredTodos = $derived(filterTodos());

	// Run on mount
	$effect(() => {
		const savedTodos = localStorage.getItem('todos');
		savedTodos && (todos = JSON.parse(savedTodos));
		console.log(savedTodos);
	});

	// Run on update todos
	$effect(() => {
		localStorage.setItem('todos', JSON.stringify(todos));
		console.log(todos);
	});

	// Run on update filter state
	$effect(() => {
		console.log(filter);
	});

	function addTodo(event: KeyboardEvent & { currentTarget: EventTarget & HTMLInputElement }) {
		if (event.key !== 'Enter') return;

		const todoEl = event.target as HTMLInputElement;
		// const id = window.crypto.randomUUID();
		const text = todoEl.value;
		const done = false;

		todos.push({ text, done });

		todoEl.value = '';
	}

	function editTodo(event: Event & { currentTarget: EventTarget & HTMLInputElement }) {
		const inputEl = event.target as HTMLInputElement;
		const index = Number(inputEl.dataset.index);
		todos[index].text = inputEl.value;
	}

	function setFilter(newFilter: Filters) {
		filter = newFilter;
	}

	function filterTodos() {
		switch (filter) {
			case 'all':
				return todos;

			case 'active':
				return todos.filter((todo) => !todo.done);

			case 'completed':
				return todos.filter((todo) => todo.done);
		}
	}

	function countAllTodos() {
		return todos.length;
	}

	function countActiveTodos() {
		return todos.filter((todo) => !todo.done).length;
	}

	function deleteCompleted() {
		todos = todos.filter((todo) => !todo.done);
	}

	function toggleTodo(i: number): void {
		console.log(`toggling todo index ${i}`);
		todos[i].done = !todos[i].done;
		if (!todos[i].done) {
			console.log('Task completed');
		} else {
			console.log('Task active');
		}
	}
</script>

<div class="app-wrapper">
	<div class="navbar">
		<h1>Bonnie's todo list</h1>
		<div id="menu-buttons">
			<DropdownMenu.Root>
				<DropdownMenu.Trigger asChild let:builder>
					<Button builders={[builder]} variant="outline" size="icon">
						<Sun
							class="h-[1.2rem] w-[1.2rem] rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0"
						/>
						<Moon
							class="absolute h-[1.2rem] w-[1.2rem] rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100"
						/>
						<span class="sr-only">Toggle theme</span>
					</Button>
				</DropdownMenu.Trigger>
				<DropdownMenu.Content align="end">
					<DropdownMenu.Item on:click={() => setMode('light')}>Light</DropdownMenu.Item>
					<DropdownMenu.Item on:click={() => setMode('dark')}>Dark</DropdownMenu.Item>
					<DropdownMenu.Item on:click={() => resetMode()}>System</DropdownMenu.Item>
				</DropdownMenu.Content>
			</DropdownMenu.Root>
		</div>
	</div>
	<div class="todo-app">
		<Input onkeydown={addTodo} placeholder="Add Todo" type="text" />

		<ModeWatcher />
		<div>
			<div class="todo-list">
				{#each filteredTodos as todo, i}
					<div class:completed={todo.done} class="todo-item">
						<Checkbox class="checkbox" on:click={() => toggleTodo(i)} checked={todo.done} />
						<Input
							oninput={editTodo}
							data-index={i}
							value={todo.text}
							type="text"
							class="todo-label"
						/>
					</div>
				{/each}
			</div>

			<div class="filters">
				<Button
					variant={filter == 'all' ? 'default' : 'secondary'}
					on:click={() => setFilter('all')}>All</Button
				>
				<Button
					variant={filter == 'active' ? 'default' : 'secondary'}
					on:click={() => setFilter('active')}>Active</Button
				>
				<Button
					variant={filter == 'completed' ? 'default' : 'secondary'}
					on:click={() => setFilter('completed')}>Completed</Button
				>
			</div>

			<div>
				<Button variant="destructive" onclick={() => deleteCompleted()}>Delete Completed</Button>
			</div>

			<p>{countAllTodos()} tasks</p>
			<p>{countActiveTodos()} remaining</p>
		</div>
	</div>
</div>

<style>
	h1 {
		font-size: 3rem;
	}

	p {
		margin-top: 1rem;
	}

	.app-wrapper {
		margin: 3rem;
	}

	.todo-list {
		margin-block-start: 1rem;
		justify-items: center;
	}

	.menu-buttons {
		margin-left: auto;
	}

	.navbar {
		display: flex;
		margin-left: 0rem;
		gap: 1rem;
		align-items: center;
		justify-content: space-between;
	}

	.todo-item {
		gap: 1rem;
		display: flex;
		margin-left: 0rem;
		margin-top: 1rem;
		align-items: center;
	}

	.checkbox {
		padding: 5px;
	}

	.todo-label {
		transition: opacity 0.3s;
	}

	.completed {
		opacity: 0.4;
	}

	.filters {
		margin-block: 1rem;
	}
</style>
