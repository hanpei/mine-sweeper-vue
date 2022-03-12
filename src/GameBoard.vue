<script setup lang="ts">
import { reactive, computed, nextTick } from 'vue';
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
  status: 'playing' | 'won' | 'lost';
}

const state = reactive(init(15, 15));
calculateAdjacent(state.board);
const board = computed(() => state.board);

function init(width: number, height: number): GameState {
  return {
    board: createBoard(width, height),
    width: width,
    height: height,
    status: 'playing',
  };
}

function createBoard(w: number, h: number): Board {
  const board = Array.from({ length: h }, (_, y) =>
    Array.from({ length: w }, (_, x) => ({
      x,
      y,
      mine: Math.random() < 0.1,
      flag: false,
      adjacent: 0,
      revealed: false,
    }))
  );
  return board;
}

function expandZeroe(block: Block) {
  if (block.adjacent > 0) return;

  getAdjacents(block).forEach((s) => {
    if (s && s.revealed === false) {
      s.revealed = true;
      expandZeroe(s);
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

function calculateAdjacent(board: Board) {
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
    nextTick(() => {
      state.status = 'lost';
    });

    return;
  }

  expandZeroe(block);
}

function restart() {
  state.board = createBoard(state.width, state.height);
  calculateAdjacent(state.board);
  state.status = 'playing';
}
</script>

<template>
  <div flex justify-center bg="#363C4F" min-h-screen flex-col items-center>
    <h2 color-white>Mine Sweeper</h2>
    <div min-w-50 bg-white min-h-50 flex flex-col border="8 dark" relative>
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
      <div v-for="(col, y) in board" :key="y" flex flex-row>
        <Cell
          v-for="(block, x) in col"
          :key="x"
          :block="block"
          @click="onClick(block)"
        />
      </div>
    </div>
  </div>
</template>
