<script setup lang="ts">
import type { Block } from './GameBoard.vue';

defineProps<{ block: Block }>();
const isDev = false;

function getClass(block: Block) {
  //   console.log(block);
  if (block.revealed) {
    return block.mine ? 'bg-red-500/40' : 'bg-gray-200/20';
  }

  return 'bg-gray-400/30';
}
</script>

<template>
  <div
    border="1 white"
    w-8
    h-8
    flex
    items-center
    justify-center
    hover="bg-light-500/50"
    cursor-pointer
    :class="getClass(block)"
  >
    <template v-if="block.revealed || isDev">
      <div v-if="block.mine">
        <i-noto-bomb text-lg />
      </div>
      <div v-else>{{ block.adjacent > 0 ? block.adjacent : '' }}</div>
    </template>
  </div>
</template>
