<script setup lang="ts">
  import { ref } from 'vue';
  import isEqual from 'lodash.isequal';

  function generateSolvablePuzzle() {
    const array = Array.from({ length: 16 }, (_, i) => i + 1);
    for (let i = array.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [array[i], array[j]] = [array[j], array[i]];
    }
    while (!isSolvable(array)) {
      for (let i = array.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]]; // Swap elements
      }
    }
    return array;
  }

  function isSolvable(puzzle: number[]) {
    let count = 0;
    for (let i = 0; i < puzzle.length; i++) {
      for (let j = i + 1; j < puzzle.length; j++) {
        if (puzzle[i] !== 0 && puzzle[j] !== 0 && puzzle[i] > puzzle[j]) {
          count++;
        }
      }
    }
    return count % 2 === 0;
  }

  const finished = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16];
  const boxes = ref(generateSolvablePuzzle());

  const selectedBox = ref(0);
  const styles = (index: number, box: number) =>
    `${
      selectedBox.value === index ? 'outline outline-offset-2 outline-1 outline-white' : ''
    } size-52 rounded-lg flex items-center justify-center shadow-lg text-xl ${
      box === 16 ? 'bg-black' : box === index + 1 ? 'bg-white' : 'bg-blue-500'
    }`;

  const checkAdjacentElements = (index: number, target: number) => {
    const row = Math.floor(target / 4);
    const col = target % 4;

    const adjacentIndices = [
      row > 0 ? (row - 1) * 4 + col : -1,
      row < 3 ? (row + 1) * 4 + col : -1,
      col > 0 ? row * 4 + (col - 1) : -1,
      col < 3 ? row * 4 + (col + 1) : -1,
    ];

    return adjacentIndices.includes(index);
  };

  const onPress = (e: KeyboardEvent) => {
    if (e.key === 'x') {
      const newArr = [...boxes.value];
      const emptyBox = boxes.value.findIndex((v: number) => v === 16);
      if (emptyBox === selectedBox.value) return;

      if (!checkAdjacentElements(selectedBox.value, emptyBox)) return;

      newArr[selectedBox.value] = boxes.value[emptyBox];
      newArr[emptyBox] = boxes.value[selectedBox.value];

      return (boxes.value = newArr);
    }

    if (e.key === 'ArrowDown' && !(selectedBox.value + 4 > 15)) {
      return (selectedBox.value += 4);
    }
    if (e.key === 'ArrowUp' && selectedBox.value - 4 > -1) {
      return (selectedBox.value -= 4);
    }

    if (e.key === 'ArrowRight') {
      if ((selectedBox.value + 1) % 4 === 0) return;
      return (selectedBox.value += 1);
    }
    if (e.key === 'ArrowLeft') {
      if ((selectedBox.value + 1) % 4 === 1) return;
      return (selectedBox.value -= 1);
    }
  };

  const isFinished = () => {
    if (isEqual(boxes.value, finished)) return 'Finished';
    return 'not Finished';
  };

  window.addEventListener('keyup', onPress);
</script>

<template>
  <div class="bg-black">
    <div class="container min-h-svh mx-auto flex justify-center">
      <div class="my-auto">
        <div class="text-white text-8xl text-center p-2">{{ isFinished() }}</div>
        <div class="grid grid-cols-4 gap-1">
          <div v-for="(box, index) in boxes">
            <div :class="styles(index, box)">
              {{ box }}
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped></style>
