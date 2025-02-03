<template>
  <div :class="['game-board-wrapper', isDarkMode ? 'dark-mode' : 'light-mode']">
    <!-- Left Side: Game Board -->
    <div class="game-board-container">
      <h1 class="game-title">Babbelas</h1>

      <!-- Game Board Grid -->
      <div class="game-board-grid">
        <div
          v-for="(number, index) in numbers"
          :key="index"
          :class="['grid-item', isPlayerPosition(number) ? 'player-position' : '']"
        >
          {{ number }}
          <!-- Player Indicator -->
          <div
            v-for="(player, playerIndex) in getPlayersOnPosition(number)"
            :key="playerIndex"
            :style="{ backgroundColor: player.color }"
            class="player-indicator"
            @mouseover="showPlayerName(player)"
            @mouseleave="clearPlayerName"
          >
            {{ player.name.charAt(0) }}
          </div>
        </div>
      </div>
    </div>

    <!-- Right Side: Player Info and Dice -->
    <div class="game-right-container">
      <Players :players="players" :gameStarted="gameStarted" @start-game="startGame" />

      <!-- Dice Roll and Current Player Info -->
      <div v-if="gameStarted" class="dice-container">
        <DiceComponent @roll="handleDiceRoll" :disabled="isDiceButtonDisabled" />
      </div>

      <div v-if="gameStarted" class="current-player-info">
        <p class="current-player-text">Current Player: 
          <span class="current-player-name">{{ players[currentPlayerIndex].name }}</span>
        </p>
      </div>

      <!-- Player Stats -->
      <div v-if="gameStarted" class="player-stats">
        <h2>Player Stats</h2>
        <div v-for="(player, index) in players" :key="index" class="player-stat">
          <div class="player-indicator" :style="{ backgroundColor: player.color }">
            {{ player.name.charAt(0) }}
          </div>
          <p>{{ player.name }}</p>
          <p>Moves: {{ player.moves }}</p>
          <p>Score: {{ player.score }}</p>
        </div>
      </div>

      <!-- End Game Button -->
      <button v-if="gameStarted" @click="endGame" class="end-game-btn">End Game</button>

      <!-- Game Results -->
      <div v-if="gameEnded" class="game-results">
        <h2>Game Over</h2>
        <div v-for="(player, index) in sortedPlayers" :key="index" class="player-result">
          <p><span class="rank">{{ getRank(index) }}</span> {{ player.name }} - Score: {{ player.score }}</p>
        </div>
      </div>
    </div>

    <!-- Light/Dark Mode Toggle -->
    <img 
      src="@/assets/light.png" 
      v-show="!isDarkMode" 
      alt="Light Mode" 
      class="mode-toggle-icon" 
      @click="toggleMode" 
    />
    <img 
      src="@/assets/dark.png" 
      v-show="isDarkMode" 
      alt="Dark Mode" 
      class="mode-toggle-icon" 
      @click="toggleMode" 
    />
  </div>
</template>

<script>
import Players from "@/components/Players.vue";
import DiceComponent from "@/components/DiceComponent.vue";

export default {
  name: "GameBoard",
  components: { Players, DiceComponent },
  data() {
    return {
      players: Array(4).fill().map(() => ({ name: "", color: "", score: 0, moves: 0 })), 
      currentPlayerIndex: 0,
      gameStarted: false,
      gameEnded: false,
      diceRoll: null,
      isDarkMode: false,
      numbers: Array.from({ length: 100 }, (_, index) => index + 1),
      playerPositions: Array(4).fill(1),
      hoverPlayerName: "",
      boingSound: new Audio(new URL("@/assets/boing.mp3", import.meta.url).href),
      isDiceButtonDisabled: false,
    };
  },
  computed: {
    getPlayersOnPosition() {
      return (number) => this.players.filter((_, index) => this.playerPositions[index] === number);
    },
    sortedPlayers() {
      return [...this.players].sort((a, b) => b.score - a.score);
    },
  },
  methods: {
    getRank(index) {
      return ["1st", "2nd", "3rd", "4th"][index] || `${index + 1}th`;
    },
    startGame() {
      this.players.forEach(player => Object.assign(player, { score: 0, moves: 0 }));
      this.playerPositions.fill(1);
      this.gameStarted = true;
      this.gameEnded = false;
      this.currentPlayerIndex = 0;
      this.isDiceButtonDisabled = false;
    },
    handleDiceRoll(rollResult) {
      if (this.gameStarted && this.playerPositions[this.currentPlayerIndex] < 100) {
        this.diceRoll = rollResult;
        this.movePlayer();
      }
    },
    movePlayer() {
      const currentPos = this.playerPositions[this.currentPlayerIndex]; 
      const targetPosition = currentPos + this.diceRoll;

      if (targetPosition <= 100) {
        this.animatePlayerMovement(currentPos, targetPosition);
      } else {
        this.endTurn();
      }
    },
    animatePlayerMovement(currentPos, targetPosition) {
      this.boingSound.volume = 0.5;
      this.boingSound.playbackRate = 0.5;

      const move = () => {
        if (currentPos < targetPosition) {
          this.playerPositions[this.currentPlayerIndex] = ++currentPos;
          this.boingSound.currentTime = 0;
          this.boingSound.play().catch(console.error);

          setTimeout(() => {
            requestAnimationFrame(move);
          }, 200);
        } else {
          this.players[this.currentPlayerIndex].score = currentPos;
          this.endTurn();
        }
      };
      requestAnimationFrame(move);
    },
    endTurn() {
      this.players[this.currentPlayerIndex].moves++;
      if (this.diceRoll !== 6 || this.playerPositions[this.currentPlayerIndex] === 100) {
        this.resetPlayerTurn();
      }
      this.checkGameOver(); 
    },
    resetPlayerTurn() {
      do {
        this.currentPlayerIndex = (this.currentPlayerIndex + 1) % this.players.length;
      } while (this.playerPositions[this.currentPlayerIndex] === 100);
    },
    checkGameOver() {
      if (this.players.filter((_, index) => this.playerPositions[index] === 100).length == 3) {
        this.endGame();
      }
    },
    endGame() {
      this.gameEnded = true;
      this.gameStarted = false;
      this.players = this.sortedPlayers();
    },
    isPlayerPosition(number) {
      return this.playerPositions.includes(number);
    },
    showPlayerName(player) {
      this.hoverPlayerName = player.name;
    },
    clearPlayerName() {
      this.hoverPlayerName = "";
    },
    toggleMode() {
      this.isDarkMode = !this.isDarkMode;
    }
  },
};
</script>


