<template>
  <div id="app" class="container">
    <h1 class="my-4">Neo Stats</h1>
    <form @submit.prevent="retrieveAsteroids">
      <div class="form-row">
        <div class="form-group col-md-6">
          <label for="startDate">Select Start Date</label>
          <input type="date" v-model="startDate" class="form-control" required />
        </div>
        <div class="form-group col-md-6">
          <label for="endDate">Select End Date</label>
          <input type="date" v-model="endDate" class="form-control" required />
        </div>
      </div>
      <button type="submit" class="btn btn-primary">Get Data</button>
      <div v-if="errorMsg" class="alert alert-danger my-3">{{ errorMsg }}</div>
    </form>

    <line-chart v-if="chartData" :chart-data="chartData" />

    <div v-if="stats" class="my-4">
      <h3>Statistics Summary</h3>
      <p>Fastest Asteroid: {{ stats.fastest.name }} with a speed of {{ stats.fastest.speed }} km/h</p>
      <p>Nearest Asteroid: {{ stats.closest.name }} at a distance of {{ stats.closest.distance }} km</p>
      <p>Average Size: {{ stats.averageSize.toFixed(2) }} km</p>
    </div>
  </div>
</template>

<script>
import LineChart from './components/LineChart.vue';

export default {
  name: 'App',
  components: {
    LineChart,
  },
  data() {
    return {
      startDate: '',
      endDate: '',
      chartData: null,
      stats: null,
      errorMsg: '',
    };
  },
  methods: {
    async retrieveAsteroids() {
      this.clearErrorMsg();

      if (!this.isValidDateRange()) return;

      const apiKey = 'eWnK88eVfF1JQHH7dUHiXevLsF7hx0u5IPg8gcvA';
      try {
        const data = await this.fetchAsteroidData(apiKey);
        this.processAsteroidData(data);
      } catch (error) {
        this.setErrorMsg('An error occurred while retrieving data. Please try again later.');
        console.error(error);
      }
    },

    clearErrorMsg() {
      this.errorMsg = '';
    },

    isValidDateRange() {
      const datePattern = /^\d{4}-\d{2}-\d{2}$/;
      if (!this.startDate.match(datePattern) || !this.endDate.match(datePattern)) {
        this.setErrorMsg('Please enter the dates in the format YYYY-MM-DD.');
        return false;
      }

      const start = new Date(this.startDate);
      const end = new Date(this.endDate);
      const duration = (end - start) / (1000 * 3600 * 24);

      if (duration < 0) {
        this.setErrorMsg('The end date must be after the start date.');
        return false;
      }
      if (duration > 7) {
        this.setErrorMsg('The selected date range cannot exceed 7 days.');
        return false;
      }
      return true;
    },

    setErrorMsg(message) {
      this.errorMsg = message;
    },

    async fetchAsteroidData(apiKey) {
      const response = await fetch(
        `https://api.nasa.gov/neo/rest/v1/feed?start_date=${this.startDate}&end_date=${this.endDate}&api_key=${apiKey}`
      );
      const data = await response.json();

      if (data.code && data.code === 400) {
        this.setErrorMsg(data.error_message);
        throw new Error(data.error_message);
      }
      return data;
    },

    processAsteroidData(data) {
      const asteroidCounts = {};
      let fastestAsteroid = { speed: 0 };
      let closestAsteroid = { distance: Infinity };
      let totalDiameter = 0;
      let asteroidCount = 0;

      for (const date in data.near_earth_objects) {
        asteroidCounts[date] = data.near_earth_objects[date].length;

        data.near_earth_objects[date].forEach(asteroid => {
          const speed = asteroid.close_approach_data[0].relative_velocity.kilometers_per_hour;
          const distance = asteroid.close_approach_data[0].miss_distance.kilometers;
          const size = this.calculateAverageSize(asteroid);

          fastestAsteroid = this.updateFastestAsteroid(fastestAsteroid, asteroid, speed);
          closestAsteroid = this.updateClosestAsteroid(closestAsteroid, asteroid, distance);

          totalDiameter += size;
          asteroidCount++;
        });
      }

      this.chartData = this.prepareChartData(asteroidCounts);
      this.stats = {
        fastest: fastestAsteroid,
        closest: closestAsteroid,
        averageSize: asteroidCount > 0 ? totalDiameter / asteroidCount : 0,
      };
    },

    calculateAverageSize(asteroid) {
      return (asteroid.estimated_diameter.kilometers.estimated_diameter_max +
              asteroid.estimated_diameter.kilometers.estimated_diameter_min) / 2;
    },

    updateFastestAsteroid(currentFastest, asteroid, speed) {
      return speed > currentFastest.speed ? { name: asteroid.name, speed } : currentFastest;
    },

    updateClosestAsteroid(currentClosest, asteroid, distance) {
      return distance < currentClosest.distance ? { name: asteroid.name, distance } : currentClosest;
    },

    prepareChartData(asteroidCounts) {
      const labels = Object.keys(asteroidCounts).map(date => {
        const [year, month, day] = date.split('-');
        return `${day}-${month}-${year}`;
      });

      return {
        labels: labels, 
        datasets: [
          {
            label: 'Asteroid Count',
            data: Object.values(asteroidCounts),
            borderColor: 'rgba(75, 192, 192, 1)',
            borderWidth: 1,
            fill: false,
          },
        ],
      };
    },
  },
};
</script>

<style>
@import '~bootstrap/dist/css/bootstrap.min.css';
.alert {
  margin-top: 20px;
}
</style>