<script setup lang="ts">
	import { ref, computed, watch } from 'vue';
	const players = ['X', 'O'] as const;
	const activePlayer = ref('');
	const squares = ref([
		['', '', ''],
		['', '', ''],
		['', '', ''],
	]);
	const gameState = computed<String | null>(() =>
		checkGameState(squares.value)
	);
	const checkGameState = (board: string[][]) => {
		let gameState = null;
		players.forEach((player) => {
			for (let i = 0; i < 3; i++) {
				if (board[i].every((cell) => cell === player))
					gameState = player;
				if (board.every((row) => row[i] === player)) gameState = player;
			}

			if (board[1][1] === player) {
				if (board[0][0] === player && board[2][2] === player)
					gameState = player;
				if (board[0][2] === player && board[2][0] === player)
					gameState = player;
			}
		});
		if (!gameState && board.every((row) => row.every((cell) => cell))) {
			return 'tie';
		}
		return gameState;
	};
	const move = (x: number, y: number) => {
		if (gameState.value || !activePlayer.value) return;
		squares.value[x][y] = activePlayer.value;
		activePlayer.value =
			activePlayer.value === players[0] ? players[1] : players[0];
	};
	const reset = () => {
		activePlayer.value = '';
		squares.value = [
			['', '', ''],
			['', '', ''],
			['', '', ''],
		];
	};
	const chooseFirstPlayer = (player: 0 | 1) => {
		activePlayer.value = players[player];
	};
	const scores: { [key: string]: number } = {
		O: 10,
		X: -10,
		tie: 0,
	};
	const bestMove = (board: string[][], npc: 'X' | 'O') => {
		let copyBoard = [...board];
		let bestScore = -Infinity;
		let move: {
			i: number;
			j: number;
		} = { i: -1, j: -1 };
		for (let i = 0; i < 3; i++) {
			for (let j = 0; j < 3; j++) {
				if (copyBoard[i][j] == '') {
					copyBoard[i][j] = 'O';
					let score = minimax(copyBoard, false);
					copyBoard[i][j] = '';
					if (score > bestScore) {
						bestScore = score;
						move = { i, j };
					}
				}
			}
		}
		return move;
	};
	const minimax = (
		board: string[][],

		isMaximizing: boolean
	) => {
		let result = checkGameState(board);
		if (result !== null) {
			return scores[result];
		}

		if (isMaximizing) {
			let bestScore = -Infinity;
			for (let i = 0; i < 3; i++) {
				for (let j = 0; j < 3; j++) {
					if (board[i][j] == '') {
						board[i][j] = 'O';
						let score = minimax(board, false);
						board[i][j] = '';
						bestScore = Math.max(score, bestScore);
					}
				}
			}
			return bestScore;
		} else {
			let bestScore = Infinity;
			for (let i = 0; i < 3; i++) {
				for (let j = 0; j < 3; j++) {
					if (board[i][j] == '') {
						board[i][j] = 'X';
						let score = minimax(board, true);
						board[i][j] = '';
						bestScore = Math.min(score, bestScore);
					}
				}
			}
			return bestScore;
		}
	};
	watch(activePlayer, () => {
		if (activePlayer.value === 'X' || '') {
			return;
		}
		if (gameState.value) return;
		const { i, j } = bestMove(squares.value, 'O');
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
		<p v-else-if="activePlayer">Active Player is: {{ activePlayer }}</p>
		<div v-else>
			<p>Choose first Player</p>
			<button @click="chooseFirstPlayer(0)">Spieler</button>
			<button @click="chooseFirstPlayer(1)">NPC</button>
		</div>
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
					:disabled="!!cell && !activePlayer"
					:class="{
						'cell-x': cell === 'X',
						'cell-o': cell === 'O',
					}"
					:style="{
						cursor:
							!cell && !!activePlayer ? 'pointer' : 'not-allowed',
					}"
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
