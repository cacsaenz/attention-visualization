<template>
  <div class="wordcloud-container w-100 h-100">
    <!-- <p class="text-center">Barchart</p> -->
    <!-- BUILD HERE THE INFO -->
    <!-- <highcharts :options="chartOptions"></highcharts> -->
    <vue-word-cloud
      :words="parsedWords"
      :font-family="'Ubuntu'"
      :spacing="1/4"
      :color="getColor"
    ></vue-word-cloud>
  </div>
</template>

<script>
import VueWordCloud from 'vuewordcloud';

export default {
  name: 'Wordcloud',
  components: {
    'vue-word-cloud': VueWordCloud,
  },
  data() {
    return {
      colors: ['#f39c12', '#3bc4c7', '#3a9eea', '#ff4e69', '#461e47'],
    };
  },
  props: {
    words: Array,
  },
  computed: {
    parsedWords() {
      return this.words.map(wordData => [wordData.word, wordData.attention]);
    },
  },
  methods: {
    getColor() {
      const random = Math.floor(Math.random() * this.colors.length);
      return this.colors[random];
    },
  },
}
</script>

<style>
.wordcloud-container {
  min-height: 400px;
}
</style>
