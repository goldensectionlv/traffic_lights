<template>
  <div class="main">
    <div class="lights">
      <BaseLight
        v-for="light in trafficLights"
        :key="light.color"
        :light-color="light.color"
        :is-active="light.color === $route.params.color"
        :countdown="light.color === $route.params.color ? light.duration + 1 - counter : null"
        :blink-active="light.color === $route.params.color && light.duration + 1 - counter < 3"
      />
    </div>
  </div>
</template>

<script>
import BaseLight from "@/components/BaseLight";

export default {
  name: 'Home',
  components: { BaseLight },
  data() {
    return {
      trafficLights: [
        // кол-во цветов и положение предупреждающего сигнала в массиве не влияют на работоспособность
        {color: 'yellow', duration: 2, isWarningColor: true},
        {color: 'red', duration: 3},
        {color: 'green', duration: 4},
      ],
      interval: null /* Переменная для очистки интервала в цикле beforeDestroy */,
      counter: 1, /* Переменная для сверки с длинной сигнала */
      activeLight: {}, /* Актуальный сигнал (не учитывает предупреждающий сигнал) */
      warningLight: {} /* Отдельная ссылка для быстрого доступа */,
      warningActive: false /* Триггер для периодического включения предупреждающего сигнала */
    }
  },
  created() {
    // Редирект в том случае, если присутствует хотя бы один элемент в списке цветов светофора
    if (this.$route.path === '/' && this.trafficLights.length)
      this.$router.push(this.trafficLights[0].color)
    // Поиск цвета предупреждения в массиве
    this.warningLight = this.trafficLights.find(light => light.isWarningColor)
    // Авто включение тригера на переключение на следующий цвет, т.к. иначе будет двойное переключение на предупреждающий цвет
    if (this.warningLight.color === this.$route.params.color)
      this.warningActive = true

    this.activeLight = JSON.parse(JSON.stringify(this.trafficLights.find(light => light.color === this.$route.params.color)))

    this.countdown()
  },
  beforeDestroy() {
    clearInterval(this.interval)
  },
  methods: {
    // управляющие функции
    countdown() {
      this.interval = setInterval(() => {
        this.counter++
        if (this.activeLight.duration < this.counter) {
          this.counter = 1
          this.warningActive ? this.switchLight() : this.enableWarningSignal()
        }
      }, 1000)
    },
    switchLight() {
      const index = this.trafficLights.findIndex(light => light.color === this.activeLight.color)

      if (this.trafficLights.length === index + 1) { /* если конец списка */
        this.restartCycle()
      } else {
        const isNextLightWarning = this.trafficLights[index + 1].color === this.warningLight.color
        isNextLightWarning ? this.jumpOverWarningSignal(index) : this.goToNextColor(index)
      }
      this.warningActive = false
    },
    // побочные функции
    enableWarningSignal() {
      this.$router.push(this.warningLight.color)
      this.activeLight.duration = this.warningLight.duration
      this.warningActive = true
      console.log('enable warning signal')
    },

    restartCycle() {
      // Исключение, если предупреждающий свет стоит первым в массиве
      const isFirstInArray = (this.trafficLights.findIndex(light => light.color === this.warningLight.color)) === 0
      const index = isFirstInArray ? 1 : 0
      this.switchColor(index)
      console.log('array end, refreshed')
    },
    jumpOverWarningSignal(index) {
      // Исключение, если предупреждающий свет стоит последним в массиве
      const isLastInArray = this.trafficLights.length === this.trafficLights.findIndex(light => light.color === this.warningLight.color) + 1
      isLastInArray ? this.restartCycle() : this.switchColor(index + 2)
      console.log('jump over warning signal, avoid double enable of warning signal')
    },
    goToNextColor(index) {
      this.switchColor(index + 1)
      console.log('go to next color in array')
    },
    switchColor(index) {
      const nextColor = this.trafficLights[index].color
      this.$router.push(nextColor)
      this.activeLight = JSON.parse(JSON.stringify(this.trafficLights[index]))
    }
  }
}
</script>

<style>
.main {
  width: 100vw;
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: #42b983;
}

.lights {
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-direction: column;
  padding: 15px 0 15px 0;
  min-height: 360px;
  width: 150px;
  border-radius: 9px;
  background-color: #2D3142;
}
</style>
