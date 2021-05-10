<template>
  <div class="wrapper">
    <h1>Прогноз погоды</h1>
    <div class="pressure">
      Максимальное давление за последние 5 дней:{{ maxPressure }}
    </div>
    <div>
      <button @click="loadPressure" class="btn">Расчёт давления</button>
    </div>
    <div class="temperature">
      День с минимальной разницей температур:{{ realDay }}
    </div>
    <div>
      <button @click="loadTemp" class="btn">Расчёт температуры</button>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "HelloWorld",
  data() {
    return {
      date: (Date.now() / 1000 - 86400).toFixed(0),
      lat: 60.99,
      lon: 30.9,
      unixDay: 86400,
      realDay: 0,
      maxPressure: 0,
    };
  },
  methods: {
    async loadPressure() {
      let array = [];
      const instance = axios.create({
        baseURL: "http://api.openweathermap.org/data/2.5/onecall",
        params: {
          appid: "21e9bc9c7139e76a6bc9e3a6a63860df",
          lat: this.lat,
          lon: this.lon,
        },
      });
      axios
        .all([
          await instance.get("/timemachine", { 
            params: { dt: this.date } 
          }),
          await instance.get("/timemachine", {
            params: { dt: this.date - 1 * this.unixDay }
          }),
          await instance.get("/timemachine", {
            params: { dt: this.date - 2 * this.unixDay }
          }),
          await instance.get("/timemachine", {
            params: { dt: this.date - 3 * this.unixDay }
          }),
          await instance.get("/timemachine", {
            params: { dt: this.date - 4 * this.unixDay }
          })
        ])
        .then((response) => {
          for (let i = 0; i < response.length; i++) {
            for (let j = 0; j < response[i].data.hourly.length; j++) {
              array.push(response[i].data.hourly[j].pressure);
            }
          }
          this.maxPressure = Math.max.apply(null, array);
        });
    },
    async loadTemp() {
      let arr = [];
      await axios
        .get("http://api.openweathermap.org/data/2.5/onecall", {
          params: {
            appid: "21e9bc9c7139e76a6bc9e3a6a63860df",
            exclude: "current,minutely,hourly,alerts",
            lat: this.lat,
            lon: this.lon,
          },
        })
        .then((response) => {
          for (let k = 0; k < response.data.daily.length; k++) {
            arr.push(
              Math.abs(
                response.data.daily[k].temp.night -
                  response.data.daily[k].temp.morn
              ).toFixed(2)
            );
          }
          let min = arr.indexOf(String(Math.min.apply(null, arr)));
          let minDay = new Date(response.data.daily[min].dt * 1000);
          this.realDay =
            minDay.getDate() +
            "/" +
            (minDay.getMonth() + 1) +
            "/" +
            minDay.getFullYear();
        });
    },
  },
};
</script>


<style scoped>
label {
  padding-right: 30px;
}

.btn {
  margin: 10px 0;
}

p {
  font-size: 10px;
  margin-top: 5px;
}

a {
  color: #42b983;
}
</style>