<style scoped>
/* Set the background for the entire page */
.game-board-wrapper {
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
  background-color: #f0f0f0; /* Light background color for the whole page */
}


.mode-toggle-icon {
  position: absolute;
  top: 10px;
  left: 10px;
  cursor: pointer;
  width: 40px;
  height: 40px;
  transition: transform 0.3s ease-in-out;
}

/* Left side: Game Board */
.game-board-container {
  flex: 1;
  padding: 20px;
  background-color: #ffffff; /* Set the left side's background to white or your preferred color */
}

.game-board-container.dark-mode {
  flex: 1;
  padding: 20px;
  background-color: #4d4d4d; /* Set the left side's background to white or your preferred color */
}


/* Right side: Player Info and Dice */
.game-right-container {
  width: 300px;
  padding: 10px;
  background-color: #f4f4f4; /* Keep the right side background color unchanged */
  border-left: 1px solid #ddd;
  font-family: 'Roboto', sans-serif;
  display: flex;
  flex-direction: column;
}

/* Right side: Player Info and Dice */
.game-right-container.dark-mode {
  width: 300px;
  padding: 10px;
  background-color: #4b4b4b; /* Keep the right side background color unchanged */
  border-left: 1px solid #535d64;
  font-family: 'Roboto', sans-serif;
  display: flex;
  flex-direction: column;
}

/* Optional: If you want to adjust the background color for dark mode */
.game-board-wrapper.dark-mode {
  background-color: #6f7274; /* Dark background for the whole page in dark mode */
}

.game-board-container.dark-mode {
  background-color: #444; /* Adjust the left side's background for dark mode */
}

.game-title {
  font-size: 4rem;
  font-weight: 500;
  color: #019262;
  text-align: center;
  text-shadow: 0 0 10px rgba(5, 89, 146, 0.6);
  animation: glow 1.5s infinite alternate;
  margin-bottom: 20px;
}

.game-title.dark-mode {
  font-size: 4rem;
  font-weight: 500;
  color: #019262;
  text-align: center;
  text-shadow: 0 0 10px rgba(5, 89, 146, 0.6);
  animation: glow 1.5s infinite alternate;
  margin-bottom: 20px;
}

@keyframes glow {
  0% {
    text-shadow: 0 0 10px rgba(107, 220, 255, 0.6);
  }
  100% {
    text-shadow: 0 0 20px rgb(0, 26, 255);
  }
}

.game-board-grid {
  display: grid;
  grid-template-columns: repeat(10, 1fr);
  gap: 5px;
}

.game-board-grid.dark-mode {
  display: grid;
  grid-template-columns: repeat(10, 1fr);
  gap: 5px;
}

.grid-item {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 50px;
  border: 1px solid #ddd;
}

.grid-item.dark-mode {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 50px;
  border: 1px solid #8ca09d;
}

.player-position {
  background-color: rgba(0, 128, 0, 0.2);
}

.player-position.dark-mode {
  background-color: rgba(0, 128, 0, 0.2);
}

.player-indicator {
  width: 18px;
  height: 18px;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  color: white;
  font-size: 12px;
  font-weight: bold;
}

.player-indicator.dark-mode {
  width: 18px;
  height: 18px;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  color: rgb(116, 126, 124);
  font-size: 12px;
  font-weight: bold;
}

