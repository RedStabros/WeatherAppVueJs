<template>
  <main class="container text-white">
    <div class="pt-4 mb-8 relative">
      <input type="text" placeholder="Search for a city or state" v-model="searchQuery" @input="getSearchResults"
      class="py-2 px-1 w-full bg-transparent border-b focus:border-weather-secondary focus:outline-none 
      focus:shadow-[0px_1px_0_0_#004E71]">
      <ul v-if="openWeatherResults" class="absolute bg-weather-secondary text-white w-full shadow-md py-2 px-1 top-[66px]">
        <p v-if="searchError">Sorry, something went wrong, try again.</p>
        <p v-if="!serverError && openWeatherResults.length === 0">No results match your query, try a different term.</p>
        <template v-else>
        <li v-for="result in openWeatherResults" :key="result.id" class="py-2 cursor-pointer" @click="previewCity(result)">
        {{ result.name }} - {{ result.state }} - {{ result.country }}
        </li>
      </template>
      </ul>
    </div>
    <div class="flex flex-col gap-4">
      <Suspense>
        <CityList/>
        <template #fallback>
          <CityCardSkeleton />
        </template>
      </Suspense>
    </div>
  </main>
</template>

<script setup>
import { ref } from 'vue';
import axios from 'axios';
import { useRouter } from 'vue-router';
import CityList from '../components/CityList.vue';
import CityCardSkeleton from '../components/CityCardSkeleton.vue';

const router = useRouter();

const previewCity = (result) => {
  //console.log(result);
  //console.log(result.name);
  //console.log(result.state);
  router.push({
    name: 'cityView',
    params: { country: result.country, name: result.name },
    query: {
      lat: result.lat,
      lon: result.lon,
      preview: true,
    },
  });
};

const openWeatherApiKey = import.meta.env.VITE_API_KEY;
const searchQuery = ref("");
const queryTimeout = ref(null);
const openWeatherResults = ref(null);
const searchError = ref(null);

const getSearchResults = () => {
  clearTimeout(queryTimeout.value);
  queryTimeout.value = setTimeout(async () => {
    if (searchQuery.value !== '') {
      try {
      const result = await axios.get(`http://api.openweathermap.org/geo/1.0/direct?q=${searchQuery.value}&limit=8&appid=${openWeatherApiKey}`);
      openWeatherResults.value = result.data;
      console.log('ResultsOW', openWeatherResults.value);
      } catch {
        searchError.value = true
      }
      return;
    }
    openWeatherResults.value = null;
  }, 500)
};
</script>
