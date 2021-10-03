<template>
  <button class="button">
    <span v-if="active">{{ pressedText }}</span>
    <span v-else>{{ unpressedText }}</span>
  </button>
</template>

<script>
export default {
  name: "BaseButton",
  props: {
    unpressedText: {
      type: String,
      default: ''
    },
    pressedText: {
      type: String,
      default: ''
    },
    active: {
      type: Boolean,
      default: false
    }
  }
}
</script>

<style scoped>
.button {
  padding: 9px;
  border-radius: 9px;
  outline: none;
  border: none;
  cursor: pointer;
  margin-top: 9px;
}
</style>