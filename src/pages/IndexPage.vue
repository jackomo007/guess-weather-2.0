<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input
        v-model="search"
        @keyup.enter="searchWeather"
        placeholder="A donde vamos?"
        borderless
        dark
      >
        <template v-slot:before>
          <q-icon @click="getLocation" name="my_location" />
        </template>

        <template v-slot:append>
          <q-btn round dense flat icon="search" @click="searchWeather" />
        </template>
      </q-input>
    </div>

    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">
          {{ weatherData.name }}
        </div>
        <div class="text-h6 text-weight-light">
          {{ weatherDaily?.days[0].conditions }}
        </div>

        <div class="text-h1 text-weight-thin q-my-lg">
          <span>{{ Math.round(weatherData.main.temp) }}</span>
          <span class="text-h4 relative-position degree">&deg;C</span>
        </div>
      </div>

      <div class="col text-center">
        <img
          :src="`https://openweathermap.org/img/wn/${weatherData.weather[0].icon}@2x.png`"
          alt="weather_icon"
        />
      </div>
      <div class="rowIcons">
        <div class="icon">
          <img
            src="icons/wind.png"
            alt="Velocidad de vientos"
            title="Velocidad de vientos"
            width="70"
          />
          <span class="complement"
            >{{ weatherDaily?.days[0]?.windspeed }} Km/h</span
          >
        </div>
        <div class="icon">
          <img
            src="icons/rain.png"
            alt="Probabilidad de Lluvia"
            title="Probabilidad de Lluvia"
            width="70"
          />
          <span class="complement"
            >{{ weatherDaily?.days[0]?.precipprob }} %</span
          >
        </div>
        <div class="icon">
          <img
            src="icons/uv.png"
            alt="Porcentaje de UV"
            title="Porcentaje de UV"
            width="70"
          />
          <span class="complement">{{ weatherDaily?.days[0]?.uvindex }} %</span>
        </div>
      </div>
      <div class="daily">
        <div v-for="hour in weatherDaily?.days[0]?.hours" :key="hour.datetime">
          <div class="hour">
            <span class="temp">{{ hour.temp }} &deg;C</span>
            <span class="time">{{ hour.datetime.replace('00:00', '00') }}</span>
          </div>
          <div class="weatherIcon">
            <img
              :src="`https://github.com/visualcrossing/WeatherIcons/blob/main/PNG/1st%20Set%20-%20Color/${hour.icon}.png?raw=true`"
              alt="weather_icon"
            />
          </div>
        </div>
      </div>
    </template>

    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin title">
          Prueba usando el gps<br />o busca un lugar que te guste...
        </div>
        <q-btn flat class="col" @click="getLocation">
          <q-icon left size="3em" name="my_location" />
          <div>Por donde andas?</div>
        </q-btn>
      </div>
    </template>
    <div class="col city"></div>
  </q-page>
</template>

<script lang="ts">
import { WeatherData, WeatherHour } from 'components/models';
import { defineComponent, onBeforeUnmount } from 'vue';
import { useQuasar, date } from 'quasar';

export default defineComponent({
  name: 'IndexPage',
  setup() {
    const $q = useQuasar();
    let timer: any;

    onBeforeUnmount(() => {
      if (timer !== void 0) {
        clearTimeout(timer);
        $q.loading.hide();
      }
    });

    return {
      showLoading() {
        $q.loading.show();
      },
      hideLoading() {
        $q.loading.hide();
      },
    };
  },
  data() {
    return {
      search: '',
      weatherData: null as WeatherData | null,
      weatherDaily: null as WeatherHour | null,
      weatherKey: 'c65ad845185d5c6fc0d1063d6e5634e4',
      weatherUrl: 'https://api.openweathermap.org/data/2.5/weather',
      weatherByHourKey: 'LA5WY8QYQ5YC9UX79YBX6EKND',
      weatherByHourUrl:
        'https://weather.visualcrossing.com/VisualCrossingWebServices/rest/services/timeline',
    };
  },
  computed: {
    bgClass() {
      if (this.weatherData) {
        if (this.weatherData.weather[0].icon.endsWith('n')) {
          return 'bg-night';
        } else {
          return 'bg-day';
        }
      }
      return 'bg-day';
    },
  },
  methods: {
    getLocation() {
      this.showLoading();
      navigator.geolocation.getCurrentPosition((position) => {
        this.getWeatherByCoords(
          position.coords.latitude,
          position.coords.longitude
        );
        this.getHourlyWeatherByCoordinates(
          position.coords.latitude,
          position.coords.longitude
        );
      });
    },
    getWeatherByCoords(lat: number, lon: number) {
      this.$axios(
        `${this.weatherUrl}?lat=${lat}&lon=${lon}&appid=${this.weatherKey}&units=metric`
      )
        .then((response) => {
          this.weatherData = response.data;
        })
        .catch((error) => this.hideLoading());
    },
    getHourlyWeatherByCoordinates(lat: number, lon: number) {
      const timeStamp = Date.now();
      const formattedDate = date.formatDate(timeStamp, 'YYYY-MM-DD');
      this.$axios(
        `${this.weatherByHourUrl}/${lat},${lon}/${formattedDate}?include=hours&lang=es&key=${this.weatherByHourKey}&unitGroup=metric`
      )
        .then((response) => {
          this.weatherDaily = response.data;
          this.hideLoading();
        })
        .catch((error) => this.hideLoading());
    },
    searchWeather() {
      this.getWeatherBySearch();
      this.getHourlyWeatherBySearch();
    },
    getWeatherBySearch() {
      this.showLoading();
      this.$axios(
        `${this.weatherUrl}?q=${this.search}&appid=${this.weatherKey}&units=metric`
      )
        .then((response) => {
          this.weatherData = response.data;
        })
        .catch((error) => this.hideLoading());
    },
    getHourlyWeatherBySearch() {
      const timeStamp = Date.now();
      const formattedDate = date.formatDate(timeStamp, 'YYYY-MM-DD');
      this.$axios(
        `${this.weatherByHourUrl}/${this.search}/${formattedDate}?include=hours&lang=es&key=${this.weatherByHourKey}&unitGroup=metric`
      )
        .then((response) => {
          this.weatherDaily = response.data;
          this.hideLoading();
        })
        .catch((error) => this.hideLoading());
    },
  },
});
</script>

<style lang="sass">
.q-page
  background: linear-gradient(to bottom, #642b73, #c6426e)
  &.bg-night
    background: linear-gradient(to bottom, #232526, #414345)
  &.bg-day
    background: linear-gradient(to bottom, #00c6ff, #0072ff)
.title
  font-size: clamp(2rem, 2.5vw, 4rem)
.degree
  top: -44px
.city
  flex: 0 0 200px
  background: url(/city2.png)
  background-size: contain
  background-position: center bottom
.daily
  display: flex
  flex-direction: row
  height: 175px
  width: 100%
  padding: 10px
  overflow-x: auto
  gap: 5px
  justify-content: center

  ::-webkit-scrollbar
   width: 100%
   height: 2px
   background-color: white

.hour
  display: flex
  width: 80px
  height: 80px
  border: 2px solid white
  flex-direction: column
  text-align: center
  justify-content: center
  border-radius: 100%
.complement
  font-size: 20px
  color: #F2C037
  width: 100px
.temp
  font-size: 20px
  color: #F2C037
.time
  font-size: 15px
  color: #BAE5FB
.rowIcons
  display: flex
  justify-content: space-evenly
.icon
  display: flex
  flex-direction: column
  justify-content: center
  align-items: center
  width: 100px
  text-align: center
.weatherIcon
  display: flex
  justify-content: center
</style>
