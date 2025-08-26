<script lang="ts">
	import moon from '$lib/assets/icon-moon.svg';
	import sun from '$lib/assets/icon-sun.svg';
	import { onMount } from 'svelte';

	interface Todo {
		id: Date;
		text: string;
		isDone: boolean;
	}

	type Filter = 'all' | 'active' | 'completed';

	type Theme = 'dark' | 'light';

	let todos = $state<Todo[]>([]);
	let filter = $state<Filter>('all');
	let inputText = $state('');
	let filteredTodos = $derived(filterTodos());
	let themeName = $state<Theme>('dark');

	$effect(() => {
		const htmlEl = document.documentElement;
		if (themeName === 'dark') {
			htmlEl.classList.add('dark');
		} else {
			htmlEl.classList.remove('dark');
		}
	});

	// $effect(() => {
	// 	const savedTodos = localStorage.getItem('todos');
	// 	if (!savedTodos) return;
	// 	savedTodos && (todos = JSON.parse(savedTodos));
	// 	// parse to convert local storage json string to real javascript
	// });

	// $effect(() => {
	// 	localStorage.setItem('todos', JSON.stringify(todos));
	// });

	// Load todos from localStorage when the component mounts
	onMount(() => {
		const savedTodos = localStorage.getItem('todos');
		if (savedTodos) {
			todos = JSON.parse(savedTodos);
		}

		const savedTheme = localStorage.getItem('theme');
		if (savedTheme === 'light' || savedTheme === 'dark') {
			themeName = savedTheme;
		}
	});

	$effect(() => {
		localStorage.setItem('theme', themeName);
	});

	// Function to save todos to localStorage
	function saveTodos() {
		localStorage.setItem('todos', JSON.stringify(todos));
	}

	// Called on keydown in input: only add todo on Enter
	function handleKeydown(event: KeyboardEvent) {
		if (event.key === 'Enter' && inputText.trim()) {
			addTodo();
		}
	}

	// Called on button click: add todo directly
	function addTodo() {
		if (!inputText.trim()) return;
		todos = [...todos, { id: new Date(), text: inputText.trim(), isDone: false }];
		inputText = ''; // clear input after adding
		saveTodos();
	}

	function filterTodos() {
		switch (filter) {
			case 'all':
				return todos;
			case 'active':
				return todos.filter((todo) => !todo.isDone);
			case 'completed':
				return todos.filter((todo) => todo.isDone);
		}
	}

	let dragedIndex: number | null = null;

	function handleDragStart(index: number) {
		dragedIndex = index;
	}

	function handleDrop(index: number) {
		if (dragedIndex === null || dragedIndex === index) return;
		const updated = [...todos];
		const [movedItem] = updated.splice(dragedIndex, 1);
		updated.splice(index, 0, movedItem);
		todos = updated;
		saveTodos();
		dragedIndex = null;
	}
</script>

<div class="grid items-center justify-center gap-2 bg-white p-8 dark:bg-gray-800">
	<div class="flex items-center justify-between">
		<h1 class="font-extrabold tracking-widest dark:text-white">TODO</h1>
		<button
			onclick={() => {
				if (themeName === 'dark') {
					themeName = 'light';
				} else {
					themeName = 'dark';
				}
				console.log(themeName);
			}}
			class="cursor-pointer"
		>
			<span class="sr-only">theme sitcher</span>
			<img
				src={themeName === 'dark' ? sun : moon}
				alt="theme switcher"
				class="rounded-full bg-gray-800 p-1 dark:bg-amber-600"
			/>
		</button>
	</div>
	<div class="flex items-center justify-center gap-1">
		<input
			type="text"
			placeholder="Add todo"
			bind:value={inputText}
			onkeydown={handleKeydown}
			class="text-gray-800 dark:bg-gray-800 dark:text-gray-200 dark:placeholder-gray-100"
		/>
		<button onclick={addTodo} class="rounded-sm border p-2 active:scale-95 dark:text-white"
			>add todo</button
		>
	</div>
	{#if todos.length === 0}
		<p class="text-gray-500 dark:text-gray-300">No todos yet</p>
	{:else}
		{#each filteredTodos as todo, i}
			<div
				class="rounded-md border p-2 dark:border-gray-400"
				draggable="true"
				ondragstart={() => handleDragStart(i)}
				ondragover={(e) => e.preventDefault()}
				ondrop={() => handleDrop(i)}
				role="listitem"
			>
				<input
					type="checkbox"
					id={'checkbox-' + todo.id}
					class="peer hidden"
					checked={todo.isDone}
					onchange={() => {
						todos = todos.map((t) => (t.id === todo.id ? { ...t, isDone: !t.isDone } : t));
						saveTodos();
					}}
				/>
				<label
					for={'checkbox-' + todo.id}
					class="relative top-1/2 inline-block h-5 w-5 translate-y-1/4 cursor-pointer rounded border-2 border-gray-500
        peer-checked:border-green-500 peer-checked:bg-green-500
        dark:border-gray-300 dark:peer-checked:bg-green-400"
				></label>
				<input
					type="text"
					value={todo.text}
					oninput={(e) => {
						const newText = e.currentTarget.value;
						todos = todos.map((t) => (t.id === todo.id ? { ...t, text: newText } : t));
						saveTodos();
					}}
					readonly={todo.isDone}
					class={`border-none font-bold outline-none dark:bg-gray-800 dark:text-gray-200  ${
						todo.isDone ? 'text-gray-600 line-through dark:text-gray-400' : 'text-gray-900'
					}`}
				/>
				<button
					onclick={() => {
						const rmo = todos.findIndex((t) => t.id === todo.id);
						~rmo && todos.splice(rmo, 1);
						saveTodos();
					}}
					class="p-2 text-red-600 hover:cursor-pointer active:scale-95 dark:text-red-400"
				>
					X
				</button>
			</div>
		{/each}

		<div class="flex items-center justify-between">
			<p class="dark:text-white">{todos.filter((t) => !t.isDone).length} items left</p>
			<div
				class="flex gap-2 [&>*]:cursor-pointer [&>*]:rounded-sm [&>*]:border [&>*]:p-0.5 [&>*]:dark:text-white"
			>
				{#each ['all', 'active', 'completed'] as f}
					<button
						class:selected={filter === f}
						aria-pressed={filter === f}
						onclick={() => {
							filter = f as Filter;
						}}
					>
						{f}
					</button>
				{/each}
			</div>
		</div>
	{/if}
</div>

<style>
	.selected {
		background-color: slategrey;
		color: white;
	}

	.dark .selected {
		background-color: cornflowerblue;
		color: black;
	}
</style>
