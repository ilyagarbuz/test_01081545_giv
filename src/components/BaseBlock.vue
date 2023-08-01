<script setup>
defineProps({
  block: {
    type: Object,
    required: true,
  },
});

defineEmits({
  "remove-block": null,
  "select-block": null,
});
</script>

<template>
  <div
    :style="{
      width: block.width + '%',
      left: block.left + '%',
      'background-color': block.color,
    }"
    :class="{ removed: block.removed, selected: block.isSelected }"
    class="block"
    @dblclick="$emit('remove-block')"
    @click="$emit('select-block')"
  ></div>
</template>

<style scoped>
.block {
  position: absolute;
  height: 100%;
  /* background: blueviolet; */
  border: 2px solid white;
  box-sizing: border-box;
}

.block.removed {
  opacity: 0;
  pointer-events: none;
}

.block.selected {
  opacity: 0.7;
}
</style>
