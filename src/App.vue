<template>
  <div class="game-container">
    <h1>Wordle - Turtle Edition</h1>

    <!-- Display Turtle's life stage and image -->
    <p>Current Streak: {{ streak }}</p>
    <div v-if="turtleLifeStage" class="turtle-status">
      <p>Stage: {{ turtleLifeStage }}</p>
      <div class="turtle-image">
        <img
          :src="lifeStagesInfo[turtleLifeStage].image"
          alt="Turtle Life Stage"
        />
      </div>
      <p>{{ lifeStagesInfo[turtleLifeStage].description }}</p>
    </div>

    <div v-if="gameStatus === 'ongoing'" class="grid">
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
    <!-- Word details section -->
    <div v-if="wordDetails">
      <h3>About this word-</h3>
      <p>
        <strong>Definition:</strong>
        {{ wordDetails.definition }}
      </p>
      <p>
        <strong>Example Usage:</strong>
        {{ wordDetails.example || "No example available" }}
      </p>
    </div>
    <!-- Continue button -->
    <button class="continue-button" @click="continueGame">Continue</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      attempts: Array(8).fill(Array(6).fill("")),
      currentGuess: "",
      correctWord: "",
      attemptCount: 0,
      gameOver: false,
      gameStatus: "ongoing",
      guessedCorrectly: false,
      streak: 0, // Initialize streak
      gameKey: 0,
      wordDetails: null,
      turtleLifeStage: "Egg", // Start with "Egg"
      lifeStagesInfo: {
        Egg: {
          image: require("@/assets/egg.png"), // Replace with the actual path to the egg image
          description: "The turtle is still in the egg. Stay tuned!",
        },
        Hatchling: {
          image: require("@/assets/hatchling.png"), // Replace with the actual path to the hatchling image
          description:
            "The turtle has just hatched. It is starting to explore!",
        },
        Juvenile: {
          image: require("@/assets/juvenile.png"), // Replace with the actual path to the juvenile image
          description: "The turtle is growing, becoming more adventurous.",
        },
        Adult: {
          image: require("@/assets/adult.png"), // Replace with the actual path to the adult image
          description:
            "The turtle is now an adult, ready to take on challenges!",
        },
        Elder: {
          image: require("@/assets/elder.png"), // Replace with the actual path to the elder image
          description:
            "The turtle has reached the elder stage, full of wisdom.",
        },
      },
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
        console.log(this.correctWord);
      } catch (error) {
        console.error("Error fetching word:", error);
      }
    },
    async fetchWordDetails() {
      if (!this.correctWord) return;
      try {
        const response = await fetch(
          `https://api.dictionaryapi.dev/api/v2/entries/en/${this.correctWord}`
        );
        if (!response.ok) throw new Error("Word not found");

        const data = await response.json();
        // Assuming we only display the first meaning/definition
        const firstMeaning = data[0]?.meanings[0]?.definitions[0];
        this.wordDetails = {
          definition: firstMeaning?.definition || "Not available",
          example: firstMeaning?.example || null,
        };
      } catch (error) {
        console.error("Error fetching word details:", error);
        this.wordDetails = {
          definition: "No details found for this word.",
          example: null,
        };
      }
    },
    getCellClass(letter) {
      if (!letter) return "empty";

      const currentAttempt = this.attempts[this.attemptCount];

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

          this.streak++;
          this.fetchWordDetails();
          this.updateTurtleLifeStage(); // Update turtle life stage after correct guess
        } else {
          this.attemptCount++;
        }
        this.currentGuess = "";
        if (this.attemptCount >= 8 && !this.guessedCorrectly) {
          this.gameOver = true;
        }
      }
    },
    updateTurtleLifeStage() {
      if (this.streak <= 4) {
        this.turtleLifeStage = [
          "Egg",
          "Hatchling",
          "Juvenile",
          "Adult",
          "Elder",
        ][this.streak];
      }
    },
    continueGame() {
      console.log(this.streak);

      // Reset game state (but keep the streak and attempts structure)
      this.attempts = Array(8).fill(Array(6).fill("")); // Reset grid, but preserve structure
      this.wordDetails = null;
      this.currentGuess = ""; // Clear the current guess input
      this.gameStatus = "ongoing"; // Set game status back to ongoing
      this.gameOver = false;
      // Optionally, fetch a new word (if you want a new word every time)
      this.fetchWord();

      // Increment the gameKey to trigger re-render of the grid without resetting other states
      this.gameKey++;
    },
  },
  mounted() {
    this.fetchWord();
  },
};
</script>

<style scoped>
.game-container {
  position: relative;
  /*background-image: url("@/assets/ocean.gif");  Your GIF URL 
  background-size: cover;
  background-position: center;*/
  width: 100vw;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  color: black; /* Ensure text is visible over the background */
}
.word-details {
  margin-top: 20px;
  padding: 15px;
  border: 1px solid #ddd;
  border-radius: 5px;
  background-color: #f9f9f9;
}

.word-details h3 {
  margin-bottom: 10px;
}

.word-details p {
  margin: 5px 0;
}

.turtle-image img {
  width: 100px;
  height: auto;
  margin-top: 10px;
}

.turtle-description p {
  margin: 0;
  font-size: 16px;
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
.continue-button {
  margin-top: 10px;
  padding: 10px;
  font-size: 18px;
  background-color: #4caf50;
  color: white;
  border: none;
  cursor: pointer;
  transition: background-color 0.3s;
}

.continue-button:hover {
  background-color: #45a049;
}
div > p {
  font-size: 1.2em;
  margin-top: 20px;
  font-weight: bold;
}
</style>
