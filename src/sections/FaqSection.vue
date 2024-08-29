<script setup>
import { ref, onMounted, watch } from 'vue';

const fixtures = ref([]);
const selectedDate = ref(new Date().toISOString().split('T')[0]);

const fetchFixtures = async (date) => {
  try {
    const response = await fetch(`https://v3.football.api-sports.io/fixtures?league=2&season=2024&date=${date}`, {
      headers: {
        'x-rapidapi-host': 'v3.football.api-sports.io',
        'x-rapidapi-key': '020ca0175ae97a08189647959cd302b3'
      }
    });
    const data = await response.json();
    fixtures.value = await Promise.all(data.response.map(async (fixture) => {
      const weather = await fetchWeather(fixture.fixture.venue.city);
      return { ...fixture, weather };
    }));
  } catch (error) {
    console.error('Error fetching fixtures:', error);
  }
};

const fetchWeather = async (city) => {
  try {
    // Primero, obtenemos las coordenadas de la ciudad
    const geoResponse = await fetch(`http://api.openweathermap.org/geo/1.0/direct?q=${city}&limit=1&appid=47fc8bb2fff633cbbf87b611532ec8c4`);
    const geoData = await geoResponse.json();
    
    if (geoData.length === 0) {
      throw new Error('City not found');
    }
    
    const { lat, lon } = geoData[0];
    
    // Luego, obtenemos el clima para esas coordenadas
    const weatherResponse = await fetch(`https://api.openweathermap.org/data/2.5/weather?lat=${lat}&lon=${lon}&appid=47fc8bb2fff633cbbf87b611532ec8c4&units=metric`);
    const weatherData = await weatherResponse.json();
    
    return weatherData;
  } catch (error) {
    console.error('Error fetching weather:', error);
    return null;
  }
};

onMounted(() => {
  fetchFixtures(selectedDate.value);
});

watch(selectedDate, (newDate) => {
  fetchFixtures(newDate);
});
</script>

<template>
  <div class="container mx-auto px-5 md:w-4/5">
    <h2 class="text-3xl md:text-4xl font-theme-heading font-medium text-center my-8">Champions League Fixtures</h2>
    
    <div class="mb-6">
      <label for="date-select" class="block mb-2 text-sm font-medium text-gray-900">Select Date:</label>
      <input 
        type="date" 
        id="date-select" 
        v-model="selectedDate" 
        class="bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-blue-500 focus:border-blue-500 block w-full p-2.5"
      >
    </div>

    <div v-if="fixtures.length === 0" class="text-center text-gray-500 my-8">
      No fixtures found for the selected date.
    </div>

    <div v-else class="grid md:grid-cols-2 gap-6">
      <div v-for="fixture in fixtures" :key="fixture.fixture.id" class="bg-white rounded-lg shadow-md p-6">
        <div class="flex justify-between items-center mb-4">
          <div class="text-lg font-semibold">{{ fixture.league.name }}</div>
          <div class="text-sm text-gray-500">{{ new Date(fixture.fixture.date).toLocaleString() }}</div>
        </div>
        <div class="flex justify-between items-center mb-4">
          <div class="flex items-center">
            <img :src="fixture.teams.home.logo" alt="Home Team" class="w-8 h-8 mr-2">
            <span>{{ fixture.teams.home.name }}</span>
          </div>
          <div class="text-xl font-bold">
            {{ fixture.goals.home }} - {{ fixture.goals.away }}
          </div>
          <div class="flex items-center">
            <span>{{ fixture.teams.away.name }}</span>
            <img :src="fixture.teams.away.logo" alt="Away Team" class="w-8 h-8 ml-2">
          </div>
        </div>
        <div v-if="fixture.weather" class="mt-4 p-4 bg-gray-100 rounded-lg">
          <div class="text-center font-semibold mb-2">Weather in {{ fixture.fixture.venue.city }}</div>
          <div class="flex justify-around">
            <div>
              <i class="fas fa-temperature-high"></i> {{ Math.round(fixture.weather.main.temp) }}°C
            </div>
            <div>
              <i class="fas fa-tint"></i> {{ fixture.weather.main.humidity }}%
            </div>
            <div>
              <i class="fas fa-wind"></i> {{ Math.round(fixture.weather.wind.speed * 3.6) }} km/h
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>