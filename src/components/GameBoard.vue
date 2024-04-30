<script setup lang="ts">
	import { ref, computed, watch } from 'vue';
	const players = ['X', 'O'] as const;
	const activePlayer = ref('X');
	const squares = ref([
		['', '', ''],
		['', '', ''],
		['', '', ''],
	]);
	const gameState = computed<String | null>(() => checkGameState());
	const checkGameState = () => {
		let gameState = null;
		players.forEach((player) => {
			for (let i = 0; i < 3; i++) {
				if (squares.value[i].every((cell) => cell === player))
					gameState = player;
				if (squares.value.every((row) => row[i] === player))
					gameState = player;
			}

			if (squares.value[1][1] === player) {
				if (
					squares.value[0][0] === player &&
					squares.value[2][2] === player
				)
					gameState = player;
				if (
					squares.value[0][2] === player &&
					squares.value[2][0] === player
				)
					gameState = player;
			}
		});
		if (
			!gameState &&
			squares.value.every((row) => row.every((cell) => cell))
		) {
			return 'tie';
		}
		return gameState;
	};
	const move = (x: number, y: number) => {
		if (gameState.value) return;
		squares.value[x][y] = activePlayer.value;
		activePlayer.value =
			activePlayer.value === players[0] ? players[1] : players[0];
	};
	const reset = () => {
		activePlayer.value = players[0];
		squares.value = [
			['', '', ''],
			['', '', ''],
			['', '', ''],
		];
	};
	const bestMove = (board: string[][], npc: 'X' | 'O') => {
		let copyBoard = [...board];
		let bestScore = -Infinity;
		let bestMove: {
			i: number;
			j: number;
		};
		for (let i = 0; i < 3; i++) {
			for (let j = 0; j < 3; j++) {
				if (copyBoard[i][j] == '') {
					copyBoard[i][j] = npc;
					let score = minimax(copyBoard);
					if (score > bestScore) {
						bestScore = score;
						bestMove = { i, j };
					}
					copyBoard[i][j] = '';
				}
			}
		}
		return bestMove;
	};
	const minimax = (board) => {
		return 1;
	};
	watch(activePlayer, () => {
		if (gameState.value) return;
		if (activePlayer.value === 'X') {
			return;
		}
		let { i, j } = bestMove(squares.value, 'O');
		move(i, j);
	});
</script>
<template>
	<div>
		<h1>Tic Tac Toe</h1>
		<p v-if="gameState && gameState !== 'tie'">
			The winner is: {{ gameState }}
		</p>
		<p v-else-if="gameState">It's a tie!</p>
		<p v-else>Active Player is: {{ activePlayer }}</p>
		<div>
			<div
				class="row"
				v-for="(row, rowIndex) of squares"
				:key="rowIndex"
			>
				<button
					class="cell"
					v-for="(cell, cellIndex) of row"
					:key="cellIndex"
					:disabled="!!cell"
					:class="{
						'cell-x': cell === 'X',
						'cell-o': cell === 'O',
					}"
					:style="{ cursor: !cell ? 'pointer' : 'not-allowed' }"
					@click="move(rowIndex, cellIndex)"
				>
					{{ cell }}
				</button>
			</div>
		</div>
		<button @click="reset">Reset</button>
	</div>
</template>
<style scoped>
	.row {
		display: flex;
	}
	.cell {
		width: 50px;
		height: 50px;
		margin-right: -1px;
		margin-bottom: -1px;
		line-height: 50px;
		text-align: center;
		border: 1px solid #bbb;
		font-size: 40px;
	}
	.cell-o {
		color: blue;
	}
	.cell-x {
		color: red;
	}
</style>
