<script setup>
import { onMounted, ref, watch } from 'vue'
import { Chart } from 'chart.js/auto'

const props = defineProps({
  hours: Array // list of 24 hourly objects
})

const canvasRef = ref(null)
let chartInstance = null

const createChart = () => {
  if (!canvasRef.value) return

  if (chartInstance) {
    chartInstance.destroy()
  }

  const labels = props.hours.map(h => h.time.split(' ')[1])
  const temps = props.hours.map(h => h.temp_f)
  const rain = props.hours.map(h => h.precip_in)

  chartInstance = new Chart(canvasRef.value, {
    type: 'line',
    data: {
      labels,
      datasets: [
        {
          label: 'Temperature (F)',
          data: temps,
          borderColor: '#3b82f6',
          backgroundColor: 'rgba(59,130,246,0.3)',
          tension: 0.25
        },
        {
          label: 'Rain (in)',
          data: rain,
          type: 'bar',
          backgroundColor: 'rgba(16,185,129,0.5)'
        }
      ]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false
    }
  })
}

onMounted(createChart)
watch(() => props.hours, createChart)
</script>

<template>
  <div class="w-full h-64">
    <canvas ref="canvasRef"></canvas>
  </div>
</template>
