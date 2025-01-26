<template>
  <div class="game-container">
    <div class="help-button-container">
      <button @click="showInstructions = true" class="help-button">
        How to Play
      </button>
    </div>

    <h1 class="game-title">Turdle - Raise your Turtle</h1>

    <div class="main-content">
      <div class="side-panel turtle-panel">
        <div class="streak-counter">
          <p>Current Streak: {{ streak }}</p>
        </div>

        <div v-if="turtleLifeStage" class="turtle-status">
          <h2>Your Turtle</h2>
          <p class="stage-label">Stage: {{ turtleLifeStage }}</p>
          <div class="turtle-image-container">
            <img
              :src="lifeStagesInfo[turtleLifeStage].image"
              alt="Turtle Life Stage"
              class="turtle-image"
            />
          </div>
          <p class="turtle-description">
            {{ lifeStagesInfo[turtleLifeStage].description }}
          </p>
        </div>
      </div>

      <div class="game-panel">
        <div v-if="gameStatus === 'ongoing'" class="grid">
          <div v-for="(row, rowIndex) in attempts" :key="rowIndex" class="row">
            <div
              v-for="(letter, colIndex) in row"
              :key="colIndex"
              :class="`cell ${getCellClass(letter, rowIndex, colIndex)}`"
            >
              {{ letter || "" }}
            </div>
          </div>
        </div>

        <div class="input-section">
          <input
            v-model="currentGuess"
            type="text"
            maxlength="6"
            placeholder="Enter your guess"
            @keyup.enter="submitGuess"
            @input="validateInput"
            :disabled="gameOver"
          />
          <button
            @click="submitGuess"
            :disabled="gameOver"
            class="submit-button"
          >
            Submit
          </button>
        </div>
        <div v-if="errorMessage" class="error-message">
          {{ errorMessage }}
        </div>
      </div>

      <div class="side-panel status-panel">
        <div v-if="gameOver" class="word-details">
          <p v-if="guessedCorrectly" class="success-message">
            Congratulations, you guessed the word!
          </p>
          <p v-else class="failure-message">
            The word was: {{ correctWord }}. Better luck next time!
          </p>
          <button class="continue-button" @click="continueGame">
            Play Again
          </button>
        </div>

        <div v-if="wordDetails" class="word-details">
          <h3>About this word</h3>
          <div class="details-content">
            <p>
              <strong>Definition:</strong>
              {{ wordDetails.definition }}
            </p>
            <p v-if="wordDetails.example">
              <strong>Example:</strong>
              {{ wordDetails.example }}
            </p>
          </div>
        </div>
        <div v-if="!gameOver && !wordDetails" class="welcome-message">
          <h3>Welcome to Turdle! 🐢</h3>
          <div class="welcome-content">
            <ul>
              <li>You have 7 attempts to guess a 6-letter word</li>
              <li>Your turtle grows with each winning streak</li>
              <li>Click 'How to Play' for more details</li>
            </ul>
          </div>
        </div>
      </div>
    </div>

    <div v-if="showInstructions" class="modal-overlay">
      <div class="modal">
        <button class="close-button" @click="showInstructions = false">
          ×
        </button>
        <h2>How to Play</h2>
        <div class="instructions">
          <p>Guess the 6-letter word in 7 tries!</p>
          <ul>
            <li>Each guess must be a valid 6-letter word</li>
            <li>
              The color of the tiles will change to show how close your guess
              was:
            </li>
          </ul>
          <div class="example-tiles">
            <div class="example">
              <div class="cell correct">A</div>
              <span>Letter is in the word and in the correct spot</span>
            </div>
            <div class="example">
              <div class="cell present">B</div>
              <span>Letter is in the word but in the wrong spot</span>
            </div>
            <div class="example">
              <div class="cell absent">C</div>
              <span>Letter is not in the word</span>
            </div>
          </div>
          <h3>Turtle Evolution</h3>
          <p>
            Your turtle evolves as you build your winning streak! Can you reach
            the Elder stage?
          </p>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import confetti from "canvas-confetti";
