<!-- eslint-disable vue/multi-word-component-names -->
// eslint-disable-next-line vue/multi-word-component-names
<template>
  <div class="players-container">
    <div v-if="!gameStarted" class="player-setup">
      <!-- Header for player selection -->
      <h2>Select Player Colors and Names</h2>
      
      <!-- Loop through each player and display name input and color selector -->
      <div class="player-selection" v-for="(player, index) in players" :key="index">
        <!-- Input for player name -->
        <input v-model="player.name" placeholder="Enter Player Name" />
        
        <!-- Dropdown for selecting color -->
        <select v-model="player.color">
          <option value="">Choose a color</option>
          <!-- Loop through color options and disable selected ones -->
          <option 
            v-for="color in availableColors" 
            :key="color" 
            :value="color" 
            :disabled="isColorSelected(color)"
          >
            {{ color }}
          </option>
        </select>
      </div>
      
      <!-- Start game button -->
      <button 
        @click="$emit('start-game')" 
        :disabled="!allPlayersReady" 
        class="start-game-btn"
      >
        Start Game
      </button>
    </div>
  </div>
</template>

<script>
export default {
  props: ["players", "gameStarted"],
  data() {
    return {
      availableColors: ["red", "blue", "green", "yellow"], // List of available colors
    };
  },
  computed: {
    // Check if all players are ready (name and color selected)
    allPlayersReady() {
      return this.players.every(player => player.name && player.color);
    },
  },
  methods: {
    // Check if a color is already selected by another player
    isColorSelected(color) {
      return this.players.some(player => player.color === color);
    },
  },
};
</script>

<style scoped>
/* General styles for the container */
.players-container {
  padding: 20px;
  text-align: center;
}

.player-setup {
  display: flex;
  flex-direction: column;
  align-items: center;
}

/* Style the header */
h2 {
  font-size: 2rem;
  font-weight: 500;
  color: #029c9c; /* Adjusted color to make it softer */
  text-align: center;
  margin-bottom: 20px;
  text-shadow: 0 0 10px rgba(5, 89, 146, 0.6);
  animation: glow 1.5s infinite alternate;
}

@keyframes glow {
  0% {
    text-shadow: 0 0 10px rgba(107, 220, 255, 0.6);
  }
  100% {
    text-shadow: 0 0 20px rgb(0, 26, 255);
  }
}

/* Styling the player selection inputs */
.player-selection {
  display: flex;
  flex-direction: column;
  align-items: center;
}

input, select {
  padding: 8px 12px;
  margin: 3px 0;
  border-radius: 4px;
  border: 1px solid #ccc;
  font-size: 1rem;
  width: 180px;
  background-color: #f0f0f0;
  transition: border-color 0.3s ease;
}

input:focus, select:focus {
  border-color: #019262; /* Greenish color on focus */
  outline: none;
}

/* Styling for the start game button */
.start-game-btn {
  background-color: #019262; /* Soft green */
  color: white;
  font-size: 1.2rem;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s ease;
  margin-top: 20px;
}

.start-game-btn:disabled {
  background-color: #ccc; /* Disabled button style */
  cursor: not-allowed;
}

/* Change button color on hover */
.start-game-btn:hover:not(:disabled) {
  background-color: #017f57; /* Slightly darker green on hover */
}
</style>
