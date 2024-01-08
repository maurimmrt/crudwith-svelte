<script lang="ts">
	import { writable } from 'svelte/store';
	import PlusIcon from './components/PlusIcon.svelte';
	import CheckIcon from './components/CheckIcon.svelte';
	import XIcon from './components/XIcon.svelte';
	import ArrowUpIcon from './components/ArrowUpIcon.svelte';

	const storedUsers = localStorage.getItem('users');
	const initialUsers = storedUsers
		? JSON.parse(storedUsers)
		: ['Patricia Lebsack', 'Kurtis Weissnat', 'Glenna Reichert'];

	const indexingUser = writable<number | null>(null);
	const textField = writable('');
	const users = writable(initialUsers);

	users.subscribe(($users) => {
		localStorage.setItem('users', JSON.stringify($users));
	});

	const createUser = () => {
		if (!$textField) return;
		users.update(($users) => [...$users, $textField]);
		textField.set('');
	};

	const deleteUser = (indexUser: number) => {
		users.update(($users) =>
			$users.filter((_user: string, index: number) => index !== indexUser)
		);
	};

	const updatingUser = (newUser: string, indexUser: number) => {
		textField.set(newUser);
		indexingUser.set(indexUser);
	};

	const updatedUser = () => {
		users.update(($users) =>
			$users.map((user: string, index: number | null) =>
				index === $indexingUser ? $textField : user
			)
		);

		textField.set('');
		indexingUser.set(null);
	};

	const keyDownEnter = (e: KeyboardEvent) => {
		if (e.key !== 'Enter') return;

		if ($indexingUser !== null) {
			updatedUser();
		} else {
			createUser();
		}
	};
</script>

<main class="m-auto w-11/12 sm:container">
	<h1 class="text-4xl font-bold text-center mb-4">CRUD with Svelte</h1>

	<section class="flex">
		<input
			type="text"
			id="textField"
			class="px-3 py-2 my-auto mx-3 w-full border border-gray-300 rounded focus:outline-none focus:ring-2 focus:ring-indigo-600"
			placeholder="Write a name"
			bind:value={$textField}
			on:keydown={keyDownEnter}
		/>

		{#if $indexingUser !== null}
			<button
				type="button"
				class="p-2 bg-green-500 duration-150 text-white rounded-full active:bg-green-600"
				on:click={updatedUser}
			>
				<CheckIcon />
			</button>
		{:else}
			<button
				type="button"
				class="p-2 bg-indigo-500 duration-150 text-white rounded-full active:bg-indigo-600"
				on:click={createUser}
			>
				<PlusIcon />
			</button>
		{/if}
	</section>
	<label for="textField" class="ms-3 text-sm text-red-600">
		{#if $textField === ''}
			Field is required
		{/if}
	</label>

	<ul class="h-80 overflow-auto">
		{#each $users as user, indexUser}
			<li class="flex justify-between py-4 px-2 border-b-2">
				<span class="my-auto">{user}</span>
				<span class="whitespace-nowrap">
					<button
						type="button"
						class="p-2 bg-red-500 duration-150 text-white rounded-full active:bg-red-600"
						on:click={() => deleteUser(indexUser)}
					>
						<XIcon />
					</button>
					<button
						type="button"
						class="p-2 bg-indigo-500 duration-150 text-white rounded-full active:bg-indigo-600"
						on:click={() => updatingUser(user, indexUser)}
					>
						<ArrowUpIcon />
					</button>
				</span>
			</li>
		{/each}
	</ul>
</main>
