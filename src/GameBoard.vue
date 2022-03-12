<script setup lang="ts">
import { reactive, computed } from 'vue';
import Cell from './Cell.vue';

export interface Block {
  x: number;
  y: number;
  mine: boolean;
  flag: boolean;
  adjacent: number;
  revealed: boolean;
}

type Board = Block[][];

interface GameState {
  board: Board;
  width: number;
  height: number;
  mines: number;
  flags: number;
  status: 'playing' | 'won' | 'lost';
}

function init(width: number, height: number, mines?: number): GameState {
  const number = mines || Math.floor((width * height) / 10);
  return {
    board: createBoard(width, height),
    width: width,
    height: height,
    mines: number,
    flags: number,
    status: 'playing',
  };
}

const state = reactive(init(15, 15));
setupMines(state.board, state.mines);
setupAdjacents(state.board);
const board = computed(() => state.board);

function createBoard(w: number, h: number): Board {
  const board = Array.from({ length: h }, (_, y) =>
    Array.from({ length: w }, (_, x) => ({
      x,
      y,
      mine: false,
      flag: false,
      adjacent: 0,
      revealed: false,
    }))
  );
  return board;
}

function setupMines(board: Board, mines: number) {
  while (mines > 0) {
    const x = Math.floor(Math.random() * board.length);
    const y = Math.floor(Math.random() * board[0].length);
    if (!board[x][y].mine) {
      board[x][y].mine = true;
      mines--;
    }
  }
}

function expandZeros(block: Block) {
  if (block.adjacent > 0) return;

  getAdjacents(block).forEach((s) => {
    if (s && s.revealed === false) {
      s.revealed = true;
      expandZeros(s);
    }
  });
}

function getAdjacents(block: Block) {
  const { x, y } = block;
  const adjacents: (Block | null)[] = [
    { x: x - 1, y: y - 1 },
    { x: x - 1, y: y },
    { x: x - 1, y: y + 1 },
    { x: x, y: y - 1 },
    { x: x, y: y + 1 },
    { x: x + 1, y: y - 1 },
    { x: x + 1, y: y },
    { x: x + 1, y: y + 1 },
  ]
    .map(({ x, y }) => {
      if (x < 0 || y < 0 || x >= state.width || y >= state.height) {
        return null;
      }

      return state.board[y][x];
    })
    .filter((block) => block !== null);
  return adjacents;
}

function setupAdjacents(board: Board) {
  board.forEach((row) => {
    row.forEach((block) => {
      if (block.mine) {
        return;
      }

      const adjacents = getAdjacents(block);
      block.adjacent = adjacents.filter((block) => block?.mine).length;
    });
  });
}

function onClick(block: Block) {
  if (block.revealed) return;
  block.revealed = true;

  if (block.mine) {
    state.board.forEach((row) => {
      row.forEach((block) => {
        block.revealed = true;
      });
    });
    state.status = 'lost';
    return;
  }
  expandZeros(block);
  checkGameStatus(state);
}

function onRightClick(block: Block) {
  console.log(state.flags);
  if (state.status !== 'playing') return;
  if (block.revealed) return;
  if (state.flags === 0) return;
  if (block.flag) {
    block.flag = false;
    state.flags++;
  } else {
    block.flag = true;
    state.flags--;
  }
  checkGameStatus(state);
}

function restart() {
  state.board = createBoard(state.width, state.height);
  setupMines(state.board, state.mines);
  setupAdjacents(state.board);
  state.status = 'playing';
}

function checkGameStatus(state: GameState) {
  const { board } = state;
  const flaged = board.flat().filter((block) => block.flag);
  const revealed = board.flat().filter((b) => b.revealed);
  if (flaged.length + revealed.length === board.length * board.length) {
    flaged.every((block) => block.mine)
      ? (state.status = 'won')
      : (state.status = 'lost');
  }
}
</script>

<template>
  <div flex justify-center bg="#363C4F" min-h-screen flex-col items-center>
    <h2 color-white>Minesweeper</h2>

    <div min-w-20 bg-white min-h-20 flex flex-col border="8 dark" relative>
      <div
        v-if="state.status === 'lost'"
        absolute
        top-0
        right-0
        left-0
        bottom-0
        flex
        justify-center
        items-center
        flex-col
        bg="#292D3E/70"
        text-white
        @click="restart()"
      >
        <h1>You Lost!</h1>
        <p>Click to restart</p>
      </div>
      <div
        v-if="state.status === 'won'"
        absolute
        top-0
        right-0
        left-0
        bottom-0
        flex
        justify-center
        items-center
        flex-col
        bg="#7E57C2/70"
        color="#FFCB6B"
        @click="restart()"
      >
        <h1>You Won!</h1>
        <p>Click to restart</p>
      </div>

      <div v-for="(col, y) in board" :key="y" flex flex-row>
        <Cell
          v-for="(block, x) in col"
          :key="x"
          :block="block"
          @click="onClick(block)"
          @contextmenu.prevent="onRightClick(block)"
        />
      </div>
    </div>
    <div flex flex-row text-white>
      <p w-20 text-center>mines: {{ state.mines }}</p>
      <p w-20 text-center>flags: {{ state.flags }}</p>
    </div>
  </div>
</template>
