<script>
	import { onMount } from "svelte";

	const TICK_TIME = 100;
	const BOARD_DIMENSIONS = { x: 20, y: 20 };

	let snake = [
		{ x: 4, y: 4 },
		{ x: 4, y: 3 },
		{ x: 4, y: 2 },
	];
	let apple = { x: 3, y: 4 };
	let score = 0;

	const CELL_SIZE = 25;

	function pos(coordinate) {
		return `top: ${CELL_SIZE * coordinate.y}px; left: ${
			CELL_SIZE * coordinate.x
		}px`;
	}

	let up = { x: 0, y: -1 };
	let down = { x: 0, y: 1 };
	let left = { x: -1, y: 0 };
	let right = { x: 1, y: 0 };

	let currentDirection = down;

	let directions = {
		37: left,
		38: up,
		39: right,
		40: down,
	};

	function equals(pos1, pos2) {
		return pos1.x === pos2.x && pos1.y === pos2.y;
	}

	function randomPoint() {
		let x = Math.floor(Math.random() * BOARD_DIMENSIONS.x);
		let y = Math.floor(Math.random() * BOARD_DIMENSIONS.y);
		return { x, y };
	}

	function randomUnoccipiedPoint() {
		let point = randomPoint();

		while (snake.some((bodyPart) => equals(bodyPart, point))) {
			point = randomPoint();
		}
		return point;
	}

	function gameLoop() {
		snake = newSnake(currentDirection);
	}

	let stopGameLoop = () => {};

	onMount(async () => {
		const intervalRef = setInterval(gameLoop, 120);
		stopGameLoop = () => clearInterval(intervalRef);
		return stopGameLoop;
	});

	let shouldGrow = false;
	$: if (equals(snake[0], apple)) {
		shouldGrow = true;
		apple = randomUnoccipiedPoint();
	}

	$: if (isOutsideBoard(snake[0])) {
		stopGameLoop();
		alert("you suck");
	}

	function isOutsideBoard(point) {
		return (
			point.x < 0 ||
			point.x >= BOARD_DIMENSIONS.x ||
			point.y < 0 ||
			point.y >= BOARD_DIMENSIONS.y
		);
	}

	function newSnake(direction) {
		let head = snake[0];
		let newHead = { x: head.x + direction.x, y: head.y + direction.y };

		const foo = [newHead, ...snake];
		if (!shouldGrow) foo.pop();

		shouldGrow = false;
		return foo;
	}

	function onKeydown(event) {
		let direction = directions[event.keyCode];
		if (!direction) {
			return;
		}

		currentDirection = direction;

		event.preventDefault();
	}
</script>

<svelte:body on:keydown={onKeydown} />
<div class="main-content min-width">
	<div class="score">{score}</div>

	<div
		class="board"
		style="--cell-size: {CELL_SIZE}px; --tick-time: {TICK_TIME}ms; --board-size-x: {BOARD_DIMENSIONS.x}; --board-size-y: {BOARD_DIMENSIONS.y}"
	>
		{#each snake as bodyPart}
			<div class="body-part" style={pos(bodyPart)} />
		{/each}

		<div class="apple" style={pos(apple)} />
	</div>
</div>

<style>
	.modal-container {
		position: absolute;
		left: 50%;
		top: 2rem;
	}

	.min-width {
		width: min-content;
	}

	.main-content {
		margin: 1rem auto 0 auto;
		display: grid;
		grid-template-columns: auto;
		gap: 2rem;
		align-items: center;
		text-align: center;
	}

	.score {
		justify-self: right;
	}

	.body-part,
	.skull,
	.apple {
		position: absolute;
		width: var(--cell-size);
		height: var(--cell-size);
		text-align: center;
		line-height: 1;
	}

	.body-part {
		background-color: var(--snake-color);
	}

	.head {
		transform: scale(1.3);
	}

	.head,
	.tail {
		transition: top var(--tick-time) linear, left var(--tick-time) linear;
	}

	.apple {
		transform: scale(1.1);
		font-size: calc(var(--cell-size) * 0.8);
	}

	.apple::before {
		content: var(--food-emoji);
	}

	.skull::before {
		content: "☠️";
		overflow: hidden;
	}

	.skull {
		/* Transform cannot be applied to a before element, so we must apply it here */
		transform: scale(3, 3) translateY(4px);
		transform-origin: 50% 50%;
	}

	.board {
		--border-width: var(--cell-size);
		width: calc(var(--board-size-x) * var(--cell-size));
		height: calc(var(--board-size-y) * var(--cell-size));

		position: relative;
		margin: var(--border-width);
		outline: var(--border-width) solid var(--board-outline-color);

		background-image: linear-gradient(
				45deg,
				var(--checker-color) 25%,
				transparent 25%
			),
			linear-gradient(-45deg, var(--checker-color) 25%, transparent 25%),
			linear-gradient(45deg, transparent 75%, var(--checker-color) 75%),
			linear-gradient(-45deg, transparent 75%, var(--checker-color) 75%);
		background-size: calc(var(--cell-size) * 2) calc(var(--cell-size) * 2);
		background-position: 0 0, 0 var(--cell-size),
			var(--cell-size) calc(-1 * var(--cell-size)),
			calc(-1 * var(--cell-size)) 0;
	}
</style>
