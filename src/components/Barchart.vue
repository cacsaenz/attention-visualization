<template>
  <div>
    <highcharts ref="chart" :options="chartOptions"></highcharts>
  </div>
</template>

<script>
import { Chart } from 'highcharts-vue';

export default {
  name: 'Barchart',
  components: {
    highcharts: Chart,
  },
  props: {
    words: Array,
    onlyPIWs: Boolean,
  },
  methods: {
    handleChartClick({ point }) {
      this.$emit('show-word-details', point.category);
    }
  },
  computed: {
    chartOptions() {
      const self = this;
      const options = {
        chart: {
          type: 'bar',
          height: '500px',
        },
        tooltip: {
          valueDecimals: 2,
        },
        title: {
          text: self.onlyPIWs ? 'Positive Influential Words' : 'Influential Words',
        },
        xAxis: {
          categories: this.words.map(word => word.word),
        },
        yAxis: {
          title: {
            text: 'Absolute attention',
          },
        },
        legend: {
          reversed: true,
        },
        plotOptions: {
          series: {
            stacking: 'normal',
            marker: {
              enabled: true,
              symbol: 'square',
              radius: 3.5
            },
          },
        },
        series: [
          {
            name: 'PAW Correct',
            data: this.words.map(word => word.attention_correct),
            type: 'bar',
            color: '#2d93ad',
            point: {
              events: {
                  click: self.handleChartClick,
              },
            },
            enableMouseTracking: true,
          },
        ],
      };
      if (!self.onlyPIWs) {
        options.series = [
          {
            data: this.words.map(word => word.threshold),
            type: 'line',
            lineWidth: 0,
            name: 'Threshold',
            color: '#000',
            states: {
              hover: {
                enabled: false,
              },
            },
            enableMouseTracking: false,
            zIndex: 1000,
          },
          {
            name: 'PAW Incorrect',
            data: this.words.map(word => word.attention_incorrect),
            type: 'bar',
            color: '#ff6978',
            point: {
              events: {
                  click: self.handleChartClick,
              },
            },
          },
          ...options.series,
        ];
      }
      return options;
    },
  },
  watch: {
    onlyPIWs() {
      this.$refs.chart.chart.reflow();
    },
  },
}
</script>
