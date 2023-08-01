<script setup>
import BaseBlock from "./BaseBlock.vue";

defineEmits({
  "remove-block": (value) => typeof value === "object",
  "select-block": (value) => typeof value === "object",
});

const props = defineProps({
  blocks: {
    type: Array,
    default() {
      return [];
    },
  },
});
</script>

<template>
  <div class="line">
    <BaseBlock
      v-for="block in blocks"
      :key="block.id"
      :block="block"
      @remove-block="$emit('remove-block', block)"
      @select-block="$emit('select-block', block)"
    />
  </div>
</template>

<style scoped>
.line {
  position: relative;
  width: 100%;
  height: 50px;
  border: 2px solid gray;
}
</style>
