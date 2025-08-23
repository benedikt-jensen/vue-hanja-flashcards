<template>
  <h2>{{ currentLanguage === 'korean' ? 'Study Hanja' : 'Study German A1' }}</h2>
  
  <!-- Language Selection -->
  <div style="margin-bottom: 12px">
    <base-button
      @click="switchLanguage('korean')"
      style="margin-right: 4px"
      :style="{ backgroundColor: currentLanguage === 'korean' ? '#007bff' : '' }"
      >Korean Hanja</base-button
    >
    <base-button
      @click="switchLanguage('german')"
      :style="{ backgroundColor: currentLanguage === 'german' ? '#007bff' : '' }"
      >German A1</base-button
    >
  </div>

  <!-- Vocabulary Set Selection -->
  <div>
    <div style="margin-bottom: 4px">
      <base-button
        v-for="i in 5"
        :key="'choose_' + i"
        @click="chooseVocabSet(i)"
        style="margin-right: 4px"
        >{{ i }}</base-button
      >
    </div>
    <div style="margin-bottom: 4px">
      <base-button
        v-for="i in [6, 7, 8]"
        :key="'choose_' + i"
        @click="chooseVocabSet(i)"
        style="margin-right: 4px"
        >{{ i }}</base-button
      >
    </div>
  </div>
  
  <div>
    <flash-card
      :word="vocab[currentVocabSet][currentVocabIndex]"
      :reverse="reverse"
      @nextWord="nextWord"
      @storeData="storeData"
    ></flash-card>
  </div>
  <div style="margin-top: 4px">
    <base-button @click="reverse = !reverse">{{
      getFlipButtonText()
    }}</base-button>
  </div>
</template>

<script>
import FlashCard from "./components/FlashCard.vue";
import arrays from "./util.js";
// import basic_hanja from "./assets/vocab/basic_hanja.txt";
import howtolearnkorean_vocab from "./assets/vocab/howtolearnkorean_com_vocab.txt";
import german_a1_vocab from "./assets/vocab/german_a1_vocab.txt";
// import hanja_junior from "./assets/vocab/hanja_junior_translation.txt";
// import hanja_senior from "./assets/vocab/hanja_senior.txt";

// const REPS = 5;
const MIN_SET_SIZE = 2; // Must >= 2
const MAX_SET_SIZE = 30; // Actual max size is pow(2, MAX_SET_SIZE)

