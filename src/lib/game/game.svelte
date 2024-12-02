<script lang="ts">
	import { onMount, onDestroy } from 'svelte';
	import Info from './info.svelte';
	import { handleTouchStart, handleTouchEnd } from '../swipe/index.svelte';
	// Game constants
	const GRID_SIZE = 20;
	const CELL_SIZE = 20;
	const DIRECTIONS = {
		ArrowUp: { x: 0, y: -1 },
		ArrowDown: { x: 0, y: 1 },
		ArrowLeft: { x: -1, y: 0 },
		ArrowRight: { x: 1, y: 0 }
	};

	// Reactive game state
	let snake = $state([{ x: 10, y: 10 }]);
	let food = $state(generateFoodPosition());
	let direction = $state(DIRECTIONS.ArrowRight);
	let gameOver = $state(false);
	let gameStarted = $state(false);
	let score = $state(0);

	// Generate random food position
	function generateFoodPosition() {
		return {
			x: Math.floor(Math.random() * GRID_SIZE),
			y: Math.floor(Math.random() * GRID_SIZE)
		};
	}

	// Game loop
	function gameLoop() {
		if (gameOver || !gameStarted) return;

		const head = { ...snake[0] };
		head.x += direction.x;
		head.y += direction.y;

		// Wall collision
		if (head.x < 0 || head.x >= GRID_SIZE || head.y < 0 || head.y >= GRID_SIZE) {
			gameOver = true;
			gameStarted = false;
			return;
		}

		// Self collision
		if (snake.some((segment) => segment.x === head.x && segment.y === head.y)) {
			gameOver = true;
			gameStarted = false;
			return;
		}

		// Food collection
		if (head.x === food.x && head.y === food.y) {
			score++;
			food = generateFoodPosition();
		} else {
			snake.pop();
		}

		snake.unshift(head);
	}

	// Handle key presses
	function handleKeydown(event: KeyboardEvent) {
		if (!gameStarted) {
			if (event.key === 'Enter' || event.key === ' ') {
				startGame();
			}
			return;
		}

		const newDirection = DIRECTIONS[event.key as keyof typeof DIRECTIONS];
		if (newDirection) {
			// Prevent 180-degree turns
			if (newDirection.x !== -direction.x || newDirection.y !== -direction.y) {
				direction = newDirection;
			}
		}
	}

	// Start game
	function startGame() {
		resetGame();
		gameStarted = true;
	}

	// Reset game
	function resetGame() {
		snake = [{ x: 10, y: 10 }];
		food = generateFoodPosition();
		direction = DIRECTIONS.ArrowRight;
		gameOver = false;
		score = 0;
	}

	// Game loop interval
	let intervalId: number;
	onMount(() => {
		window.addEventListener('keydown', handleKeydown);
		intervalId = setInterval(gameLoop, 200);
		return () => {
			window.removeEventListener('keydown', handleKeydown);
			clearInterval(intervalId);
		};
	});
</script>

{#if gameStarted}
	<div
		class="swipe-background"
		role="button"
		tabindex="0"
		ontouchstart={handleTouchStart}
		ontouchend={(e) => {
			const ret = handleTouchEnd(e);
			handleKeydown({ key: ret });
		}}
	></div>
{/if}
<div class="game-container">
	{#if !gameStarted}
		<div class="start-screen">
			<h1>Snake Game</h1>
			{#if gameOver}
				<div class="game-over-dialog">
					<h2>Game Over!</h2>
					<p>Your Score: {score}</p>
				</div>
			{/if}
			<button onclick={startGame}>
				{gameOver ? 'Play Again' : 'Start Game'}
			</button>
			<p>Use Arrow Keys to Control</p>
		</div>
	{/if}

	{#if gameStarted}
		<div
			class="game-board"
			style="
				width: {GRID_SIZE * CELL_SIZE}px; 
				height: {GRID_SIZE * CELL_SIZE}px;
			"
		>
			{#each snake as segment}
				<div
					class="snake-segment"
					style="
						left: {segment.x * CELL_SIZE}px; 
						top: {segment.y * CELL_SIZE}px;
					"
				></div>
			{/each}

			<div
				class="food"
				style="
					left: {food.x * CELL_SIZE}px; 
					top: {food.y * CELL_SIZE}px;
				"
			></div>
		</div>

		<div class="score">Score: {score}</div>
	{/if}
	<Info />
</div>

<style>
	.game-container {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		height: 100vh;
		background-color: #f0f0f0;
	}

	.start-screen {
		text-align: center;
		background-color: white;
		padding: 30px;
		border-radius: 10px;
		box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
	}

	.game-over-dialog {
		background-color: #f8d7da;
		color: #721c24;
		padding: 15px;
		border-radius: 5px;
		margin-bottom: 20px;
	}

	button {
		background-color: #28a745;
		color: white;
		border: none;
		padding: 10px 20px;
		font-size: 18px;
		border-radius: 5px;
		cursor: pointer;
		margin-bottom: 15px;
	}

	.game-board {
		position: relative;
		background-color: white;
		border: 2px solid #333;
	}

	.snake-segment {
		position: absolute;
		width: 20px;
		height: 20px;
		background-color: green;
	}

	.food {
		position: absolute;
		width: 20px;
		height: 20px;
		background-color: red;
	}

	.score {
		margin-top: 10px;
		font-size: 18px;
	}
	.swipe-background {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
	}
</style>