export default {
  data() {
    return {
      showInstructions: false,
      attempts: Array(7)
        .fill()
        .map(() => Array(6).fill("")),
      currentGuess: "",
      correctWord: "",
      attemptCount: 0,
      gameOver: false,
      gameStatus: "ongoing",
      guessedCorrectly: false,
      streak: 0,
      gameKey: 0,
      wordDetails: null,
      errorMessage: "",
      turtleLifeStage: "Egg",
      lifeStagesInfo: {
        Egg: {
          image: require("@/assets/egg.png"),
          description: "The turtle is still in the egg. Keep playing!",
        },
        Hatchling: {
          image: require("@/assets/hatchling.png"),
          description:
            "The turtle has just hatched. It is starting to explore!",
        },
        Juvenile: {
          image: require("@/assets/juvenile.png"),
          description: "The turtle is growing, becoming more adventurous.",
        },
        Adult: {
          image: require("@/assets/adult.png"),
          description:
            "The turtle is now an adult, ready to take on challenges!",
        },
        Elder: {
          image: require("@/assets/elder.png"),
          description:
            "The turtle has reached the elder stage, full of wisdom.",
        },
      },
    };
  },
  methods: {
    validateInput() {
      this.currentGuess = this.currentGuess.replace(/[^a-zA-Z]/g, "");
    },
    async fetchWord() {
      try {
        const response = await fetch(
          "https://random-word-api.vercel.app/api?words=1&length=6"
        );
        const data = await response.json();
        this.correctWord = data[0].toUpperCase();
        // console.log("Correct word:", this.correctWord); // debugging
        const result = await this.validateWord(this.correctWord);
        const isValidWord = result.valid;
        if (!isValidWord) {
          throw new Error(
            "generated word does not belong in dictionary or api/network down"
          );
        }
      } catch (error) {
        console.error("Error fetching word:", error);
        //Fallback in case API fails
        const fallbackWords = [
          "TURTLE",
          "SILENT",
          "BRIGHT",
          "ABUSER",
          "SOLEMN",
          "CASTLE",
          "ZEPHYR",
          "DYNAMO",
          "ALLIES",
          "OXFORD",
          "OBLAST",
        ];
        this.correctWord =
          fallbackWords[Math.floor(Math.random() * fallbackWords.length)];
        console.log("Fallback word:", this.correctWord);
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
    getCellClass(letter, rowIndex, colIndex) {
      if (!letter || rowIndex > this.attemptCount) return "empty";

      const currentGuess = this.attempts[rowIndex].join("").toUpperCase();
      letter = letter.toUpperCase();

      if (currentGuess.length !== 6) return "empty";

      const correctPositions = new Array(6).fill(false);
      const usedIndices = new Array(6).fill(false);

      for (let i = 0; i < 6; i++) {
        if (currentGuess[i] === this.correctWord[i]) {
          correctPositions[i] = true;
          usedIndices[i] = true;
        }
      }

      if (correctPositions[colIndex]) {
        return "correct";
      }

      if (this.correctWord.includes(letter)) {
        let availableCount = 0;
        for (let i = 0; i < 6; i++) {
          if (!usedIndices[i] && this.correctWord[i] === letter) {
            availableCount++;
          }
        }

        let previousCount = 0;
        for (let i = 0; i < colIndex; i++) {
          if (currentGuess[i] === letter && !correctPositions[i]) {
            previousCount++;
          }
        }

        if (previousCount < availableCount) {
          return "present";
        }
      }

      return "absent";
    },

    async validateWord(word) {
      const apiUrl = `https://api.dictionaryapi.dev/api/v2/entries/en/${word}`;
      try {
        const response = await fetch(apiUrl);

        if (!response.ok) {
          if (response.status === 404) {
            //Word not found
            return {
              valid: false,
              message: "Not a valid English word.",
            };
          } else {
            //other server errors like 500
            return {
              valid: false,
              message:
                "The dictionary is currently unavailable. Try again later!",
            };
          }
        }

        const data = await response.json();
        return { valid: true, data };
      } catch (error) {
        return {
          valid: false,
          message: "Please check your connection and try again!",
        };
      }
    },
    async submitGuess() {
      if (this.currentGuess.length !== 6 || this.gameOver) return;

      this.currentGuess = this.currentGuess.toUpperCase();
      const result = await this.validateWord(this.currentGuess);
      const isValidWord = result.valid;
      const err = result.message;
      if (!isValidWord) {
        this.errorMessage = err;
        return;
      }

      this.errorMessage = "";

      const newAttempts = [...this.attempts];
      newAttempts[this.attemptCount] = this.currentGuess.split("");
      this.attempts = newAttempts;

      if (this.currentGuess === this.correctWord) {
        this.getCellClass();
        this.streak++;
        this.updateTurtleLifeStage();
        // Trigger confetti
        this.launchConfetti();
        await this.fetchWordDetails();
        this.guessedCorrectly = true;
        this.gameOver = true;
      } else {
        this.attemptCount++;

        if (this.attemptCount >= 7) {
          this.gameOver = true;
          this.guessedCorrectly = false;
          this.streak = 0; // reset streak on failure
          this.turtleLifeStage = "Egg";
          await this.fetchWordDetails();
        }
      }

      this.currentGuess = "";
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
      this.attempts = Array(7)
        .fill()
        .map(() => Array(6).fill(""));
      this.attemptCount = 0;
      this.gameOver = false;
      this.guessedCorrectly = false;
      this.gameStatus = "ongoing";
      this.wordDetails = null;
      this.currentGuess = "";
      this.fetchWord();
      this.gameKey++;
      this.errorMessage = "";
    },
    launchConfetti() {
      confetti({
        particleCount: 150,
        spread: 70,
        origin: { y: 0.6 },
      });
    },
  },
  mounted() {
    this.fetchWord();
  },
};
</script>

<style scoped>
.game-container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 1rem;
  min-height: 100vh;
  position: relative;
  font-family: "Fredoka", sans-serif;
}
.game-title {
  text-align: center;
  font-size: 2rem;
  margin-bottom: 1.5rem;
  color: #2c3e50;
}

.turtle-panel {
  background: #f8f9fa;
  border-radius: 0.75rem;
  padding: 1.25rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}
.error-message {
  text-align: center;
  padding: none;
  margin: none;
}

.streak-counter {
  text-align: center;
  font-size: 1.1rem;
  margin-bottom: 1rem;
}

.turtle-status {
  text-align: center;
}

.stage-label {
  font-size: 1.1rem;
  color: #2c3e50;
  margin: 0.75rem 0;
}

.turtle-image-container {
  background: white;
  border-radius: 50%;
  padding: 1rem;
  margin: 1rem auto;
  width: 150px;
  height: 150px;
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  background-image: url("@/assets/ocean.gif");
  background-size: cover;
  background-position: center;
}

.turtle-image {
  width: 120px;
  height: 120px;
  object-fit: contain;
}

.turtle-description {
  font-size: 0.9rem;
  line-height: 1.4;
  color: #4a5568;
}

.game-panel {
  background: #4299e1;
  border-radius: 0.75rem;
  padding: 1.5rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.grid {
  display: grid;
  grid-template-rows: repeat(7, 1fr);
  gap: 0.25rem;
  margin-bottom: 1.5rem;
  max-width: 360px;
  margin: 0 auto 1.5rem;
}

.row {
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  gap: 0.25rem;
}

.cell {
  aspect-ratio: 1;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.25rem;
  font-weight: bold;
  border: 2px solid #e2e8f0;
  border-radius: 0.25rem;
  transition: all 0.3s ease;
}

.input-section {
  display: flex;
  gap: 0.75rem;
  margin: 1.5rem auto;
  max-width: 360px;
}

input {
  flex: 1;
  padding: 0.5rem 0.75rem;
  font-size: 1rem;
  border: 2px solid #e2e8f0;
  border-radius: 0.25rem;
  transition: border-color 0.3s ease;
}

.submit-button,
.continue-button {
  padding: 0.5rem 1rem;
  font-size: 0.9rem;
}

.word-details {
  padding: 1rem;
  background: rgb(219, 232, 245);
  border-radius: 0.5rem;
  max-width: 360px;
}

.details-content {
  font-size: 0.9rem;
}

/* Responsive */
@media (max-width: 768px) {
  .main-content {
    grid-template-columns: 1fr;
  }
  .input-section {
    display: grid;
    margin: auto;
  }

  .game-container {
    padding: 0.75rem;
  }

  .turtle-panel,
  .game-panel {
    padding: 1rem;
  }
  .grid {
    max-width: 300px;
  }

  .cell {
    font-size: 1rem;
  }

  .input-section {
    max-width: 300px;
  }

  .word-details {
    max-width: 300px;
  }
}
.help-button-container {
  position: absolute;
  top: 1rem;
  right: 1rem;
}

.help-button {
  padding: 0.5rem 1rem;
  background: #4299e1;
  color: white;
  border: none;
  border-radius: 0.25rem;
  cursor: pointer;
  font-size: 0.9rem;
}

.help-button:hover {
  background: #3182ce;
}

.main-content {
  display: grid;
  grid-template-columns: 290px minmax(auto, 360px) 290px;
  gap: 3rem;
  align-items: start;
  justify-content: center;
  max-width: 1300px;
  margin: 0 auto;
  padding: 0 1rem;
}

.side-panel {
  background: #f8f9fa;
  background: rgb(219, 232, 245);
  border-radius: 0.75rem;
  padding: 1.25rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  height: fit-content;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
}

.modal {
  background: white;
  border-radius: 0.75rem;
  padding: 2rem;
  max-width: 500px;
  width: 90%;
  max-height: 90vh;
  overflow-y: auto;
  position: relative;
}

.close-button {
  position: absolute;
  top: 1rem;
  right: 1rem;
  font-size: 1.5rem;
  border: none;
  background: none;
  cursor: pointer;
  color: #4a5568;
}

.instructions {
  margin-top: 1rem;
}

.instructions ul {
  margin: 1rem 0;
  padding-left: 1.5rem;
}

.instructions li {
  margin: 0.5rem 0;
}

.example-tiles {
  margin: 1rem 0;
}

.example {
  display: flex;
  align-items: center;
  gap: 1rem;
  margin: 0.5rem 0;
}

.example .cell {
  width: 2.5rem;
  height: 2.5rem;
  font-size: 1.25rem;
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

@media (max-width: 1024px) {
  .main-content {
    grid-template-columns: 1fr;
    max-width: 500px;
    margin: 0 auto;
  }

  .help-button-container {
    position: static;
    text-align: right;
    margin-bottom: 1rem;
  }

  .side-panel {
    max-width: none;
  }

  .modal {
    width: 95%;
    margin: 1rem;
  }
}
</style>
