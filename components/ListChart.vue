<template>
  <div>
    <label>
      <input v-model="shigaPrefecture" type="checkbox" value="25" @click="changePrefectureCode(25)"> 滋賀県
    </label>
    <label>
      <input v-model="kyotoPrefecture" type="checkbox" value="26" @click="changePrefectureCode(26)"> 京都府
    </label>
    <highcharts ref="chart" :options="chartOptions" />
  </div>
</template>

<script>

import axios from 'axios';
export default {
  data() {
    return {
      chartData: [],
      chartOptions: {},
      prefCode: 25, // Default prefecture code
      filteredLabel: '出生数', // Default filtered label
      shigaPrefecture: true,
      kyotoPrefecture: false,
    }
  },
  mounted() {
  this.fetchChartData();
  },
  methods: {
    async fetchChartData() {
      try {
        const response = await axios.get('https://opendata.resas-portal.go.jp/api/v1/population/sum/estimate', {
            headers: {
              'X-API-KEY': 'tz1My6YajsPFJacWLq91GqYLXB8pNkNRD4zQ1vJQ',
            },
            params: {
              cityCode: '-', // Select all the cities
              prefCode: this.prefCode // Use the selected prefecture code
            }
          }
        );

        // Process the API response and extract necessary data
        const resasData = response.data.result.data;

        // Find the data object that matches the filtered label
        const birthData = resasData.find(item => item.label === this.filteredLabel);

        if (birthData) {
          // Extract xAxis categories
          const xAxisCategories = birthData.label;

          // Extract series data
          const seriesData = {
            name: birthData.label,
            data: birthData.data.map(dataItem => [dataItem.year, dataItem.value])
          }

          // Update chart options
            this.updateChartOptions(xAxisCategories, seriesData);
        }
      } catch (error) {
        return error
      }
    },
    updateChartOptions(xAxisCategories, seriesData) {
      this.chartOptions = {
        chart: {
          type: 'line',
        },
        title: {
          text: '出生数',
        },
        xAxis: {
          categories: xAxisCategories,
        },
        yAxis: {
          title: {
            text: '',
          },
        },
        series: seriesData,
      };
    },
    async changePrefectureCode(prefCode) {
      if (prefCode === 25) {
        this.shigaPrefecture = true
        this.kyotoPrefecture = false
      } else {
        this.kyotoPrefecture = true
        this.shigaPrefecture = false
      }
      this.prefCode = prefCode; // Update the selected prefecture code
      await this.fetchChartData(); // Fetch data for the selected prefecture code
    },
  }
}
</script>