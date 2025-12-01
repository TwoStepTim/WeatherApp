<script setup>
import { ref, computed } from 'vue'
import BorderLine from './BorderLine.vue'
import WeatherForecastDay from './WeatherForecastDay.vue'
import WeatherInfo from './WeatherInfo.vue'
import HourlyChart from './HourlyChart.vue'

const { place } = defineProps({
  place: Object
})



const emit = defineEmits(['delete-place'])

const showDetail = ref(false)
const showHourly = ref(false)

// Only show remaining hours starting NOW
const upcomingHours = computed(() => {
  if (
    !place ||
    !place.forecast ||
    !place.forecast.forecastday ||
    !place.forecast.forecastday[0] ||
    !place.forecast.forecastday[0].hour
  ) {
    return []
  }

  const hours = place.forecast.forecastday[0].hour
  const currentHour = new Date(place.location.localtime).getHours()

  return hours.filter(h => {
    const hourNum = parseInt(h.time.split(" ")[1].split(":")[0])
    return hourNum >= currentHour
  })
})

const removePlace = (placeName) => {
  emit('delete-place', placeName)
  showDetail.value = false
}
</script>

<template>
  <div
    :class="place.current.is_day === 1 ? 'bg-day' : 'bg-night'"
    class="text-white p-10 rounded-lg shadow-lg gap-6 mb-6 relative overflow-hidden"
  >
    <!-- Location & time -->
    <div class="mb-2 flex justify-between items-center">
      <div class="flex items-center justify-center gap-2">
        <i class="fa-solid fa-location-dot"></i>
        <h1 class="text-3xl">{{ place.location.name }}</h1>
      </div>
      <div class="flex items-center justify-center gap-2">
        <i class="fa-solid fa-clock"></i>
        <h1 class="text-3xl">
          {{ new Date(place.location.localtime).getHours() }}:{{
            new Date(place.location.localtime).getMinutes().toString().padStart(2, "0")
          }}
        </h1>
      </div>
    </div>

    <!-- current weather -->
    <div class="text-center flex-1">
      <img :src="place.current.condition.icon" alt="icon" width="200" class="mx-auto -mb-10" />
      <h1 class="text-9xl mb-2 -mr-4">{{ Math.round(place.current.temp_f) }}&deg;</h1>
      <p class="text-2xl">{{ place.current.condition.text }}</p>
    </div>

    <BorderLine />

    <!-- Forecast Switch -->
    <div class="flex gap-4 justify-center my-4">
      <button
        @click="showHourly = false"
        class="px-4 py-2 rounded-md"
        :class="!showHourly ? 'bg-blue-600 text-white' : 'bg-gray-300'"
      >
        5 Day
      </button>

      <button
        @click="showHourly = true"
        class="px-4 py-2 rounded-md"
        :class="showHourly ? 'bg-blue-600 text-white' : 'bg-gray-300'"
      >
        Hourly
      </button>
    </div>

    <!-- 5 DAY FORECAST -->
    <div v-if="!showHourly">
      <div v-for="(day, idx) in place.forecast.forecastday" :key="idx">
        <WeatherForecastDay :day="day" />
      </div>
    </div>

    <!-- HOURLY FORECAST -->
    <div v-else>
      <!-- Hourly Chart -->
      <HourlyChart :hours="upcomingHours" />

      <!-- Hour Cards -->
      <div class="grid grid-cols-2 md:grid-cols-4 gap-4 mt-6">
        <div
          v-for="(hour, idx) in upcomingHours"
          :key="idx"
          class="p-4 bg-white/20 rounded-lg text-center backdrop-blur"
        >
          <p class="font-bold">
            {{ hour.time.split(' ')[1] }}
          </p>
          <img :src="hour.condition.icon" class="w-12 mx-auto" />
          <p>{{ Math.round(hour.temp_f) }}°F</p>
          <p class="text-sm">Rain: {{ hour.precip_in }} in</p>
        </div>
      </div>
    </div>

    <!-- info -->
    <Transition name="fade">
      <div v-show="showDetail">
        <WeatherInfo
          :place="place"
          @close-info="showDetail = false"
          @remove-place="removePlace(place.location.name)"
        />
      </div>
    </Transition>

    <!-- forecast btn -->
    <div class="flex justify-end items-center gap-1 mt-10">
      <button @click="showDetail = true">
        More <i class="fa-solid fa-arrow-right text-sm -mb-px"></i>
      </button>
    </div>
  </div>
</template>

<style scoped>
.bg-day {
  background-color: #7dd3fc;
  background-image: linear-gradient(135deg, #93c5fd 0%, #fde68a 100%);
}

.bg-night {
  background-color: #1e3a8a;
  background-image: linear-gradient(135deg, #312e81 0%, #1e3a8a 100%);
}


.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
