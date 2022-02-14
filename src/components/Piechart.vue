<template>
  <div>
    <highcharts :options="chartOptions"></highcharts>
  </div>
</template>

<script>
import { Chart } from 'highcharts-vue';

export default {
  name: 'Piechart',
  components: {
    highcharts: Chart,
  },
  props: {
    word: Object,
  },
  computed: {
    chartOptions() {
      const totalAttention = (
        this.word.antichina_attention + this.word.antivacina_attention + this.word.provacina_attention
      );
      return {
        chart: {
          type: 'pie',
          height: '250px',
        },
        title: {
          text: 'Predominance on each class',
        },
        tooltip: {
          pointFormat: '{series.name}: <b>{point.percentage:.1f}%</b>'
        },
        series: [{
          name: 'Stances',
          colorByPoint: true,
          data: [
            {
              name: 'Anti-sinovaxxers',
              y: this.word.antichina_attention / totalAttention * 100,
              color: '#4053d3',
            },
            {
              name: 'Anti-vaxxers',
              y: this.word.antivacina_attention / totalAttention * 100,
              color: '#ddb310'
            },
            {
              name: 'Pro-vaxxers',
              y: this.word.provacina_attention / totalAttention * 100,
              color: '#b51d14',
            },
          ],
        }],
      };
    },
  },
  watch: {
    onlyPIWs() {
      this.$refs.chart.chart.reflow();
    },
  },
}
</script>
