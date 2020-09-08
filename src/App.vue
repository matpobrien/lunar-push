<template>
  <div id="app">
    <section class="header">
      <h1>Lunar Push</h1>
      <h3>Phases of the Moon and Tides</h3>
    </section>
    <section class="moon-phase">
      <div class="first-new">
        <img src="../src/assets/new_moon.svg" alt="" />
      </div>
      <div class="waxing-c">
        <img src="../src/assets/full_moon.svg" alt="" />
      </div>
      <div class="first-q">
        <img src="../src/assets/first_quarter.svg" alt="" />
      </div>
      <div class="waxing-g">
        <img src="../src/assets/waxing_gibeous.svg" />
      </div>
      <div class="full">
        <img src="../src/assets/full_moon.svg" alt="" />
      </div>
      <div class="waning-g">
        <img src="../src/assets/waning_gibeous.svg" alt="" />
      </div>
      <div class="last-q">
        <img src="../src/assets/last_quarter.svg" alt="" />
      </div>
      <div class="waning-c">
        <img src="../src/assets/waning_crescent.svg" alt="" />
      </div>
      <div class="second-new">
        <img src="../src/assets/new_moon.svg" alt="" />
      </div>
    </section>
    <section class="location">
      <form class="location-selector" @submit.prevent="fetchTide()">
        <label for="location">Location: </label>
        <input type="text" name="location" id="location" v-model="location" />
        <input type="submit" id="submit" />
        <h3 id="hidden" class="alert">
          No tide station found. Please try another city.
        </h3>
        <!-- <label for="stations" id="stations"
          >Please select the nearest tide station:
        </label>
        <select name="stations">
          <option
            :value="station.id"
            v-for="station in stations"
            :key="station.id"
            >{{ station.name }}</option
          >
        </select> -->
      </form>
    </section>

    <div id="chart-content">
      <h1>Tide Information for {{ location }} on {{ today }}</h1>
      <div id="chart-container">
        <canvas id="my-chart"></canvas>
      </div>
    </div>
  </div>
</template>

<script>
  const LOC_KEY = "109abfc0a41c4f7ca49144f18df50078";
  const TIDE_KEY = "e6b7fc92-7a1c-4e77-8702-447567b462d3";
  import Chart from "chart.js";
  export default {
    data() {
      return {
        locationURL: "https://api.opencagedata.com/geocode/v1/json?q=",
        location: "",
        lat: 0,
        long: 0,
        tideType: [],
        tideURL: "https://www.worldtides.info/api/v2?extremes&",
        tideHeight: [],
        fullTideTime: [],
        chart: null,
        stations: [],
        today: "",
      };
    },
    methods: {
      currentDate() {
        const date = new Date();
        let year = date.getFullYear();
        let month = date.getMonth() + 1;
        let day = date.getDate();
        if (month < 10) {
          month = "0" + month;
        }
        if (day < 10) {
          day = "0" + day;
        }
        this.today = `${year}-${month}-${day}`;
      },
      async fetchLatLong() {
        const locationURL =
          "https://api.opencagedata.com/geocode/v1/json?q=" +
          encodeURIComponent(this.location) +
          "&key=" +
          LOC_KEY;
        // this.createLocationURL();
        try {
          const res = await fetch(locationURL);
          const { results } = await res.json();
          this.long = results[0].geometry.lng.toFixed(6);
          this.lat = results[0].geometry.lat.toFixed(6);
        } catch (error) {
          console.error(error);
        }
      },
      resetTides() {
        this.tideType = [];
        this.tideHeight = [];
        this.stations = [];
        this.fullTideTime = [];
      },
      async fetchTide() {
        document.getElementById("hidden").classList.remove("alert");
        this.resetTides();
        this.currentDate();
        await this.fetchLatLong();
        const tideURL =
          "https://www.worldtides.info/api/v2?extremes" +
          "&date=" +
          this.today +
          "&lat=" +
          this.lat +
          "&lon=" +
          this.long +
          "&key=" +
          TIDE_KEY;
        try {
          const res = await fetch(tideURL);
          const results = await res.json();
          if (!results.error && this.chart == null) {
            for (let i = 0; i < results.extremes.length; i++) {
              this.tideType.push(results.extremes[i].type);
              this.tideHeight.push(results.extremes[i].height);
              this.fullTideTime.push(results.extremes[i].date);
            }
            var ctx = document.getElementById("my-chart").getContext("2d");
            this.chart = new Chart(ctx, {
              // The type of chart we want to create
              type: "line",
              // The data for our dataset
              data: {
                labels: this.fullTideTime,
                datasets: [
                  {
                    label: this.location,
                    backgroundColor: "#0077be",
                    borderColor: "#00a9cc",
                    data: this.tideHeight,
                  },
                ],
              },
              // Configuration options go here
              options: {},
            });
          } else {
            document.getElementById("hidden").classList.add("alert");
          }
        } catch (error) {
          console.error(error);
        }
        console.log(this.tideHeight);
      },
    },

    mounted() {},
  };
</script>

<style>
  @import url("https://fonts.googleapis.com/css2?family=Orbitron:wght@400;500;600;700;800;900&display=swap");
  * {
    margin: 0;
    padding: 0;
    font-family: Orbitron;
  }
  canvas {
    height: 500px;
    width: 500px;
  }
  h3#hidden {
    color: red;
    visibility: hidden;
  }
  h3.alert {
    visibility: visible;
  }
  img {
    max-width: 200px;
    max-height: 200px;
  }
  .moon-phase {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr 1fr 1fr 1fr 1fr 1fr 1fr;
  }
  .hidden {
    visibility: hidden;
  }
  #chart-content {
    width: 100%;
  }
  #chart-container {
    margin: 0 auto;
    height: 500px;
    width: 700px;
  }

  .header {
    width: 100%;
    height: 10vh;
    display: grid;
    grid-template-rows: repeat(50% 50%);
    text-align: center;
  }
</style>
