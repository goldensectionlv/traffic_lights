<template>
  <div>
    <div
      class="light"
      :class="{
        'lightActive': isActive,
        'blink': blinkActive
      }"
      :style="{backgroundColor: lightColor}"
    >
      {{ countdown }}
    </div>
  </div>
</template>

<script>
export default {
  name: "Light",
  props: {
    lightColor: {
      type: String,
      default: ''
    },
    isActive: {
      type: Boolean,
      default: false
    },
    countdown: {
      type: Number,
      default: null
    },
    blinkActive: {
      type: Boolean,
      default: false
    }
  }
}
</script>

<style lang="scss" scoped>
.light {
  width: 100px;
  height: 100px;
  border-radius: 50%;
  opacity: 0.1;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
}

.lightActive {
  opacity: 1;
}

.blink {
  -webkit-animation: blink 1s linear infinite;
  animation: blink 1s linear infinite;
}

@-webkit-keyframes blink {
  100% {
    opacity: 0.3
  }
}

@keyframes blink {
  100% {
    opacity: 0.3
  }
}

</style>