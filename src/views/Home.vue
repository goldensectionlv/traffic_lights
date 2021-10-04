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
    <BaseButton
      unpressed-text="Включить автосохранение?"
      pressed-text="Выключить автосохранение и очистить память?"
      :active="userWantsToStore"
      @click.native="buttonClick"
    />
  </div>
</template>

<script>
import BaseButton from "@/components/BaseButton";
import BaseLight from "@/components/BaseLight";

export default {
  name: 'Home',
  components: { BaseLight, BaseButton },
  data() {
    return {
      trafficLights: [
        {color: 'red', duration: 6},
        {color: 'yellow', duration: 3, isWarningColor: true},
        {color: 'green', duration: 7},
      ],
      interval: null /* Переменная для очистки интервала в цикле beforeDestroy */,
      counter: 1, /* Переменная для сверки с длинной сигнала */
      activeLight: {}, /* Актуальный сигнал */
      warningLight: {} /* Предупреждающий сигнал */,
      warningActive: false, /* Триггер для включения предупреждающего сигнала */
      previousLight: {}, /* Обьект для сохранения состояния прохождения по массиву после предупреждающего сигнала */
      userWantsToStore: false
    }
  },
  created() {
    const pathHomeAndLightsExists = this.$route.path === '/' && this.trafficLights.length
    if (pathHomeAndLightsExists) this.$router.push(this.trafficLights[0].color)

    this.warningLight = this.trafficLights.find(light => light.isWarningColor)

    const currentLightIsWarningType = this.warningLight.color === this.$route.params.color
    if (currentLightIsWarningType) this.warningActive = true

    if (JSON.parse(localStorage.getItem('userWantsToStore'))) {
      this.userWantsToStore = true
      this.localStorageRequest()
    }
    this.activeLight = JSON.parse(JSON.stringify(this.trafficLights.find(light => light.color === this.$route.params.color)))
    this.lightsManager()
  },
  beforeDestroy() {
    clearInterval(this.interval)
  },
  methods: {
    lightsManager() {
      this.interval = setInterval(() => {
        this.counter++
        const lightDurationEnded = this.activeLight.duration < this.counter
        if (lightDurationEnded) {
          this.counter = 1
          this.warningActive ? this.switchLight() : this.enableWarningSignal()
        }
        if (this.userWantsToStore) this.localStorageSet()
      }, 1000)
    },
    switchLight() {
      const index = this.trafficLights.findIndex(light => light.color === this.previousLight.color)

      const isEndOfLightsArray = this.trafficLights.length === index + 1
      if (isEndOfLightsArray) {
        this.restartCycle()
      } else {
        const isNextLightWarning = this.trafficLights[index + 1].color === this.warningLight.color
        isNextLightWarning ? this.jumpOverWarningSignal(index) : this.goToNextColor(index)
      }
      this.warningActive = false
    },
    enableWarningSignal() {
      this.$router.push(this.warningLight.color)
      this.previousLight = this.activeLight
      this.activeLight = this.warningLight
      this.warningActive = true
      console.log('enable warning signal')
    },

    restartCycle() {
      const isFirstInArray = (this.trafficLights.findIndex(light => light.color === this.warningLight.color)) === 0
      const index = isFirstInArray ? 1 : 0
      this.switchColor(index)
      console.log('array ended, cycle restarted')
    },
    jumpOverWarningSignal(index) {
      const isLastInArray = this.trafficLights.length === this.trafficLights.findIndex(light => light.color === this.warningLight.color) + 1
      isLastInArray ? this.restartCycle() : this.switchColor(index + 2)
      console.log('jump over warning signal, avoid double enable of warning signal')
    },
    goToNextColor(index) {
      this.switchColor(index + 1) /* Специально не стал обьединять с функцией switchColor из логических соображений */
      console.log('go to next color in array')
    },
    switchColor(index) {
      const nextColor = this.trafficLights[index].color
      this.$router.push(nextColor)
      this.activeLight = JSON.parse(JSON.stringify(this.trafficLights[index]))
    },
    localStorageRequest() {
      const activeLight = localStorage.getItem('activeLight')
      const counter = localStorage.getItem('counter')
      const previousLight = localStorage.getItem('previousLight')

      if (activeLight && counter && previousLight) {
        this.activeLight = JSON.parse(activeLight)
        this.warningActive = this.activeLight.color === this.warningLight.color;
        this.$router.push(this.activeLight.color).catch(() => {})
        this.counter = JSON.parse(counter)
        this.previousLight = JSON.parse(previousLight)
      }
    },
    localStorageSet() {
      localStorage.setItem('activeLight', JSON.stringify(this.activeLight))
      localStorage.setItem('previousLight', JSON.stringify(this.previousLight))
      localStorage.setItem('counter', JSON.stringify(this.counter))
    },
    buttonClick() {
      if (this.userWantsToStore) localStorage.clear()
      this.userWantsToStore = !this.userWantsToStore
      localStorage.setItem('userWantsToStore', JSON.stringify(this.userWantsToStore))
    }
  }
}
</script>

<style>
.main {
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
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
