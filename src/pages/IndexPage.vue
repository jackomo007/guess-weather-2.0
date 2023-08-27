<template>
  <q-page class="flex column" :class="bgClass">
    <div class="col q-pt-lg q-px-md">
      <q-input
        v-model="search"
        @keyup.enter="getWeatherBySearch"
        placeholder="Tell me where..."
        borderless
        dark
      >
        <template v-slot:before>
          <q-icon @click="getLocation" name="my_location" />
        </template>

        <template v-slot:append>
          <q-btn round dense flat icon="search" @click="getWeatherBySearch" />
        </template>
      </q-input>
    </div>

    <template v-if="weatherData">
      <div class="col text-white text-center">
        <div class="text-h4 text-weight-light">
          {{ weatherData.name }}
        </div>
        <div class="text-h6 text-weight-light">
          {{ weatherDaily.days[0].conditions }}
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
    </template>

    <template v-else>
      <div class="col column text-center text-white">
        <div class="col text-h2 text-weight-thin">Guess<br />Weather</div>
        <q-btn flat class="col" @click="getLocation">
          <q-icon left size="3em" name="my_location" />
          <div>Guess where i am...</div>
        </q-btn>
      </div>
    </template>
    <div class="col city"></div>
  </q-page>
</template>

<script lang="ts">
import { WeatherData, WeatherHour } from 'components/models';
import { defineComponent, onBeforeUnmount } from 'vue';
import { useQuasar } from 'quasar';

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
        this.getWeatherByHour(
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
    getWeatherByHour(lat: number, lon: number) {
      this.$axios(
        `${this.weatherByHourUrl}/${lat},${lon}/2023-08-27?include=hours&lang=es&key=${this.weatherByHourKey}&unitGroup=metric`
      )
        .then((response) => {
          console.log(response);
          this.weatherDaily = response.data;
          this.hideLoading();
        })
        .catch((error) => this.hideLoading());
    },
    getWeatherBySearch() {
      this.showLoading();
      this.$axios(
        `${this.weatherUrl}?q=${this.search}&appid=${this.weatherKey}&units=metric`
      )
        .then((response) => {
          this.weatherData = response.data;
          this.search = '';
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
.degree
  top: -44px
.city
  flex: 0 0 200px
  background: url(/city2.png)
  background-size: contain
  background-position: center bottom
</style>
