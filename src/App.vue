<script setup>
import { ref } from "vue";
import { uid } from "uid";
import BaseLine from "./components/BaseLine.vue";

const blockSize = ref(25);
const blocks = ref([]);
const emptySpace = ref(100);
const errorMessage = ref("");

const addBlock = () => {
  errorMessage.value = "";
  if (!blockSize.value || blockSize.value > emptySpace.value) {
    errorMessage.value =
      emptySpace.value === 0 ? "No space" : "Max value " + emptySpace.value;
    return;
  }

  let newBlock = {
    id: uid(),
    width: blockSize.value,
    color: getRandomColor(),
    isSelected: false,
    removed: false,
  };

  const emptyBlockIndexes = blocks.value.reduce((list, block, i) => {
    if (block.removed) {
      list.push(i);
    }
    return list;
  }, []);

  if (emptyBlockIndexes.length) {
    let indexesCount = emptyBlockIndexes.length;
    emptyBlockIndexes.forEach((index) => {
      if (!newBlock) return;
      indexesCount -= 1;
      const emptyBlockWidth = blocks.value[index].width;
      if (emptyBlockWidth >= newBlock.width) {
        blocks.value[index] = newBlock;
        if (emptyBlockWidth - newBlock.width > 0) {
          const emptyNewBlock = {
            id: uid(),
            width: emptyBlockWidth - newBlock.width,
            isSelected: false,
            color: getRandomColor(),
            removed: true,
          };
          blocks.value.splice(index + 1, 0, emptyNewBlock);
        }
        newBlock = null;
      } else {
        const extraNewBlock = {
          id: uid(),
          width: emptyBlockWidth,
          color: newBlock.color,
          isSelected: false,
          removed: false,
        };
        newBlock.width = newBlock.width - emptyBlockWidth;
        blocks.value[index] = extraNewBlock;
        if (indexesCount <= 0) {
          blocks.value.push(newBlock);
        }
      }
    });
  } else {
    blocks.value.push(newBlock);
  }
  emptySpace.value = 100;

  blocks.value = blocks.value.map((block) => {
    const currentBlock = { ...block, left: Math.abs(emptySpace.value - 100) };
    emptySpace.value -= block.width;
    return currentBlock;
  });

  blocks.value.forEach((block) => {
    if (block.removed) {
      emptySpace.value += block.width;
    }
  });
};

const removeBlock = (block) => {
  const blockIndexes = getIndexListByColor(block.color);

  blockIndexes.forEach((index) => {
    blocks.value[index].removed = true;
    emptySpace.value += blocks.value[index].width;
  });

  blocks.value = mergeRemovedBlocks([...blocks.value]);
};

const selectBlockToogler = (block) => {
  const blockIndexes = getIndexListByColor(block.color);

  blockIndexes.forEach((index) => {
    blocks.value[index].isSelected = !blocks.value[index].isSelected;
  });
};

const getIndexListByColor = (color) => {
  return blocks.value.reduce((list, item, i) => {
    if (item.color === color) {
      list.push(i);
    }
    return list;
  }, []);
};

const mergeRemovedBlocks = (blocks) => {
  const result = [];

  let i = 0;
  while (i < blocks.length) {
    const current = blocks[i];

    if (current.removed) {
      let sumWidth = current.width;
      let j = i + 1;

      while (j < blocks.length && blocks[j].removed) {
        sumWidth += blocks[j].width;
        j++;
      }

      const mergedBlock = { id: uid(), width: sumWidth, removed: true };
      result.push(mergedBlock);

      i = j;
    } else {
      result.push(current);
      i++;
    }
  }

  return result;
};

const getRandomHexValue = () => {
  return Math.floor(Math.random() * 256)
    .toString(16)
    .padStart(2, "0");
};

const getRandomColor = () => {
  const red = getRandomHexValue();
  const green = getRandomHexValue();
  const blue = getRandomHexValue();
  return `#${red}${green}${blue}`;
};
</script>

<template>
  <div class="container">
    <base-line
      :blocks="blocks"
      @remove-block="removeBlock"
      class="line"
      @select-block="selectBlockToogler"
    />
    <form @submit.prevent="addBlock">
      <input v-model.number="blockSize" type="number" :min="1" />
      <button type="submit">Add block</button>
    </form>
    <div v-show="errorMessage" class="error">{{ errorMessage }}</div>
  </div>
</template>

<style scoped>
.container {
  width: 100%;
}

.line {
  margin-bottom: 24px;
}

.error {
  color: red;
}

input {
  margin-right: 24px;
}
</style>
