<template>
  <div class="game-container">
    <div class="game-header">
      <div class="score">得分: {{ score }}</div>
      <button @click="toggleGame" :class="{ 'playing': isPlaying }">
        {{ isPlaying ? '暂停' : '开始' }}
      </button>
    </div>
    
    <div class="game-board" :style="boardStyle">
      <!-- 蛇身 -->
      <div
        v-for="(segment, index) in snake"
        :key="index"
        class="snake-segment"
        :style="getSegmentStyle(segment)"
      ></div>
      
      <!-- 食物 -->
      <div
        class="food"
        :style="getFoodStyle()"
      ></div>
    </div>

    <div class="game-over" v-if="gameOver">
      <h2>游戏结束</h2>
      <p>最终得分: {{ score }}</p>
      <button @click="resetGame">重新开始</button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Snake',
  data() {
    return {
      boardSize: 20, // 棋盘大小
      cellSize: 20, // 每个格子的大小（像素）
      snake: [], // 蛇身坐标数组
      direction: 'right', // 移动方向
      food: { x: 0, y: 0 }, // 食物坐标
      score: 0, // 得分
      isPlaying: false, // 游戏状态
      gameOver: false, // 游戏是否结束
      gameInterval: null, // 游戏循环间隔
      speed: 150, // 移动速度（毫秒）
    }
  },
  computed: {
    boardStyle() {
      return {
        width: `${this.boardSize * this.cellSize}px`,
        height: `${this.boardSize * this.cellSize}px`,
        gridTemplateColumns: `repeat(${this.boardSize}, ${this.cellSize}px)`,
      }
    },
  },
  methods: {
    initGame() {
      // 初始化蛇的位置（从中心开始）
      const centerPos = Math.floor(this.boardSize / 2)
      this.snake = [
        { x: centerPos, y: centerPos },
        { x: centerPos - 1, y: centerPos },
        { x: centerPos - 2, y: centerPos },
      ]
      this.direction = 'right'
      this.generateFood()
      this.score = 0
      this.gameOver = false
    },
    generateFood() {
      // 随机生成食物位置
      let newFood
      do {
        newFood = {
          x: Math.floor(Math.random() * this.boardSize),
          y: Math.floor(Math.random() * this.boardSize),
        }
      } while (this.snake.some(segment => segment.x === newFood.x && segment.y === newFood.y))
      this.food = newFood
    },
    moveSnake() {
      if (!this.isPlaying || this.gameOver) return

      const head = { ...this.snake[0] }
      
      // 根据方向移动蛇头
      switch (this.direction) {
        case 'up': head.y--; break
        case 'down': head.y++; break
        case 'left': head.x--; break
        case 'right': head.x++; break
      }

      // 检查碰撞
      if (this.checkCollision(head)) {
        this.endGame()
        return
      }

      // 添加新的蛇头
      this.snake.unshift(head)

      // 检查是否吃到食物
      if (head.x === this.food.x && head.y === this.food.y) {
        this.score += 10
        this.generateFood()
        // 增加速度
        if (this.speed > 50) {
          this.speed -= 5
          this.updateGameInterval()
        }
      } else {
        // 如果没有吃到食物，移除蛇尾
        this.snake.pop()
      }
    },
    checkCollision(head) {
      // 检查是否撞墙
      if (
        head.x < 0 ||
        head.x >= this.boardSize ||
        head.y < 0 ||
        head.y >= this.boardSize
      ) {
        return true
      }

      // 检查是否撞到自己
      return this.snake.some(segment => segment.x === head.x && segment.y === head.y)
    },
    toggleGame() {
      if (this.gameOver) {
        this.resetGame()
        return
      }

      this.isPlaying = !this.isPlaying
      if (this.isPlaying) {
        this.updateGameInterval()
      } else {
        clearInterval(this.gameInterval)
      }
    },
    updateGameInterval() {
      clearInterval(this.gameInterval)
      this.gameInterval = setInterval(() => {
        this.moveSnake()
      }, this.speed)
    },
    endGame() {
      this.gameOver = true
      this.isPlaying = false
      clearInterval(this.gameInterval)
    },
    resetGame() {
      this.initGame()
      this.speed = 150
      this.isPlaying = true
      this.updateGameInterval()
    },
    getSegmentStyle(segment) {
      return {
        left: `${segment.x * this.cellSize}px`,
        top: `${segment.y * this.cellSize}px`,
        width: `${this.cellSize}px`,
        height: `${this.cellSize}px`,
      }
    },
    getFoodStyle() {
      return {
        left: `${this.food.x * this.cellSize}px`,
        top: `${this.food.y * this.cellSize}px`,
        width: `${this.cellSize}px`,
        height: `${this.cellSize}px`,
      }
    },
    handleKeydown(event) {
      if (!this.isPlaying) return

      const key = event.key
      let newDirection = this.direction

      switch (key) {
        case 'ArrowUp':
          if (this.direction !== 'down') newDirection = 'up'
          break
        case 'ArrowDown':
          if (this.direction !== 'up') newDirection = 'down'
          break
        case 'ArrowLeft':
          if (this.direction !== 'right') newDirection = 'left'
          break
        case 'ArrowRight':
          if (this.direction !== 'left') newDirection = 'right'
          break
      }

      if (newDirection !== this.direction) {
        this.direction = newDirection
      }
    },
  },
  mounted() {
    this.initGame()
    window.addEventListener('keydown', this.handleKeydown)
  },
  beforeUnmount() {
    window.removeEventListener('keydown', this.handleKeydown)
    clearInterval(this.gameInterval)
  },
}
</script>

<style scoped>
.game-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
}

.game-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  width: 400px;
}

.score {
  font-size: 24px;
  font-weight: bold;
  color: #2c3e50;
}

button {
  padding: 10px 20px;
  font-size: 16px;
  border: none;
  border-radius: 5px;
  background-color: #42b983;
  color: white;
  cursor: pointer;
  transition: background-color 0.3s;
}

button:hover {
  background-color: #3aa876;
}

button.playing {
  background-color: #e74c3c;
}

button.playing:hover {
  background-color: #c0392b;
}

.game-board {
  position: relative;
  background-color: #fff;
  border: 2px solid #2c3e50;
  border-radius: 5px;
}

.snake-segment {
  position: absolute;
  background-color: #42b983;
  border-radius: 3px;
  transition: all 0.1s linear;
}

.food {
  position: absolute;
  background-color: #e74c3c;
  border-radius: 50%;
  transition: all 0.1s linear;
}

.game-over {
  position: fixed;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: rgba(0, 0, 0, 0.8);
  padding: 30px;
  border-radius: 10px;
  text-align: center;
  color: white;
}

.game-over h2 {
  font-size: 32px;
  margin-bottom: 20px;
}

.game-over p {
  font-size: 24px;
  margin-bottom: 30px;
}

.game-over button {
  background-color: #42b983;
  font-size: 20px;
  padding: 15px 30px;
}
</style>
