<script setup>
import { ref, computed } from 'vue'

const dollar = new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD' })
const number = new Intl.NumberFormat('en-US', { style: 'decimal' })

const revenue = ref(200_000)
const installations = ref(200_000)
const mode = ref('personal')
const emergingMarket = ref(false)

const revenueOverTheThreshold = computed(() => {
  if (mode.value === 'personal') {
    return revenue.value - 200_000
  } else if (mode.value === 'plus') {
    return revenue.value - 200_000
  } else if (mode.value === 'pro') {
    return revenue.value - 1_000_000
  } else if (mode.value === 'enterprise') {
    return revenue.value - 1_000_000
  }
})

const installationsOverTheThreshold = computed(() => {
  if (mode.value === 'personal') {
    return installations.value - 200_000
  } else if (mode.value === 'plus') {
    return installations.value - 200_000
  } else if (mode.value === 'pro') {
    return installations.value - 1_000_000
  } else if (mode.value === 'enterprise') {
    return installations.value - 1_000_000
  }
})

const costPerInstall = computed(() => {
  if (mode.value === 'personal' || mode.value === 'plus') {
    if (emergingMarket.value) {
      return 0.02
    }
    return 0.2
  } else if (mode.value === 'pro') {
    if (emergingMarket.value) {
      return 0.01
    }
    if (installationsOverTheThreshold.value > 0 && installationsOverTheThreshold.value <= 100_000) {
      return 0.15
    } else if (installationsOverTheThreshold.value > 100_000 && installationsOverTheThreshold.value <= 500_000) {
      return 0.075
    } else if (installationsOverTheThreshold.value > 500_000 && installationsOverTheThreshold.value <= 1_000_000) {
      return 0.03
    }
    return 0.02    
  } else if (mode.value === 'enterprise') {
    if (emergingMarket.value) {
      return 0.005
    }
    if (installationsOverTheThreshold.value > 0 && installationsOverTheThreshold.value <= 100_000) {
      return 0.125
    } else if (installationsOverTheThreshold.value > 100_000 && installationsOverTheThreshold.value <= 500_000) {
      return 0.06
    } else if (installationsOverTheThreshold.value > 500_000 && installationsOverTheThreshold.value <= 1_000_000) {
      return 0.02
    }
    return 0.01
  }
  return 0
})

const toPay = computed(() => {
  if (revenueOverTheThreshold.value < 0 || installationsOverTheThreshold.value <= 0) {
    return 0
  }
  return costPerInstall.value * installationsOverTheThreshold.value
})

const toPayPercentage = computed(() => {
  if (revenueOverTheThreshold.value < 0 || installationsOverTheThreshold.value <= 0) {
    return 0
  }
  return (toPay.value / revenue.value) * 100
})

const costPerInstallFormatted = computed(() => dollar.format(costPerInstall.value))
const toPayFormatted = computed(() => dollar.format(toPay.value))
const toPayPercentageFormatted = computed(() => `${toPayPercentage.value.toFixed(2)}%`)
const installationsOverTheThresholdFormatted = computed(() => number.format(installationsOverTheThreshold.value < 0 ? 0 : installationsOverTheThreshold.value))
</script>

<template>
  <div id="app">
    <form class="form">
      <div class="form-group">
        <label for="installations">Installations</label>
        <input id="installations" type="number" v-model="installations" />
      </div>
      <div class="form-group">
        <label for="revenue">Revenue</label>
        <input id="revenue" type="number" v-model="revenue" />
      </div>
      <div class="form-group modes">
        <div class="form-group">
          <input id="personal" name="mode" type="radio" value="personal" v-model="mode" checked />
          <label for="personal">Unity Personal</label>
        </div>
        <div class="form-group">
          <input id="plus" name="mode" type="radio" value="plus" v-model="mode" />
          <label for="plus">Unity Plus</label>
        </div>
        <div class="form-group">
          <input id="pro" name="mode" type="radio" value="pro" v-model="mode" />
          <label for="pro">Unity Pro</label>
        </div>
        <div class="form-group">
          <input id="enterprise" name="mode" type="radio" value="enterprise" v-model="mode" />
          <label for="enterprise">Unity Enterprise</label>
        </div>
      </div>
      <div class="form-group">
        <input id="emergingMarket" type="checkbox" v-model="emergingMarket" />
        <label for="emergingMarket">Emerging Market</label>
      </div>
      <details>
        <summary>What is an Emerging Market?</summary>
        Standard fees apply to app installs in the United States, Australia, Austria, Belgium, Canada, Denmark, Finland, France, Germany, Ireland, Japan, Netherlands, New Zealand, Norway, Sweden, Switzerland, South Korea, and the United Kingdom. Emerging market fees apply to app installs in all other countries.
      </details>
    </form>
    <div class="to-pay">
      <div class="operation">{{ installationsOverTheThresholdFormatted }} &times; {{ costPerInstallFormatted }} = {{ toPayFormatted }}</div>
      <div class="explanation">
        That's the <span class="percentage">{{ toPayPercentageFormatted }}</span> of your revenue
      </div>
    </div>
  </div>
</template>

<style scoped>
#app {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.form, .form-group {
  display: flex;
  gap: 0.5rem;
}

.modes {
  gap: 2rem;
}

.form {
  flex-direction: column;
}
.form-group {
  flex-direction: row;
}

.to-pay {
  font-size: 1.5rem;
}

details {
  max-width: 32rem;
  display: flex;
  flex-direction: row;
  justify-content: start;
  align-items: center;
}

.operation {
  font-size: 150%;
}

.percentage {
  font-weight: bold;
}
</style>
