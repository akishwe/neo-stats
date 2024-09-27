<template>
  <div class="container">
    <div class="row">
      <div class="col-md-12">
        <h2 class="text-center my-4">Asteroid Statistics Chart</h2>
        <canvas ref="asteroidChart" class="shadow-sm"></canvas>
      </div>
    </div>
  </div>
</template>

<script>
import { Chart, registerables } from 'chart.js';

Chart.register(...registerables);

export default {
  name: 'LineChart',

  props: {
    chartData: {
      type: Object,
      required: true,
    },
  },

  mounted() {
    this.renderChart();
  },

  watch: {
    chartData: 'renderChart',
  },

  methods: {
    renderChart() {
      if (this.chart) {
        this.chart.destroy();
      }

      const ctx = this.$refs.asteroidChart.getContext('2d');

      this.chart = new Chart(ctx, {
        type: 'line',
        data: this.chartData,
        options: {
          scales: {
            y: {
              beginAtZero: true,
            },
          },
        },
      });
    },
  },

  beforeUnmount() {
    if (this.chart) {
      this.chart.destroy();
    }
  },
};
</script>

<style scoped>
.shadow-sm {
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.1);
}
</style>