/* Styling the current player */
.current-player-info {
  font-size: 1.2rem;
  font-weight: bold;
  color: #022558b4;
  margin: 20px 0;
}

.current-player-info.dark-mode {
  font-size: 1.2rem;
  font-weight: bold;
  color: #305285b4;
  margin: 20px 0;
}

.current-player-name {
  font-size: 1.4rem;
  font-weight: bold;
}

.current-player-name.dark-mode {
  font-size: 1.4rem;
  font-weight: bold;
}


.current-player-name.highlight {
  color: #019262; /* Green color to highlight the current player */
  background-color: #e5f8e5; /* Light green background for highlight */
  padding: 5px;
  border-radius: 5px;
}

/* Player Indicator */
.player-indicator {
  width: 25px; /* Adjusted size */
  height: 25px;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 14px; /* Increased font size */
  font-weight: bold;
  color: rgba(0, 0, 0, 0.651);
  background-color: #019262; /* Default color for indicator */
  margin-right: 15px; /* Space between indicator and player name */
  box-shadow: 0 0 10px rgba(0, 5, 2, 1.3); /* Subtle shadow for better visibility */
  
  /* Pulsing effect */
  animation: pulse 1.5s infinite ease-in-out; /* Apply the pulse animation */
}

/* Player Indicator */
.player-indicator.darkmode {
  width: 25px; /* Adjusted size */
  height: 25px;
  border-radius: 50%;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 14px; /* Increased font size */
  font-weight: bold;
  color: rgba(0, 0, 0, 0.651);
  background-color: #019262; /* Default color for indicator */
  margin-right: 15px; /* Space between indicator and player name */
  box-shadow: 0 0 10px rgba(0, 5, 2, 1.3); /* Subtle shadow for better visibility */
  
  /* Pulsing effect */
  animation: pulse 1.5s infinite ease-in-out; /* Apply the pulse animation */
}

/* Keyframes for pulsing effect */
@keyframes pulse {
  0% {
    transform: scale(1); /* Normal size */
    opacity: 1; /* Full opacity */
  }
  50% {
    transform: scale(1.2); /* Slightly bigger */
    opacity: 0.7; /* Slightly transparent */
  }
  100% {
    transform: scale(1); /* Return to normal size */
    opacity: 1; /* Full opacity */
  }
}

/* Hover effect for player indicators */
.player-indicator:hover {
  transform: scale(1.1); /* Slightly enlarge the indicator on hover */
  box-shadow: 0 0 15px rgba(0, 128, 0, 0.6); /* Increase shadow on hover */
  cursor: pointer;
}


/* Player name and stats container */
.player-stat {
  display: flex;
  align-items: center;
  margin-bottom: 15px; /* Add spacing between player stats */
}

.player-stat.dark-mode {
  display: flex;
  align-items: center;
  margin-bottom: 15px; /* Add spacing between player stats */
}

/* Player name Styling */
.player-stat p:first-child {
  font-weight: 700; /* Stronger weight for better readability */
  font-size: 1.2rem; /* Larger font size for player name */
  color: #019262; /* Green color for the player's name */
  text-transform: capitalize;
  margin-right: 10px; /* Space between name and stats */
}


/* Moves and Score Styling */
.player-stat p:nth-child(2),
.player-stat p:nth-child(3) {
  color: #4a4a4a; /* Darker color for contrast */
  margin-left: 15px; /* Space between stats */
}

/* Additional styling for the player stats section */
.player-stats {
  margin-top: 30px; /* Space before stats section */
  font-family: 'Roboto', sans-serif; /* Ensure consistent font */
}

/* Hover effect for player stats */
.player-stat:hover {
  background-color: rgba(0, 128, 0, 0.1); /* Light green background on hover */
  border-radius: 5px;
  padding: 10px; /* Add padding on hover for better interaction */
}

/* Player Stats Section Header */
.player-stats h2 {
  font-size: 1.5rem;
  font-weight: 700;
  color: #019262;
  margin-bottom: 20px;
}

.player-stats h2.dark-mode {
  font-size: 1.5rem;
  font-weight: 700;
  color: #095239;
  margin-bottom: 20px;
}

/* Player Stats Details */
.player-stats p {
  margin: 0;
  font-size: 1.2rem;
  color: #333;
  font-weight: 600;
}

/* End Game Button */
.end-game-btn {
  background-color: #ff4444;
  color: white;
  padding: 10px;
  border: none;
  cursor: pointer;
  border-radius: 5px;
  font-size: 1rem;
  font-weight: bold;
  transition: background-color 0.3s ease;
}

.end-game-btn.dark-mode {
  background-color: #ff4444;
  color: white;
  padding: 10px;
  border: none;
  cursor: pointer;
  border-radius: 5px;
  font-size: 1rem;
  font-weight: bold;
  transition: background-color 0.3s ease;
}

.end-game-btn:hover {
  background-color: #550a0a;
}

/* Dice Container */
.dice-container {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 20px;
}

</style>
