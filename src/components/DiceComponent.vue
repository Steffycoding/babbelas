<template>
  <div class="dice-container">
    <div class="dice">
      <span
        v-for="dot in dots[currentNumber - 1]"
        :key="dot"
        :class="['dot', dot]"
      ></span>
    </div>
    <button class="roll-btn" @click="rollDice" :disabled="rolling">Roll Dice</button>
    <p>Total: {{ currentNumber }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      currentNumber: 1,
      rolling: false, // Add this line to track the rolling state
      dots: [
        ["center"], // 1
        ["top-left", "bottom-right"], // 2
        ["top-left", "center", "bottom-right"], // 3
        ["top-left", "top-right", "bottom-left", "bottom-right"], // 4
        ["top-left", "top-right", "center", "bottom-left", "bottom-right"], // 5
        ["top-left", "top-right", "middle-left", "middle-right", "bottom-left", "bottom-right"], // 6
      ],
    };
  },
  methods: {
    rollDice() {
      if (this.rolling) return; // Prevent multiple rolls if dice is already rolling

      this.rolling = true; // Set rolling flag to true

      const rollSound = new Audio('/src/assets/dice-roll.mp3'); // Roll sound effect
      rollSound.play();

      // Interval to simulate dice rolling by changing the dice value every 50ms
      const rollInterval = setInterval(() => {
        this.currentNumber = Math.floor(Math.random() * 6) + 1; // Random dice value between 1 and 6
      }, 50);

      // After 1 second, stop the rolling animation and set final dice value
      setTimeout(() => {
        rollSound.pause();
        clearInterval(rollInterval); // Stop the rolling interval
        this.rolling = false; // Reset the rolling flag

        const result = Math.floor(Math.random() * 6) + 1;
        this.currentNumber = result;
        this.$emit("roll", result);
      }, 1000); // Stop rolling after 1 second
    },
  },
};
</script>

<style scoped>
/* Container for the dice and button */
.dice-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-top: 20px;
}

/* Dice styling */
.dice {
  width: 100px; /* Increased dice size */
  height: 100px; /* Increased dice size */
  display: flex;
  justify-content: center;
  align-items: center;
  background: var(--dice-background-color, #f0f0f0); /* Use theme background color */
  border: 3px solid var(--dice-border-color, #005f52); /* Use theme border color */
  border-radius: 10px;
  position: relative;
}

/* Dot styling */
.dot {
  width: 14px; /* Increased dot size */
  height: 14px; /* Increased dot size */
  background: var(--dot-color,#005f52); /* Use theme dot color */
  border-radius: 50%;
  position: absolute;
}

/* Dot positions based on dice values */
.dot.center {
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.dot.top-left {
  top: 20%;
  left: 20%;
}

.dot.top-right {
  top: 20%;
  right: 20%;
}

.dot.bottom-left {
  bottom: 20%;
  left: 20%;
}

.dot.bottom-right {
  bottom: 20%;
  right: 20%;
}

.dot.middle-left {
  top: 50%;
  left: 20%;
  transform: translateY(-50%);
}

.dot.middle-right {
  top: 50%;
  right: 20%;
  transform: translateY(-50%);
}

/* Roll Dice Button styling */
.roll-btn {
  margin-top: 20px;
  padding: 15px 30px; /* Increased button size */
  background-color: var(--button-bg-color, #028559); /* Use theme background color */
  color: white;
  border: none;
  cursor: pointer;
  font-size: 18px; /* Increased font size */
  border-radius: 8px; /* Rounded corners */
  transition: background-color 0.3s ease;
}

/* Disabled button styling */
.roll-btn:disabled {
  background-color: #ccc;
  cursor: not-allowed;
}

/* Button hover effect */
.roll-btn:hover:not(:disabled) {
  background-color: var(--button-hover-color, #3a9d42); /* Use theme hover color */
}

/* Dice total result text styling */
p {
  font-size: 22px; /* Increased font size */
  margin-top: 15px;
  color: var(--text-color, #333); /* Use theme text color */
}
</style>
