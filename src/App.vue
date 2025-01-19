<template>
  <div class="game-container">
    <h1>Wordle</h1>
    <div class="grid">
      <!-- Loop over attempts (max 8) -->
      <div v-for="(row, rowIndex) in attempts" :key="rowIndex" class="row">
        <!-- Loop over each letter in the row (6 letters per row) -->
        <div
          v-for="(letter, colIndex) in row"
          :key="colIndex"
          :class="`cell ${getCellClass(letter)}`"
        >
          {{ letter || "" }}
        </div>
      </div>
    </div>

    <!-- Guess input section -->
    <input
      v-model="currentGuess"
      type="text"
      maxlength="6"
      placeholder="Enter your guess"
      @keyup.enter="submitGuess"
      @input="validateInput"
      :disabled="gameOver"
    />
    <button @click="submitGuess" :disabled="gameOver">Submit Guess</button>

    <div v-if="gameOver">
      <p v-if="guessedCorrectly">Congratulations, you guessed the word!</p>
      <p v-else>The word was: {{ correctWord }}. Better luck next time!</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      attempts: Array(8).fill(Array(6).fill("")), // 8 rows, 6 columns (for a 6-letter word)
      currentGuess: "",
      correctWord: "",
      attemptCount: 0,
      gameOver: false,
      guessedCorrectly: false,
    };
  },
  methods: {
    validateInput() {
      // Remove numbers and spaces from the input
      this.currentGuess = this.currentGuess.replace(/[^a-zA-Z]/g, "");
    },
    async fetchWord() {
      try {
        const response = await fetch(
          "https://random-word-api.herokuapp.com/word?length=6"
        );
        const data = await response.json();
        this.correctWord = data[0].toUpperCase(); // Ensure it's a 6-letter word
      } catch (error) {
        console.error("Error fetching word:", error);
      }
    },
    getCellClass(letter) {
      if (!letter) return "empty";

      // Ensure attempts[this.attemptCount] is valid before accessing it
      const currentAttempt = this.attempts[this.attemptCount];

      // If it's the last attempt and no valid guess yet, return "empty"
      if (!currentAttempt || currentAttempt.length === 0) return "empty";

      if (this.correctWord.includes(letter)) {
        return this.correctWord.indexOf(letter) ===
          currentAttempt.indexOf(letter)
          ? "correct"
          : "present";
      }
      return "absent";
    },
    submitGuess() {
      this.currentGuess = this.currentGuess.toUpperCase();
      if (
        this.currentGuess.length === 6 &&
        this.attemptCount < 8 &&
        !this.gameOver
      ) {
        this.attempts[this.attemptCount] = this.currentGuess.split("");
        if (this.currentGuess === this.correctWord) {
          this.guessedCorrectly = true;
          this.gameOver = true;
        } else {
          this.attemptCount++;
        }
        this.currentGuess = "";
        if (this.attemptCount >= 8 && !this.guessedCorrectly) {
          this.gameOver = true;
        }
      }
    },
  },

  mounted() {
    this.fetchWord();
  },
};
</script>

<style scoped>
.game-container {
  text-align: center;
  background-color: rgb(61, 15, 15); /* Light grey color */
  font-family: Arial, sans-serif;
  display: flex;

  align-items: center; /* Center vertically */

  flex-direction: column; /* Stack elements vertically */
}

.grid {
  display: grid;
  grid-template-rows: repeat(8, 0fr); /* 8 rows */
  grid-template-columns: repeat(6, 0fr); /* 6 columns for a 6-letter word */
  gap: 2px;
  justify-items: center;
}

.row {
  display: contents; /* Ensure it doesn’t wrap each row but aligns inside the grid */
}

.cell {
  width: 50px;
  height: 50px;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 24px;
  border: 2px solid #ddd;
  transition: background-color 0.3s ease;
}

.empty {
  background-color: #f0f0f0;
}

.correct {
  background-color: green;
  color: white;
}

.present {
  background-color: yellow;
  color: black;
}

.absent {
  background-color: gray;
  color: white;
}

input {
  margin-top: 20px;
  padding: 10px;
  font-size: 18px;
  width: 200px;
}

button {
  margin-top: 10px;
  padding: 10px;
  font-size: 18px;
  background-color: #4caf50;
  color: white;
  border: none;
  cursor: pointer;
}

button:hover {
  background-color: #45a049;
}

div > p {
  font-size: 1.2em;
  margin-top: 20px;
  font-weight: bold;
}
</style>
