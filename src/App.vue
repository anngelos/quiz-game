<template>
  <div v-if="question">
    <ScoreBoard :winCount="this.winCount" :loseCount="this.loseCount"/>
    <template v-if="this.answers">
      <h1 v-html="question"></h1>
      <template v-for="(answer, i) in this.answers" :key="i">
        <input
          :disabled="this.answerSubmitted"
          type="radio"
          name="options"
          :value="answer"
          v-model="this.chosenAnswer"
        />
        <label v-html="answer"></label>
        <br/>
      </template>
      <br/>
      <button v-if="!this.answerSubmitted" @click="submitAnswer()" class="send" type="button">Send</button>
      
      <section v-if="this.answerSubmitted" class="result">
        <h5 v-if="this.chosenAnswer == this.correctAnswer" class="correct">✅ CORRECT ✅</h5>
        <h5 v-else class="incorrect">❌ INCORRECT ❌</h5>
        <button @click="getNewQuestion()" class="send" type="button">
          Next Question
        </button>
      </section>
    </template>
  </div>
  <div v-else class="spinner-border" role="status">
    <span class="visually-hidden">Loading...</span>
  </div>
</template>

<script>
import correctSound from "../public/correct-answer.wav";
import incorrectSound from "../public/incorrect-answer.mp3";
import ScoreBoard from "./components/ScoreBoard.vue"

export default {
  name: "App",
  components: {
    ScoreBoard
  },
  data() {
    return {
      question: undefined,
      incorrectAnswers: [],
      correctAnswer: undefined,
      chosenAnswer: null,
      answerSubmitted: false,
      winCount: 0,
      loseCount: 0,
      correctSound: new Audio(correctSound),
      incorrectSound: new Audio(incorrectSound),
    };
  },
  computed: {
    answers() {
      var answers = JSON.parse(JSON.stringify(this.incorrectAnswers));
      answers.splice(
        Math.round(Math.random() * answers.length),
        0,
        this.correctAnswer
      );
      return answers;
    },
  },
  methods: {
    submitAnswer() {
      if (!this.chosenAnswer) {
        alert("Escolha uma das alternativas!");
      } else {
        this.answerSubmitted = true;
        if (this.chosenAnswer == this.correctAnswer) {
          this.correctSound.play();
          this.winCount++
        } else {
          this.incorrectSound.play();
          this.loseCount++
        }
      }
    },
    getNewQuestion() {
      this.answerSubmitted = false;
      this.chosenAnswer = null;
      this.question = undefined;

      this.axios
        .get("https://opentdb.com/api.php?amount=1&category=18")
        .then((res) => {
          this.question = res.data.results[0].question;
          this.incorrectAnswers = res.data.results[0].incorrect_answers;
          this.correctAnswer = res.data.results[0].correct_answer;
        });
    },
  },
  created() {
    this.getNewQuestion();
  },
};
</script>

<style lang="scss">
@font-face {
  font-family: 'Aldo';
  src: url('~@/assets/fonts/Aldo.ttf');
}

#app {
  font-family: Aldo;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin: 60px auto;
  max-width: 960px;

  input[type="radio"] {
    margin: 12px 4px;
  }

  button.send {
    margin-top: 12px;
    height: 40px;
    min-width: 120px;
    padding: 0 16px;
    color: #fff;
    background-color: #1867c0;
    border: 1px solid #1867c0;
    cursor: pointer;
  }
}

.correct {
  color: #00D26A;
  font-weight: bold;
}

.incorrect {
  color: #F92F60;
  font-weight: bold;
}
</style>
