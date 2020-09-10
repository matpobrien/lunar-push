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
        <h3 id="hidden" class="alert" v-if="showAlert">
          No tide station found. Please try another city.
        </h3>
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
  import Chart from "chart.js";

  const LOC_KEY = "109abfc0a41c4f7ca49144f18df50078";
  const TIDE_KEY = "e6b7fc92-7a1c-4e77-8702-447567b462d3";

  const getLocationURL = (location) => {
    return (
      "https://api.opencagedata.com/geocode/v1/json?q=" +
      encodeURIComponent(location) +
      "&key=" +
      LOC_KEY
    );
  };

  const getTideURL = (lat, long, today) => {
    return (
      "https://www.worldtides.info/api/v2?extremes" +
      "&date=" +
      today +
      "&lat=" +
      lat +
      "&lon=" +
      long +
      "&key=" +
      TIDE_KEY
    );
  };

  const getCurrentDate = () => {
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
    return `${year}-${month}-${day}`;
  };

  async function fetchLatLong(location) {
    const res = await fetch(getLocationURL(location));
    const { results } = await res.json();
    const geometry = results[0].geometry;
    return { lat: geometry.lat.toFixed(6), long: geometry.lng.toFixed(6) };
  }

  async function fetchTideAPI(lat, long, today) {
    console.log(lat, long, today);
    console.log(getTideURL);
    // const res = await fetch(getTideURL(lat, long, this.today));
    // return await res.json();
    return {
      status: 200,
      callCount: 1,
      copyright:
        "Tidal data retrieved from www.worldtides.info. Copyright (c) 2014-2020 Brainware LLC. Licensed for use of individual spatial coordinates on behalf of by an end-user. Source data created by the Center for Operational Oceanographic Products and Services (CO-OPS) and is not subject to copyright protection. NO GUARANTEES ARE MADE ABOUT THE CORRECTNESS OF THIS DATA. You may not use it if anyone or anything could come to harm as a result of using it (e.g. for navigational purposes).",
      requestLat: 33.768321,
      requestLon: -118.195617,
      responseLat: 33.7717,
      responseLon: -118.21,
      atlas: "NOAA",
      station: "Long Beach, Inner Harbor, California",
      extremes: [
        {
          dt: 1599553934,
          date: "2020-09-08T08:32+0000",
          height: 0.16,
          type: "High",
        },
        {
          dt: 1599572314,
          date: "2020-09-08T13:38+0000",
          height: -0.132,
          type: "Low",
        },
        {
          dt: 1599596635,
          date: "2020-09-08T20:23+0000",
          height: 0.528,
          type: "High",
        },
        {
          dt: 1599623786,
          date: "2020-09-09T03:56+0000",
          height: -0.316,
          type: "Low",
        },
        {
          dt: 1599647405,
          date: "2020-09-09T10:30+0000",
          height: 0.047,
          type: "High",
        },
        {
          dt: 1599659688,
          date: "2020-09-09T13:54+0000",
          height: -0.009,
          type: "Low",
        },
        {
          dt: 1599686362,
          date: "2020-09-09T21:19+0000",
          height: 0.506,
          type: "High",
        },
        {
          dt: 1599716584,
          date: "2020-09-10T05:43+0000",
          height: -0.375,
          type: "Low",
        },
      ],
    };
  }

  export default {
    data() {
      return {
        location: "",
        chart: null,
        today: "",
        showAlert: false,
      };
    },
    methods: {
      async fetchTide() {
        try {
          this.showAlert = false;
          this.today = getCurrentDate();
          const { lat, long } = await fetchLatLong(this.location);
          const tideData = await fetchTideAPI(lat, long, this.today);
          if (!tideData.error) {
            this.createChart(tideData.extremes);
          } else {
            this.showAlert = true;
          }
        } catch (error) {
          console.error(error);
        }
      },

      createChart(extremes) {
        if (this.chart) {
          return;
        }
        var ctx = document.getElementById("my-chart").getContext("2d");
        this.chart = new Chart(ctx, {
          // The type of chart we want to create
          type: "line",
          // The data for our dataset
          data: {
            labels: extremes.map((extreme) => extreme.date),
            datasets: [
              {
                label: this.location,
                backgroundColor: "#0077be",
                borderColor: "#00a9cc",
                data: extremes.map((extreme) => extreme.height),
              },
            ],
          },
          // Configuration options go here
          options: {},
        });
      },
    },
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
