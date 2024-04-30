<script setup lang="ts">
	import { ref, computed } from 'vue';
	const players = ['X', 'O'] as const;
	const activePlayer = ref('X');
	const squares = ref([
		['', '', ''],
		['', '', ''],
		['', '', ''],
	]);
	const tie = computed<Boolean>(() => checkTie());
	const winner = computed<String | null>(() => checkWinner());
	const checkWinner = () => {
		let winningPlayer = null;
		players.forEach((player) => {
			for (let i = 0; i < 3; i++) {
				if (squares.value[i].every((cell) => cell === player))
					winningPlayer = player;
				if (squares.value.every((row) => row[i] === player))
					winningPlayer = player;
			}

			if (squares.value[1][1] === player) {
				if (
					squares.value[0][0] === player &&
					squares.value[2][2] === player
				)
					winningPlayer = player;
				if (
					squares.value[0][2] === player &&
					squares.value[2][0] === player
				)
					winningPlayer = player;
			}
		});
		return winningPlayer;
	};
	const checkTie = () => {
		let isTie = false;
		if (
			!winner.value &&
			squares.value.every((row) => row.every((cell) => cell))
		) {
			isTie = true;
		}
		return isTie;
	};
	const move = (x: number, y: number) => {
		if (winner.value) return;
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
</script>
<template>
	<div>
		<h1>Tic Tac Toe</h1>
		<p v-if="winner">The winner is: {{ winner }}</p>
		<p v-else-if="tie">It's a tie!</p>
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
