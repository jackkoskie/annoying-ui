<script lang="ts">
	import { goto } from '$app/navigation';
	import Button, { buttonVariants } from '$lib/components/ui/button/button.svelte';
	import { data } from '../state.svelte';
	let phone: number[] = $state([]);
	let currentLocation: [number, number] = $state([0, 0]);

	console.warn(
		'STOP! Below you will see the entire maze, which spoils some of the fun of the game. Only use this if you are really stuck or want to see the maze layout.'
	);

	type Cell = { number: number | null; walls: [boolean, boolean, boolean, boolean] };

	const SIZE = 10;
	let maze: Cell[][] = [];

	// Initialize the maze with all walls intact and no numbers
	for (let y = 0; y < SIZE; y++) {
		maze[y] = [];
		for (let x = 0; x < SIZE; x++) {
			maze[y][x] = { number: null, walls: [true, true, true, true] };
		}
	}

	// Direction vectors: up, right, down, left
	const dirs = [
		[-1, 0],
		[0, 1],
		[1, 0],
		[0, -1]
	];

	const oppositeWall = [2, 3, 0, 1];

	function generateMaze(x: number, y: number, visited: boolean[][]) {
		visited[y][x] = true;
		const directions = [0, 1, 2, 3].sort(() => Math.random() - 0.5);
		for (let dir of directions) {
			const nx = x + dirs[dir][1];
			const ny = y + dirs[dir][0];
			if (nx >= 0 && nx < SIZE && ny >= 0 && ny < SIZE && !visited[ny][nx]) {
				maze[y][x].walls[dir] = false;
				maze[ny][nx].walls[oppositeWall[dir]] = false;
				generateMaze(nx, ny, visited);
			}
		}
	}

	const visited = Array.from({ length: SIZE }, () => Array(SIZE).fill(false));
	generateMaze(0, 0, visited);

	const positions = new Set<string>();
	while (positions.size < 10) {
		const x = Math.floor(Math.random() * SIZE);
		const y = Math.floor(Math.random() * SIZE);
		const key = `${x},${y}`;
		if (!positions.has(key)) {
			maze[y][x].number = positions.size; // number from 0 to 9
			positions.add(key);
		}
	}

	// make outer walls
	for (let x = 0; x < SIZE; x++) {
		maze[0][x].walls[0] = true;
		maze[SIZE - 1][x].walls[2] = true;
	}
	for (let y = 0; y < SIZE; y++) {
		maze[y][0].walls[3] = true;
		maze[y][SIZE - 1].walls[1] = true;
	}

	// Logging function generated with the help of AI
	// to visualize the maze in the console as a development aid
	// Log the maze with visible walls
	function logMaze(maze: Cell[][], currentLocation: [number, number] = [0, 0]) {
		const [px, py] = currentLocation;
		const SIZE = maze.length;
		let output = '';
		for (let y = 0; y < SIZE; y++) {
			// Top walls
			let top = '';
			let mid = '';
			for (let x = 0; x < SIZE; x++) {
				top += maze[y][x].walls[0] ? '+---' : '+   ';
				mid += maze[y][x].walls[3] ? '|' : ' ';
				mid += ' ';
				if (x === px && y === py) {
					mid += 'X';
				} else {
					mid += maze[y][x].number !== null ? maze[y][x].number!.toString() : '.';
				}
				mid += ' ';
			}
			top += '+\n';
			mid += (maze[y][SIZE - 1].walls[1] ? '|' : ' ') + '\n';
			output += top + mid;
		}
		// Bottom walls for the last row
		let bottom = '';
		for (let x = 0; x < SIZE; x++) {
			bottom += maze[SIZE - 1][x].walls[2] ? '+---' : '+   ';
		}
		bottom += '+\n';
		output += bottom;
		console.log(output);
	}

	logMaze(maze);

	phone = data.last ? data.last.toString().padStart(4, '0').split('').map(Number) : [];
</script>

<section class="flex min-h-screen flex-col items-center justify-center gap-3">
	<h1>
		Please attempt to select the last part of your phone number by finding the digits in the {SIZE}x{SIZE}
		labarynth below.
	</h1>
	{#if phone.length > 0}
		<h2>Currently Entered Phone Number:</h2>
		<div class="flex flex-row gap-2">
			{#each phone as number, index}
				<Button
					class="hover:bg-destructive"
					onclick={() => (phone = phone.filter((_, i) => i !== index))}>{number}</Button
				>
			{/each}
		</div>
	{/if}
	<div class="flex flex-row justify-center gap-2">
		<Button
			disabled={maze[currentLocation[1]][currentLocation[0]].walls[3] || phone.length >= 4}
			onclick={() => {
				currentLocation[0] -= 1;
				logMaze(maze, currentLocation);
			}}>Left</Button
		>
		<Button
			disabled={maze[currentLocation[1]][currentLocation[0]].walls[0] || phone.length >= 4}
			onclick={() => {
				currentLocation[1] -= 1;
				logMaze(maze, currentLocation);
			}}>Up</Button
		>
		<Button
			disabled={maze[currentLocation[1]][currentLocation[0]].walls[2] || phone.length >= 4}
			onclick={() => {
				currentLocation[1] += 1;
				logMaze(maze, currentLocation);
			}}>Down</Button
		>
		<Button
			disabled={maze[currentLocation[1]][currentLocation[0]].walls[1] || phone.length >= 4}
			onclick={() => {
				currentLocation[0] += 1;
				logMaze(maze, currentLocation);
			}}>Right</Button
		>
	</div>
	{#if phone.length >= 4}
		<p>
			You have entered the last section of your phone number. Remove digits to continue selecting or
			press back to return to the home page
		</p>
	{:else}
		<p>Current Location: {currentLocation}</p>
		{#if currentLocation[0] === 0 && currentLocation[1] === 0}
			<p>You are at the start, use the directions below to move around</p>
		{/if}
		{#if currentLocation[0] !== 0 || currentLocation[1] !== 0}
			{#if maze[currentLocation[1]][currentLocation[0]].number === null}
				<p>Theres Nothing Here...</p>
			{:else}
				<p>
					You found a {maze[currentLocation[1]][currentLocation[0]].number}! Would you like to add
					it to you're phone number?
				</p>
				<div class="flex flex-row gap-2">
					<Button
						onclick={() => {
							phone.push(maze[currentLocation[1]][currentLocation[0]].number!);
							currentLocation = [0, 0];
						}}>Yes, then return to start</Button
					>
				</div>
				<p>To continue exploring, use the direction buttons above</p>
			{/if}
		{/if}
	{/if}
	<Button
		class="fixed bottom-5"
		onclick={() => {
			data.last = Number(phone.join(''));
			goto('/');
		}}>Back</Button
	>
</section>
