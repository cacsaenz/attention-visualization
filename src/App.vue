<template>
  <div id="app">
    <!-- TODO: Download button -->
    <h1 class="text-center mb-5">BERT-based models Interpretability</h1>
    <b-row class="align-items-center">
      <b-col cols="6" md="4" lg="3">
        <b-form-select v-model="top" :options="topOptions"></b-form-select>
      </b-col>
      <b-col cols="3">
        <b-form-checkbox class="d-inline-block" v-model="onlyPIWs">Only PIWs </b-form-checkbox>
        <span
          v-b-tooltip.hover
          title="Show only the words that POSITIVELY impact the model's predictions"
        >
          <b-icon
            class="ml-1"
            icon="question-circle"
            variant="dark"
          ></b-icon>
        </span>
      </b-col>
    </b-row>
    <b-row>
      <b-col :class="{ 'col-xs-7 col-md-8': onlyPIWs }">
        <Barchart
          :words="filteredWords"
          :onlyPIWs="onlyPIWs"
          @show-word-details="toggleWordDetails"
        ></Barchart>
      </b-col>
      <b-col class="col-xs-5 col-md-4" v-if="onlyPIWs">
        <Wordcloud :words="filteredWords"></Wordcloud>
      </b-col>
    </b-row>
    <b-modal
      size="xl"
      ref="modal"
      :hide-footer="true"
      title="Word details"
    >
      <b-container v-if="selectedWord !== null" fluid>
        <b-row class="align-items-center">
          <b-col sm="8" md="9">
            <b-row class="align-items-center justify-content-center">
              <b-col class="text-right" md="5">
                <h3 class="m-0">{{ selectedWord.word }}</h3>
              </b-col>
              <b-col>
                <div>
                  <p class="font-weight-bold m-0 text-primary d-inline-block">{{
                      `${(selectedWord.attention_correct / selectedWord.attention * 100).toFixed(1)}% (${(selectedWord.attention_correct).toFixed(1)} Positive PAW)`
                  }}</p>
                  <span
                    v-b-tooltip.hover
                    title="The attention weight of this word corresponding to correct predictions"
                  >
                    <b-icon
                      class="ml-1"
                      icon="question-circle"
                      variant="primary"
                    ></b-icon>
                  </span>
                </div>
                <div>
                  <p class="font-weight-bold m-0 text-danger d-inline-block">{{
                    `${(selectedWord.attention_incorrect / selectedWord.attention * 100).toFixed(1)}% (${(selectedWord.attention_incorrect).toFixed(1)} Negative PAW)`
                  }}</p>
                  <span
                    v-b-tooltip.hover
                    title="The attention weight of this word corresponding to incorrect predictions"
                  >
                    <b-icon
                      class="ml-1"
                      icon="question-circle"
                      variant="danger"
                    ></b-icon>
                  </span>
                </div>
              </b-col>
            </b-row>
            <b-row class="mt-4">
              <piechart :word="selectedWord"></piechart>
            </b-row>
          </b-col>
          <b-col sm="4" md="3">
            <div
              v-if="selectedWord.attention_correct >= selectedWord.threshold"
              class="text-center"
            >
              <div class="piw-symbol">✅</div>
              <div>This is a Positive Influential Word</div>
            </div>
            <div
              v-else
              class="text-center"
            >
              <div class="piw-symbol">❌</div>
              <div>This is NOT a Positive Influential Word</div>
            </div>
          </b-col>
        </b-row>
      </b-container>
    </b-modal>
  </div>
</template>

<script>
import * as d3 from 'd3';
import Barchart from './components/Barchart.vue';
import Piechart from './components/Piechart.vue';
import Wordcloud from './components/Wordcloud.vue';

export default {
  name: 'App',
  components: {
    Barchart,
    Wordcloud,
    Piechart,
  },
  data() {
    return {
      resultCounts: [],
      wordsMetrics: [],
      threshold: 0,
      onlyPIWs: false,
      top: 10,
      showModal: false,
      selectedWord: null,
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
        antichina_attention: parseFloat(data.antichina_attention),
        antivacina_attention: parseFloat(data.antivacina_attention),
        provacina_attention: parseFloat(data.provacina_attention),
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
    wordsMap() {
      const map = {};
      this.filteredWords.forEach((word) => (map[word.word] = word));
      return map;
    },
  },
  methods: {
    toggleWordDetails(word){
      this.selectedWord = this.wordsMap[word];
      this.$refs.modal.show();
    },
  },
}
</script>

<style>
body {
  padding: 2rem;
}

.piw-symbol {
  font-size: 2.5rem;
}
</style>