export default {
  name: "App",
  components: {
    FlashCard,
  },
  data() {
    return {
      vocab: [],
      currentLanguage: 'korean', // 'korean' or 'german'
      currentVocabSet: 1,
      currentSubSet: [0, MIN_SET_SIZE],
      currentSubSetShuffled: arrays.range(0, MIN_SET_SIZE),
      currentVocabIndex: 0,
      currentSubSetIndex: 0,
      reverse: false,
      madeError: false,
      progress: Array(MAX_SET_SIZE).fill(0),
      depth: 0,
    };
  },
  mounted() {
    this.chooseVocabSet(1);
  },
  async created() {
    this.loadVocabulary();
  },
  watch: {
    currentSubSetIndex(value) {
      this.currentVocabIndex = this.currentSubSetShuffled[value];
    },
  },
  methods: {
    loadVocabulary() {
      this.vocab = [];
      
      if (this.currentLanguage === 'korean') {
        let vocabSetSize = 40;
        for (let i = 0; i < 10; i++) {
          let vocabSet = this.getVocabulary(
            howtolearnkorean_vocab
              .split("\n")
              .slice(i * vocabSetSize, (i + 1) * vocabSetSize)
              .map((line) => line.split(" : "))
              .map(
                (line) =>
                  line[0] + " (" + line[2] + " - " + line[1] + " - " + line[3] + ")"
              )
              .join("\n")
          );
          this.vocab.push(vocabSet);
        }
      } else if (this.currentLanguage === 'german') {
        let vocabSetSize = 40;
        // Count total German vocabulary entries
        const germanLines = german_a1_vocab.split("\n").filter(line => line.trim() !== "");
        const totalSets = Math.ceil(germanLines.length / vocabSetSize);
        
        for (let i = 0; i < totalSets; i++) {
          let vocabSet = this.getVocabulary(
            germanLines
              .slice(i * vocabSetSize, (i + 1) * vocabSetSize)
              .map((line) => line.split(" : "))
              .map(
                (line) =>
                  line[0] + " (" + line[3] + ")"
              )
              .join("\n")
          );
          this.vocab.push(vocabSet);
        }
      }
    },

    switchLanguage(language) {
      this.currentLanguage = language;
      this.loadVocabulary();
      this.chooseVocabSet(1); // Reset to first set
    },

    getFlipButtonText() {
      if (this.currentLanguage === 'korean') {
        return this.reverse ? "Meaning → Hanja" : "Hanja → Meaning";
      } else {
        return this.reverse ? "English → German" : "German → English";
      }
    },

    chooseVocabSet(num) {
      this.currentVocabSet = num - 1;
      this.currentSubSet = [0, MIN_SET_SIZE];
      this.currentSubSetShuffled = arrays.range(0, MIN_SET_SIZE);
      this.currentVocabIndex = 0;
      this.currentSubSetIndex = 0;
      this.depth = 0;
      this.progress.fill(0);
      this.madeError = false;
    },
    getVocabulary(vocabListString) {
      const lines = vocabListString.split("\n").map((line) => line.trim());
      this.fileContent = lines.filter((line) => line.trim() !== ""); // Filter out empty lines
      return lines;
    },
    // newDataObj() {
    //   const data = {
    //     0: new Array(REPS).fill([]),
    //     1: new Array(REPS).fill([]),
    //     2: new Array(REPS).fill([]),
    //     3: new Array(REPS).fill([]),
    //     4: new Array(REPS).fill([]),
    //   };
    //   for (let i = 0; i < REPS; i++) {
    //     for (let j = 0; j < 100; j++) {
    //       data[i][0].push(j);
    //     }
    //   }
    //   return data;
    // },
    storeData(guessedRight) {
      guessedRight == true;
      // if (guessedRight) {
      //   for (let i = 0; i < REPS; i++) {
      //     if (this.progressData[set][i].has(index)) {
      //       if (i < REPS - 1) {
      //         this.progressData[set][i + 1].add(index);
      //         this.progressData[set][i].delete(index);
      //       }
      //     }
      //   }
      // } else {
      //   for (let i = 0; i < REPS; i++) {
      //     if (this.progressData[set][i].has(index)) {
      //       this.progressData[set][i].delete(index);
      //       this.progressData[set][0].add(index);
      //     }
      //   }
      // }
    },

    nextWord(knewWord) {
      if (!knewWord) {
        this.madeError = true;
      } 
      if (this.currentSubSetIndex >= this.currentSubSetShuffled.length - 1) {
        if (this.madeError) {
          this.madeError = false;
        } else {
          if (MIN_SET_SIZE * Math.pow(2, this.depth) >= this.vocab[this.currentVocabSet].length) {
            alert("You now know all hanja in this set!")
          }
          this.nextSet();
        }

        // shuffle subset
        let currentWord = Number(this.currentSubSetShuffled.slice(-1));
        do {
          this.currentSubSetShuffled = arrays.range(
            this.currentSubSet[0],
            this.currentSubSet[1]
          );
          arrays.shuffleArray(this.currentSubSetShuffled);
        } while (currentWord === this.currentSubSetShuffled[0]);
        this.currentSubSetIndex = 0;
      } else {
        this.currentSubSetIndex++;
      }
    },

    nextSet() {
      this.madeError = false;
      this.progress[this.depth]++;
      if (this.progress[this.depth] % 2 == 0 || this.currentSubSet[1]==this.vocab[this.currentVocabSet].length) {
        this.depth++;
        let newSetSize = MIN_SET_SIZE * Math.pow(2, this.depth);
        this.currentSubSet[0] = newSetSize * this.progress[this.depth];
        this.currentSubSet[1] = this.currentSubSet[0] + newSetSize;
      } else {
        this.depth = 0;
        this.currentSubSet[0] = MIN_SET_SIZE * this.progress[this.depth];
        this.currentSubSet[1] = this.currentSubSet[0] + MIN_SET_SIZE;
      }
      // Don't go beyond array index bounds
      if (this.currentSubSet[1] > this.vocab[this.currentVocabSet].length) {
        this.currentSubSet[1] = this.vocab[this.currentVocabSet].length;
      }
      // // Set subset start to 0 when done
      // if (
      //   this.currentSubSet[0] >=
      //   this.vocab[this.currentVocabSet].length - 1
      // ) {
      //   this.currentSubSet[0] = 0;
      // }
    },

    // nextWord() {
    //   let vocabset = this.vocab[this.currentVocabSet];

    //   this.currentVocabIndex = Math.floor(Math.random() * vocabset.length);
    // },

    randomWord() {
      let vocabset = this.vocab[this.currentVocabSet];

      // for (let i = 0; i<REPS; i++) {
      //   if (vocabset[i].size > 0) {
      //     return vocabset[i].values().next();
      //   }
      // }

      return vocabset[this.currentWordIndex];
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
