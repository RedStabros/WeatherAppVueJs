<template>
  <div class="flex flex-col flex-1 items-center">
    <div v-if="route.query.preview" class="text-white p-4 bg-weather-secondary w-full text-center">
        <p>You are currently previewing this city, click the "+" icon to start tracking this city</p>
    </div>
    <div class="flex flex-col items-center text-white py-12">
        <h1 class="text-4xl mb-2">{{ route.params.name }}</h1>
        <p class="text-sm mb-12">
        {{
          new Date(weatherData.currentTime).toLocaleDateString(
            "en-us",
            {
              weekday: "short",
              day: "2-digit",
              month: "long",
            }
          )
        }}
        {{
          new Date(weatherData.currentTime).toLocaleTimeString(
            "en-us",
            
                //{  hour: "2-digit",  minute: "2-digit" },
                { timeStyle: "short" }
            
          )
        }}
      </p>
      <p class="text-8xl mb-8">
        {{ Math.round(weatherData.main.temp) }} &deg;C
      </p>
      <p>
        Feels like
        {{ Math.round(weatherData.main.feels_like) }} &deg;C
      </p>
      <p class="capitalize">
        {{ weatherData.weather[0].description }}
      </p>
      <img
        class="w-[150px] h-auto"
        :src="
          `http://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`
        "
        alt=""
      />
    </div>
    <hr class="border-white border-opacity-10 border w-full" />

    <!-- 3 Hourly Weather -->

    <div class="max-w-screen-md w-full py-12 mb-6">
      <div class="mx-8 text-white">
        <h2 class="mb-4 text-lg">Every 3 hours Weather </h2>
        <div class="flex gap-10 overflow-x-scroll">
          <div
            v-for="foreHourData in forecastData.list"
            :key="foreHourData.dt"
            class="flex flex-col gap-4 items-center"
          >
            <p class="whitespace-nowrap text-md">
              {{
                new Date(
                  foreHourData.dt_txt
                ).toLocaleTimeString("en-us", {
                  hour: "numeric", day: "numeric", month: "short"
                })
              }}
            </p>
            <img
              class="w-auto h-[50px] object-cover"
              :src="
                `http://openweathermap.org/img/wn/${foreHourData.weather[0].icon}@2x.png`
              "
              alt=""
            />
            <p class="text-xl">
              {{ Math.round(foreHourData.main.temp) }}&deg;C
            </p>
          </div>
        </div>
      </div>
    </div>

    <hr class="border-white border-opacity-10 border w-full" />

    <!-- 2 Weekly Weather -->
    <div class="max-w-screen-md w-full py-12 mt-4">
      <div class="mx-8 text-white">
        <h2 class="mb-4 text-lg">5 Days Forecast Weather</h2>
        <div
          v-for="day in forecastData.list"
          :key="day.dt"
          class="flex items-center"
        >
          <p class="flex-1">
            {{
              new Date(day.dt * 1000).toLocaleDateString(
                "en-us",
                {
                  weekday: "long",
                }
              )
            }}
          </p>
          <img
            class="w-[50px] h-[50px] object-cover"
            :src="
              `http://openweathermap.org/img/wn/${day.weather[0].icon}@2x.png`
            "
            alt=""
          />
          <div class="flex gap-2 flex-1 justify-end">
            <p>H: {{ Math.round(day.main.temp_max) }}&deg;C</p>
            <p>L: {{ Math.round(day.main.temp_min) }}&deg;C</p>
          </div>
        </div>
      </div>
    </div>
    <div class="flex items-center gap-2 cursor-pointer py-12 duration-150 text-white hover:text-slate-800" @click="removeCity">
      <i class="fa-solid fa-trash"></i>
      <p>Remove City</p>
    </div>
  </div>
</template>

<script>

</script>

<script setup>
import axios from "axios";
import { useRoute, useRouter } from "vue-router";
import.meta.env.VITE_API_KEY;

const openWeatherApiKey = import.meta.env.VITE_API_KEY;

const route = useRoute();
const getWeatherData = async () => {
  console.log('open', openWeatherApiKey)
  try {
    const weatherData = await axios.get(
      `https://api.openweathermap.org/data/2.5/weather?lat=${route.query.lat}&lon=${route.query.lon}&appid=${openWeatherApiKey}&units=metric`
    );
    
    // cal current date & time
    const localOffset = new Date().getTimezoneOffset() * 60000;
    const utc = weatherData.data.dt * 1000 + localOffset;
    weatherData.data.currentTime =
      utc + 1000 * weatherData.data.timezone;
/*
    // cal hourly weather offset
    weatherData.data.hourly.forEach((hour) => {
      const utc = hour.dt * 1000 + localOffset;
      hour.currentTime = utc + 1000 * weatherData.data.timezone_offset;
        
    });*/

     //Flicker Delay
     await new Promise((res) => setTimeout(res, 500));

    return weatherData.data;

  } catch (error) {
    console.log(error);
  }
};

const weatherData = await getWeatherData();
console.log(weatherData)

// 3 Hours

const getForecastData = async () => {
  try {
    const forecastData = await axios.get(
      `https://api.openweathermap.org/data/2.5/forecast?lat=${route.query.lat}&lon=${route.query.lon}&appid=${openWeatherApiKey}&units=metric`
    );
    
    // cal current date & time
    const localOffset = new Date().getTimezoneOffset() * 60000;
    const utc = forecastData.data.dt * 1000 + localOffset;
    forecastData.data.currentTime =
      utc + 1000 * forecastData.data.timezone;
/*
    // cal hourly weather offset
    forecastData.data.hourly.forEach((hour) => {
      const utc = hour.dt * 1000 + localOffset;
      hour.currentTime = utc + 1000 * forecastData.data.timezone_offset;
        
    });*/
    return forecastData.data;

  } catch (error) {
    console.log(error);
  }
};

const forecastData = await getForecastData();
console.log(forecastData)

//16 Daily (No API access)
/*
const getDailyForecastData = async () => {
  try {
    const dailyForecastData = await axios.get(
      `https://api.openweathermap.org/data/2.5/forecast/daily?lat=${route.query.lat}&lon=${route.query.lon}&appid=${openWeatherApiKey}&units=metric&cnt=16`
    );

    return dailyForecastData.data;

  } catch (error) {
    console.log(error);
  }
};

const dailyForecastData = await getDailyForecastData();
console.log('daily', dailyForecastData)
*/
//REMOVE
const router = useRouter();
const removeCity = () => {
  const cities = JSON.parse(localStorage.getItem('savedCities'));
  const updatedCities = cities.filter((city) => city.id !== route.query.id);
  localStorage.setItem('savedCities', JSON.stringify(updatedCities));
  router.push({
    name: 'home',
  });
};

</script>
