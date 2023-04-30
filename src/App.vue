<template>
  <div class="game">
    <div>
      Очки: <span>{{ this.score }}</span>
    </div>
    <div>
      Уровень: <span>{{ this.currentLevel }}</span>
    </div>
    <button class="btn btn-start" @click="btnStart">Старт</button>
    <button class="btn btn-pause" @click="btnPause">Пауза</button>
    <div class="main" ref="main"></div>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      // playfild: Array(20).fill(Array(10).fill(0)),
      playfild: [
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
      ],
      mainInnerHTML: "",
      canRemove: true,
      filledLines: 0,
      gameTimerID: null,
      activeTetro: [],
      isPaused: true,
      score: 0,
      currentLevel: 1,
      possiblelevels: {
        1: {
          scorePerLine: 10,
          speed: 1000,
          nextLevelScore: 50,
        },
        2: {
          scorePerLine: 15,
          speed: 700,
          nextLevelScore: 150,
        },
        3: {
          scorePerLine: 20,
          speed: 500,
          nextLevelScore: 300,
        },
        4: {
          scorePerLine: 30,
          speed: 400,
          nextLevelScore: 1000,
        },
        5: {
          scorePerLine: 60,
          speed: 250,
          nextLevelScore: 2000,
        },
        6: {
          scorePerLine: 100,
          speed: 200,
          nextLevelScore: 5000,
        },
        6: {
          scorePerLine: 150,
          speed: 150,
          nextLevelScore: Infinity,
        },
      },
      figures: {
        O: [
          [1, 1],
          [1, 1],
        ],
        I: [
          [0, 1, 0, 0],
          [0, 1, 0, 0],
          [0, 1, 0, 0],
          [0, 1, 0, 0],
        ],
        S: [
          [0, 1, 1],
          [1, 1, 0],
          [0, 0, 0],
        ],
        Z: [
          [1, 1, 0],
          [0, 1, 1],
          [0, 0, 0],
        ],
        L: [
          [1, 0, 0],
          [1, 1, 1],
          [0, 0, 0],
        ],
        J: [
          [0, 0, 1],
          [1, 1, 1],
          [0, 0, 0],
        ],
        T: [
          [1, 1, 1],
          [0, 1, 0],
          [0, 0, 0],
        ],
      },
    };
  },
  methods: {
    removePrevActiveTetro() {
      for (let y = 0; y < this.playfild.length; y++) {
        for (let x = 0; x < this.playfild[y].length; x++) {
          if (this.playfild[y][x] === 1) {
            this.playfild[y][x] = 0;
          }
        }
      }
    },
    addActiveTetro() {
      this.removePrevActiveTetro();
      for (let y = 0; y < this.activeTetro.shape.length; y++) {
        for (let x = 0; x < this.activeTetro.shape[y].length; x++) {
          if (this.activeTetro.shape[y][x] === 1) {
            this.playfild[this.activeTetro.y + y][this.activeTetro.x + x] =
              this.activeTetro.shape[y][x];
          }
        }
      }
    },
    rotateTetro() {
      const prevTetroState = this.activeTetro.shape;
      this.activeTetro.shape = this.activeTetro.shape[0].map((val, index) =>
        this.activeTetro.shape.map((row) => row[index]).reverse()
      );
      if (this.hasCollision()) {
        this.activeTetro.shape = prevTetroState;
      }
    },
    hasCollision() {
      for (let y = 0; y < this.activeTetro.shape.length; y++) {
        for (let x = 0; x < this.activeTetro.shape[y].length; x++) {
          if (
            this.activeTetro.shape[y][x] &&
            (this.playfild[this.activeTetro.y + y] === undefined ||
              this.playfild[this.activeTetro.y + y][this.activeTetro.x + x] ===
                undefined ||
              this.playfild[this.activeTetro.y + y][this.activeTetro.x + x] ===
                2)
          ) {
            return true;
          }
        }
      }
      return false;
    },
    draw() {
      this.mainInnerHTML = "";
      for (let y = 0; y < this.playfild.length; y++) {
        for (let x = 0; x < this.playfild[y].length; x++) {
          if (this.playfild[y][x] === 1) {
            this.mainInnerHTML += '<div class="main__cell movingCell"></div>';
          } else if (this.playfild[y][x] === 2) {
            this.mainInnerHTML += '<div class="main__cell fixedCell"></div>';
          } else {
            this.mainInnerHTML += '<div class="main__cell"></div>';
          }
        }
      }
      this.$refs.main.innerHTML = this.mainInnerHTML;
    },
    dropTetro() {
      for (let y = this.activeTetro.y; y < this.playfild.length; y++) {
        this.activeTetro.y += 1;
        if (this.hasCollision()) {
          this.activeTetro.y -= 1;
          break;
        }
      }
    },
    removeFullLines() {
      for (let y = 0; y < this.playfild.length; y++) {
        for (let x = 0; x < this.playfild[y].length; x++) {
          if (this.playfild[y][x] !== 2) {
            this.canRemove = false;
            break;
          }
        }
        if (this.canRemove) {
          this.playfild.splice(y, 1);
          this.playfild.splice(0, 0, [0, 0, 0, 0, 0, 0, 0, 0, 0, 0]);
          this.filledLines += 1;
        }
        this.canRemove = true;
      }
      switch (this.filledLines) {
        case 1:
          this.score += this.possiblelevels[this.currentLevel].scorePerLine;
          break;
        case 2:
          this.score += this.possiblelevels[this.currentLevel].scorePerLine * 3;
          break;
        case 3:
          this.score += this.possiblelevels[this.currentLevel].scorePerLine * 6;
          break;
        case 4:
          this.score +=
            this.possiblelevels[this.currentLevel].scorePerLine * 12;
          break;
      }
      if (this.score >= this.possiblelevels[this.currentLevel].nextLevelScore) {
        this.currentLevel++;
      }
      this.filledLines = 0;
    },
    moveTetroDown() {
      this.activeTetro.y += 1;
      if (this.hasCollision()) {
        this.activeTetro.y -= 1;
        this.fixTetro();
        this.removeFullLines();
        this.activeTetro = this.getNewTetro();
        if (this.hasCollision()) {
          alert("ПРОИГРЫШ");
          this.reset();
        }
      }
    },
    reset() {
      this.isPaused = true;
      clearTimeout(this.gameTimerID);
      this.playfild = [
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
      ];
      this.draw();
    },
    getNewTetro() {
      const possibleFigures = "OILJSZT";
      const rand = Math.floor(Math.random() * 7);
      const newTetro = this.figures[possibleFigures[rand]];
      // return this.figures[possibleFigures[rand]];
      return {
        x: Math.floor((10 - newTetro[0].length) / 2),
        y: 0,
        shape: newTetro,
      };
    },
    fixTetro() {
      for (let y = 0; y < this.playfild.length; y++) {
        for (let x = 0; x < this.playfild[y].length; x++) {
          if (this.playfild[y][x] === 1) {
            this.playfild[y][x] = 2;
          }
        }
      }
    },
    startGame() {
      this.moveTetroDown();
      if (!this.isPaused) {
        this.updateGameState();
        this.gameTimerID = setTimeout(
          this.startGame,
          this.possiblelevels[this.currentLevel].speed
        );
      }
    },
    btnPause(e) {
      if (e.target.innerHTML === "Пауза") {
        e.target.innerHTML = "Продолжить";
        clearTimeout(this.gameTimerID);
      } else {
        e.target.innerHTML = "Пауза";
        this.gameTimerID = setTimeout(
          this.startGame,
          this.possiblelevels[this.currentLevel].speed
        );
      }
      this.isPaused = !this.isPaused;
    },
    btnStart() {
      this.isPaused = false;
      this.gameTimerID = setTimeout(
        this.startGame,
        this.possiblelevels[this.currentLevel].speed
      );
    },
    updateGameState() {
      if (!this.isPaused) {
        this.addActiveTetro();
        this.draw();
      }
    },
  },
  mounted() {
    this.activeTetro = this.getNewTetro();
    document.addEventListener("keydown", (e) => {
      if (!this.isPaused) {
        if (e.keyCode === 37) {
          this.activeTetro.x -= 1;
          if (this.hasCollision()) {
            this.activeTetro.x += 1;
          }
        } else if (e.keyCode === 39) {
          this.activeTetro.x += 1;
          if (this.hasCollision()) {
            this.activeTetro.x -= 1;
          }
        } else if (e.keyCode === 40) {
          this.moveTetroDown();
        } else if (e.keyCode === 38) {
          this.rotateTetro();
        } else if (e.keyCode === 32) {
          this.dropTetro();
        }

        this.updateGameState();
      }
    });
    this.draw();
  },
};
</script>

<style lang="less">
* {
  box-sizing: border-box;
}
body {
  margin: 0;
  padding: 0;
  background: black;
  color: #fff;
  font-family: Arial, "Helvetica Neue", Helvetica, sans-serif;
  font-weight: 600;
}
.game {
  padding-top: 40px;
  width: 320px;
  margin: 0 auto;
  font-size: 26px;
}
.main {
  width: 320px;
  height: 620px;
  margin: 40px auto 0 auto;
  border: 10px solid silver;
  font-size: 0;
  &__cell {
    display: inline-block;
    width: 30px;
    height: 30px;
    border: 1px solid rgba(6, 159, 247, 0.219);
  }
}
.movingCell {
  background: blue;
}
.fixedCell {
  background: green;
}
.btn-start {
  margin-right: 15px;
}
.btn {
  border: none;
  margin-top: 15px;
  padding: 10px 15px;
  font-weight: 600;
  color: rgb(23, 104, 7);
  border-radius: 5px;
  transition: all 100ms ease;
  &:hover {
    transform: scale(0.95);
    background: rgb(221, 221, 221);
  }
}
</style>
