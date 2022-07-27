<template>
  <main :class="getClass()">
      <div class="hello">
        <div class="inner">
          <PartlyCloudy class="icon"/>
          <span class="header">{{ msg }}</span>
        </div>
      </div>
      <div id="autocomplete" class="autocomplete-container" @keyup.enter="getWeather"></div>
    <el-alert
        :title="city + ' is not a city'"
        type="error"
        description="Please enter a valid city name."
        show-icon
        id="alert"
        style="display: none"
    />
      <div class="city">
        {{ weather.province }}, {{ weather.country }}
      </div>
      <div class="date">
        {{ dateBuilder() }}
      </div>
      <div class="tempBox">
        <span class="temperature">
          {{ Math.round(weather.temp) }}°C
        </span>
      </div>
      <div class="weatherState">
        {{ weather.state }}
      </div>
  </main>
</template>

<script>
import axios from "axios";
import { PartlyCloudy } from '@element-plus/icons-vue'
import { GeocoderAutocomplete } from '@geoapify/geocoder-autocomplete';


export default {
  components: {
    PartlyCloudy,
  },
  name: 'WeatherComponent',
  props: {
    msg: String
  },
  data() {
    return {
      api_key: "8feb28dfe3efd04df7e475a9327244fa",
      url: "https://api.openweathermap.org/data/2.5/",
      city: "",
      currentLocation: "",
      weather: {
        country: "",
        temp: null,
        state: "",
        province: "",
        temp_min: null,
        temp_max: null,
      },
      lat: 0,
      lon: 0,
      autocomplete: null
    }
  },
  methods: {
    async getWeather() {
      document.getElementById("alert").style.display = "none"
      let response = await axios.get(
          `${this.url}weather?q=${this.city}&units=metric&APPID=${this.api_key}`
      ).catch(function (error) {
        if(error.request) {
          document.getElementById("alert").style.display = "block"
        }
      })
      this.weather.country = response.data.sys.country
      this.weather.temp = response.data.main.temp
      this.weather.state = response.data.weather[0].main
      this.weather.province = response.data.name
      this.weather.temp_max = response.data.main.temp_max
      this.weather.temp_min = response.data.main.temp_min
    },
    async getLocation() {
      await this.getCoordinates()
      const lat = localStorage.getItem("lat")
      const lon = localStorage.getItem("lon")
      let response = await axios.get(`https://api.opencagedata.com/geocode/v1/json?q=${lat}+${lon}&key=1f05dca7ebf14879a24839d393819557`)
          .catch(function (error) {
            if(error.request) {
              console.log(error)
            }
          })
      this.city = response.data.results[0].components.state
    },
    dateBuilder() {
      let d = new Date();
      let months = [
        "January",
        "February",
        "March",
        "April",
        "May",
        "June",
        "July",
        "August",
        "September",
        "October",
        "November",
        "December",
      ];
      let days = [
        "Sunday",
        "Monday",
        "Tuesday",
        "Wednesday",
        "Thursday",
        "Friday",
        "Saturday",
      ];
      let day = days[d.getDay()];
      let date = d.getDate();
      let month = months[d.getMonth()];
      let year = d.getFullYear();
      return `${day} ${date} ${month} ${year}`;
    },
    getClass() {
      return {
        'sunny': this.weather.state === "Clear",
        'cloudy': this.weather.state === "Clouds",
        'rainy': this.weather.state === "Rain",
        'mist': this.weather.state === "Mist"
      }
    },
    autocompleteMount() {
      this.autocomplete = new GeocoderAutocomplete(
          document.getElementById("autocomplete"),
          '1c8f452fd2464379b84c43ce6710999a',
          {type: "city", placeholder: "Enter a city...",
          skipDetails: true})
      this.autocomplete.on('select', (location) => {
        this.city = location.properties.city
      })
      this.autocomplete.on('suggestions', (suggestions) => {
        for(let i = 0; i < suggestions.length; i++) {
          suggestions[i].properties.formatted = suggestions[i].properties.city
        }
      })
    },
    async getCoordinates() {
      navigator.geolocation.getCurrentPosition(position => {
        localStorage.setItem("lat", position.coords.latitude)
        localStorage.setItem("lon", position.coords.longitude)
      })
    }
  },
  async mounted() {
    await this.getLocation()
    await this.getWeather()
    this.autocompleteMount()
  },

}
</script>

<style scoped>
@import "~@geoapify/geocoder-autocomplete/styles/minimal.css";

.mist {
  background-image: url("../assets/mist.jpg");
  background-size: 100% 100%;
}
.sunny {
  background-image: url("../assets/clear.jpg");
  background-size: 100% 100%;
}
.cloudy {
  background-image: url("../assets/partly_cloudy.jpeg");
  background-size: 100% 100%;
}
.rainy {
  background-image: url("../assets/rain.jpg");
  background-size: 100% 100%;
}
.autocomplete-container {
  position: relative;
  width: 30%;
  margin: 0 auto;
}
main {
  height: 600px;
}
.city {
  font-size: 18px;
  margin: 20px auto;
}
.weatherState {
  font-size: 30px;
}
.temperature {
  position: absolute;
  margin-top: 18px;
  right: 603px;
}
.tempBox {
  border: 1px solid black;
  font-size: 35px;
  border-radius: 8px;
  box-shadow: black;
  width: 150px;
  height: 80px;
  margin: 10px auto;
}
.date {
  margin: 10px auto;
  font-size: 20px;
}
.searchBar {
  padding: 10px;
  width: 250px;
  font-size: 18px;
}
.inner {
  width: 1050px;
}
.icon{
  height: 100px;
}
.header {
  font-size: 40px;
  position: absolute;
  margin-top: 26px;
  margin-left: 10px;
}
h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.el-alert {
  margin: 20px auto;
  width: 30%;
}
.el-alert:first-child {
  margin: 0 auto;
}
</style>
