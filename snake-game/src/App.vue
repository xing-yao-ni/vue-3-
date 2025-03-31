<template>
  <div class="game-wrapper">
    <div class="game-container">
      <div class="game-header">
        <h1>贪吃蛇游戏</h1>
        <div class="score-container">
          <div class="score">得分: {{ score }}</div>
          <div class="high-score">最高分: {{ highScore }}</div>
        </div>
      </div>
      <div class="game-board" 
           ref="gameBoard"
           @touchstart="handleTouchStart"
           @touchmove="handleTouchMove"
           @touchend="handleTouchEnd">
        <div v-for="(cell, index) in board" 
             :key="index" 
             :class="['cell', getCellClass(index)]">
        </div>
      </div>
      <div class="controls">
        <button @click="startGame" :disabled="isPlaying" class="start-button">
          {{ isPlaying ? '游戏中...' : '开始游戏' }}
        </button>
        <button @click="togglePause" v-if="isPlaying" class="pause-button">
          {{ isPaused ? '继续' : '暂停' }}
        </button>
        <button @click="restartGame" class="restart-button">重新开始</button>
      </div>
      <div class="difficulty-settings">
        <label>难度：</label>
        <select v-model="difficulty" @change="updateSpeed">
          <option value="easy">简单</option>
          <option value="medium">中等</option>
          <option value="hard">困难</option>
        </select>
      </div>
      <div class="instructions">
        <p>使用方向键 ↑ ↓ ← → 控制蛇的移动</p>
      </div>
    </div>
    <div class="game-over" v-if="gameOver">
      <div class="game-over-content">
        <h2>游戏结束！</h2>
        <p>最终得分: {{ score }}</p>
        <button @click="restartGame" class="restart-button">重新开始</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      board: Array(400).fill(null),
      snake: [],
      food: null,
      direction: 'right',
      isPlaying: false,
      isPaused: false,
      gameOver: false,
      score: 0,
      highScore: localStorage.getItem('snakeHighScore') || 0,
      gameInterval: null,
      speed: 600,
      difficulty: 'medium',
      touchStartX: 0,
      touchStartY: 0
    }
  },
  methods: {
    startGame() {
      clearInterval(this.gameInterval);
      this.resetGame();
      this.isPlaying = true;
      this.isPaused = false;
      this.gameOver = false;
      this.gameInterval = setInterval(this.moveSnake, this.speed);
      document.addEventListener('keydown', this.handleKeyPress);
    },
    togglePause() {
      if (this.isPaused) {
        this.gameInterval = setInterval(this.moveSnake, this.speed);
        this.isPaused = false;
      } else {
        clearInterval(this.gameInterval);
        this.isPaused = true;
      }
    },
    updateSpeed() {
      switch (this.difficulty) {
        case 'easy':
          this.speed = 600;
          break;
        case 'medium':
          this.speed = 400;
          break;
        case 'hard':
          this.speed = 200;
          break;
      }
      if (this.isPlaying && !this.isPaused) {
        clearInterval(this.gameInterval);
        this.gameInterval = setInterval(this.moveSnake, this.speed);
      }
    },
    restartGame() {
      this.gameOver = false;
      this.startGame();
    },
    resetGame() {
      this.snake = [0, 1, 2];
      this.direction = 'right';
      this.score = 0;
      this.board = Array(400).fill(null);
      this.generateFood();
    },
    generateFood() {
      let newFood;
      do {
        newFood = Math.floor(Math.random() * this.board.length);
      } while (this.snake.includes(newFood));
      this.food = newFood;
    },
    moveSnake() {
      const head = this.snake[this.snake.length - 1];
      let newHead;

      switch (this.direction) {
        case 'up':
          newHead = head - 20;
          break;
        case 'down':
          newHead = head + 20;
          break;
        case 'left':
          newHead = head - 1;
          break;
        case 'right':
          newHead = head + 1;
          break;
      }

      if (this.isCollision(newHead)) {
        this.endGame();
        return;
      }

      this.snake.push(newHead);

      if (newHead === this.food) {
        this.score += 10;
        if (this.score > this.highScore) {
          this.highScore = this.score;
          localStorage.setItem('snakeHighScore', this.highScore);
        }
        this.generateFood();
      } else {
        this.snake.shift();
      }
    },
    isCollision(head) {
      if (this.direction === 'right' && head % 20 === 0) return true;
      if (this.direction === 'left' && head % 20 === 19) return true;
      if (this.direction === 'up' && head < 0) return true;
      if (this.direction === 'down' && head >= 400) return true;
      return this.snake.includes(head);
    },
    handleKeyPress(event) {
      const keyMap = {
        'ArrowUp': 'up',
        'ArrowDown': 'down',
        'ArrowLeft': 'left',
        'ArrowRight': 'right'
      };

      const newDirection = keyMap[event.key];
      if (!newDirection) return;

      const opposites = {
        'up': 'down',
        'down': 'up',
        'left': 'right',
        'right': 'left'
      };

      if (opposites[newDirection] !== this.direction) {
        this.direction = newDirection;
      }
    },
    endGame() {
      this.isPlaying = false;
      this.gameOver = true;
      clearInterval(this.gameInterval);
      document.removeEventListener('keydown', this.handleKeyPress);
    },
    getCellClass(index) {
      if (this.snake.includes(index)) return 'snake';
      if (index === this.food) return 'food';
      return '';
    },
    handleTouchStart(event) {
      this.touchStartX = event.touches[0].clientX;
      this.touchStartY = event.touches[0].clientY;
    },
    handleTouchMove(event) {
      event.preventDefault();
      const touchX = event.touches[0].clientX;
      const touchY = event.touches[0].clientY;
      
      const deltaX = touchX - this.touchStartX;
      const deltaY = touchY - this.touchStartY;
      
      // 设置最小滑动距离阈值
      const minSwipeDistance = 30;
      
      if (Math.abs(deltaX) > minSwipeDistance || Math.abs(deltaY) > minSwipeDistance) {
        if (Math.abs(deltaX) > Math.abs(deltaY)) {
          // 水平滑动
          if (deltaX > 0 && this.direction !== 'left') {
            this.direction = 'right';
          } else if (deltaX < 0 && this.direction !== 'right') {
            this.direction = 'left';
          }
        } else {
          // 垂直滑动
          if (deltaY > 0 && this.direction !== 'up') {
            this.direction = 'down';
          } else if (deltaY < 0 && this.direction !== 'down') {
            this.direction = 'up';
          }
        }
        
        // 更新起始点
        this.touchStartX = touchX;
        this.touchStartY = touchY;
      }
    },
    handleTouchEnd(event) {
      // 可以在这里添加触摸结束的处理逻辑
    }
  },
  beforeUnmount() {
    clearInterval(this.gameInterval);
    document.removeEventListener('keydown', this.handleKeyPress);
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  background: linear-gradient(-45deg, #ee7752, #e73c7e, #23a6d5, #23d5ab);
  background-size: 400% 400%;
  animation: gradientBG 15s ease infinite;
  min-height: 100vh;
  font-family: 'Arial', sans-serif;
  color: #fff;
  position: relative;
  overflow: hidden;
  margin: 0;
  padding: 0;
}

body::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: radial-gradient(circle at center, transparent 0%, rgba(0,0,0,0.3) 100%);
  pointer-events: none;
}

