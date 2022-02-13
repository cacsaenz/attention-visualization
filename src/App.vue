<template>
  <div id="app">
    <!-- TODO: Download button -->
    <!-- TODO: README tooltip -->
    <h1 class="text-center mb-5">BERT-based models Interpretability</h1>
    <b-row class="align-items-center">
      <b-col cols="6" md="4" lg="3">
        <b-form-select v-model="top" :options="topOptions"></b-form-select>
      </b-col>
      <b-col cols="3">
        <b-form-checkbox v-model="onlyPIWs">Only PIWs</b-form-checkbox>
      </b-col>
    </b-row>
    <b-row>
      <b-col :class="{ 'col-xs-7 col-md-8': onlyPIWs }">
        <Barchart :words="filteredWords" :onlyPIWs="onlyPIWs"></Barchart>
      </b-col>
      <b-col class="col-xs-5 col-md-4" v-if="onlyPIWs">
        <Wordcloud :words="filteredWords"></Wordcloud>
      </b-col>
    </b-row>
    <!-- <pre>
      <code>
        {{ filteredWords }}
      </code>
    </pre> -->
  </div>
</template>

<script>
import * as d3 from 'd3';
import Barchart from './components/Barchart.vue';
import Wordcloud from './components/Wordcloud.vue';

export default {
  name: 'App',
  components: {
    Barchart,
    Wordcloud,
  },
  data() {
    return {
      resultCounts: [],
      wordsMetrics: [],
      threshold: 0,
      onlyPIWs: false,
      top: 10,
      topOptions: [
        {
          value: null,
          text: 'Select a number of top words',
        },
        {
          value: 10,
          text: '10',
        },
        {
          value: 15,
          text: '15',
        },
        {
          value: 20,
          text: '20',
        },
        {
          value: 25,
          text: '25',
        },
        {
          value: 30,
          text: '30',
        },
        {
          value: 35,
          text: '35',
        },
        {
          value: 40,
          text: '40',
        },
        {
          value: 45,
          text: '45',
        },
        {
          value: 50,
          text: '50',
        },
      ],
    };
  },
  async mounted() {
    const counts = {};
    await d3.csv('./data/result_counts.csv', (data) => {
      counts[data.name] = parseFloat(data.count)
    });
    this.resultCounts = counts;

    this.threshold = (
        this.resultCounts['antichina_correct'] +
        this.resultCounts['antivacina_correct'] +
        this.resultCounts['provacina_correct']
      ) / this.resultCounts['total'];

    const wordsMetrics = await d3.csv('./data/words_metrics.csv', (data) => {
      return {
        word: data.word,
        attention: parseFloat(data.attention),
        attention_correct: parseFloat(data.attention_correct),
        attention_incorrect: parseFloat(data.attention_incorrect),
        threshold: this.threshold * parseFloat(data.attention),
      };
    });
    this.wordsMetrics = wordsMetrics;
  },
  computed: {
    filteredWords() {
      if (this.top === null) {
        return [];
      }
      let words = this.wordsMetrics;
      if (this.onlyPIWs) {
        return words.filter(
          (word) => word.attention_correct > word.threshold
        ).sort((wordA, wordB) => {
          return wordB.attention_correct - wordA.attention_correct;
        }).slice(0, this.top);
      }
      return words.sort((wordA, wordB) => {
        return wordB.attention - wordA.attention;
      }).slice(0, this.top);
    },
  },
}
</script>

<style>
body {
  padding: 2rem;
}
</style>