@keyframes gradientBG {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

.game-wrapper {
  width: 100vw;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  position: relative;
  padding: 10px;
}

.game-container {
  background: rgba(255, 255, 255, 0.15);
  backdrop-filter: blur(20px);
  border-radius: 20px;
  padding: 20px;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
  width: 100%;
  max-width: 400px;
  position: relative;
  border: 1px solid rgba(255, 255, 255, 0.2);
  transform-style: preserve-3d;
  perspective: 1000px;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 15px;
}

.game-container::before {
  content: '';
  position: absolute;
  top: -2px;
  left: -2px;
  right: -2px;
  bottom: -2px;
  background: linear-gradient(45deg, #ff6b6b, #4ecdc4, #45b7d1, #96c93d);
  border-radius: 32px;
  z-index: -1;
  animation: borderGlow 3s linear infinite;
  background-size: 300% 300%;
}

@keyframes borderGlow {
  0% { background-position: 0% 50%; }
  50% { background-position: 100% 50%; }
  100% { background-position: 0% 50%; }
}

.game-header {
  text-align: center;
  width: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;
}

h1 {
  font-size: 2em;
  margin-bottom: 10px;
  background: linear-gradient(45deg, #4CAF50, #45a049);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2);
  letter-spacing: 2px;
}

.score-container {
  display: flex;
  justify-content: center;
  gap: 15px;
  font-size: 1em;
  width: 100%;
}

.score, .high-score {
  background: rgba(255, 255, 255, 0.15);
  padding: 10px 20px;
  border-radius: 12px;
  min-width: 120px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
}

.game-board {
  display: grid;
  grid-template-columns: repeat(20, 15px);
  grid-template-rows: repeat(20, 15px);
  gap: 1px;
  background-color: rgba(255, 255, 255, 0.1);
  padding: 15px;
  border-radius: 15px;
  width: fit-content;
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.2);
  border: 1px solid rgba(255, 255, 255, 0.2);
  position: relative;
  place-self: center;
  margin: 0 auto;
  touch-action: none; /* 防止触摸事件的默认行为 */
  user-select: none; /* 防止文本选择 */
  -webkit-tap-highlight-color: transparent; /* 移除点击高亮 */
}

.cell {
  width: 15px;
  height: 15px;
  background-color: rgba(255, 255, 255, 0.05);
  border-radius: 2px;
  transition: all 0.2s ease;
  position: relative;
}

.cell::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  border-radius: 3px;
  box-shadow: inset 0 0 5px rgba(255, 255, 255, 0.1);
}

.snake {
  background: linear-gradient(45deg, #4CAF50, #45a049);
  box-shadow: 0 0 15px rgba(76, 175, 80, 0.6);
  animation: snakePulse 1.5s infinite;
  border-radius: 4px;
  z-index: 1;
}

.snake::after {
  box-shadow: inset 0 0 5px rgba(255, 255, 255, 0.3);
}

.food {
  background: linear-gradient(45deg, #ff6b6b, #f44336);
  border-radius: 50%;
  box-shadow: 0 0 15px rgba(244, 67, 54, 0.6);
  animation: foodPulse 1.5s infinite;
  z-index: 1;
}

.food::after {
  box-shadow: inset 0 0 5px rgba(255, 255, 255, 0.3);
}

.controls {
  margin-top: 15px;
  text-align: center;
  width: 100%;
  display: flex;
  justify-content: center;
  gap: 10px;
}

.start-button, .pause-button, .restart-button {
  padding: 12px 30px;
  font-size: 1em;
  background: linear-gradient(45deg, #4CAF50, #45a049);
  color: white;
  border: none;
  border-radius: 25px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(76, 175, 80, 0.3);
  text-transform: uppercase;
  letter-spacing: 1px;
  font-weight: bold;
}

.start-button:hover:not(:disabled), .pause-button:hover, .restart-button:hover {
  transform: translateY(-3px) scale(1.05);
  box-shadow: 0 6px 20px rgba(76, 175, 80, 0.4);
}

.start-button:disabled {
  background: #cccccc;
  cursor: not-allowed;
  transform: none;
  box-shadow: none;
}

.pause-button {
  background: linear-gradient(45deg, #ff9800, #f57c00);
  box-shadow: 0 4px 15px rgba(255, 152, 0, 0.3);
}

.pause-button:hover {
  transform: translateY(-3px) scale(1.05);
  box-shadow: 0 6px 20px rgba(255, 152, 0, 0.4);
}

.restart-button {
  background: linear-gradient(45deg, #2196F3, #1976D2);
  box-shadow: 0 4px 15px rgba(33, 150, 243, 0.3);
}

.restart-button:hover {
  transform: translateY(-3px) scale(1.05);
  box-shadow: 0 6px 20px rgba(33, 150, 243, 0.4);
}

.difficulty-settings {
  margin-top: 10px;
  display: flex;
  align-items: center;
  gap: 10px;
  color: rgba(255, 255, 255, 0.9);
  font-size: 0.9em;
}

.difficulty-settings select {
  background: rgba(255, 255, 255, 0.15);
  border: 1px solid rgba(255, 255, 255, 0.2);
  color: white;
  padding: 8px 15px;
  border-radius: 8px;
  cursor: pointer;
  font-size: 0.9em;
  backdrop-filter: blur(5px);
}

.difficulty-settings select:focus {
  outline: none;
  border-color: rgba(255, 255, 255, 0.4);
  box-shadow: 0 0 10px rgba(255, 255, 255, 0.1);
}

.difficulty-settings option {
  background: #2c3e50;
  color: white;
}

.game-over {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.8);
  display: flex;
  justify-content: center;
  align-items: center;
  backdrop-filter: blur(10px);
  z-index: 9999;
}

.game-over-content {
  background: rgba(255, 255, 255, 0.15);
  padding: 40px 60px;
  border-radius: 20px;
  text-align: center;
  animation: fadeIn 0.5s ease;
  border: 1px solid rgba(255, 255, 255, 0.2);
  box-shadow: 0 8px 32px rgba(0, 0, 0, 0.3);
  position: relative;
  overflow: hidden;
    display: flex;
  flex-direction: column;
  align-items: center;
  gap: 25px;
  min-width: 320px;
  backdrop-filter: blur(5px);
  margin: auto;
}

.game-over-content::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(45deg, rgba(255,255,255,0.1), rgba(255,255,255,0));
  pointer-events: none;
}

.game-over-content h2 {
  color: #ff6b6b;
  margin: 0;
  font-size: 2.5em;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.3);
}

.game-over-content p {
  font-size: 1.3em;
  margin: 0;
  color: #fff;
}

.game-over-content .restart-button {
  margin-top: 10px;
  padding: 15px 40px;
  font-size: 1.2em;
  background: linear-gradient(45deg, #4CAF50, #45a049);
  color: white;
  border: none;
  border-radius: 30px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(76, 175, 80, 0.3);
  text-transform: uppercase;
  letter-spacing: 1px;
  font-weight: bold;
}

.game-over-content .restart-button:hover {
  transform: translateY(-3px) scale(1.05);
  box-shadow: 0 6px 20px rgba(76, 175, 80, 0.4);
}

.instructions {
  margin-top: 15px;
  text-align: center;
  color: rgba(255, 255, 255, 0.8);
  font-size: 0.9em;
  background: rgba(255, 255, 255, 0.1);
  padding: 10px;
  border-radius: 8px;
  border: 1px solid rgba(255, 255, 255, 0.2);
  width: 100%;
}

.instructions p {
  margin-bottom: 5px;
}

.instructions p:last-child {
  margin-bottom: 0;
  font-size: 0.9em;
  color: rgba(255, 255, 255, 0.7);
}

.game-wrapper {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 0;
  margin: 0;
  overflow: hidden;
}

@keyframes snakePulse {
  0% { box-shadow: 0 0 5px rgba(76, 175, 80, 0.6); }
  50% { box-shadow: 0 0 20px rgba(76, 175, 80, 0.8); }
  100% { box-shadow: 0 0 5px rgba(76, 175, 80, 0.6); }
}

@keyframes foodPulse {
  0% { box-shadow: 0 0 5px rgba(244, 67, 54, 0.6); }
  50% { box-shadow: 0 0 20px rgba(244, 67, 54, 0.8); }
  100% { box-shadow: 0 0 5px rgba(244, 67, 54, 0.6); }
}

@keyframes fadeIn {
  from { 
    opacity: 0; 
    transform: scale(0.9) translateY(20px);
  }
  to { 
    opacity: 1; 
    transform: scale(1) translateY(0);
  }
}

@media (max-height: 700px) {
  .game-container {
    padding: 15px;
    gap: 8px;
  }

  .game-header {
    gap: 8px;
  }

  h1 {
    font-size: 1.8em;
    margin-bottom: 5px;
  }

  .game-board {
    padding: 12px;
  }

  .controls {
    margin-top: 8px;
  }

  .instructions {
    margin-top: 8px;
    padding: 8px;
  }
}

@supports (-webkit-touch-callout: none) {
  .game-wrapper {
    height: -webkit-fill-available;
  }
}
</style>
