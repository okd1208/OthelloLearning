<template>
  <div>
    <div class="othello-board">
      <div v-for="row of 8" :key="row" class="row">
        <div v-for="col of 8" :key="col" class="col" @click="putDiscByUser(row,col)">
          <div :id="getClassName(row,col)"></div>
        </div>
      </div>
    </div>
    <div class="result-board" v-if="getResultText">
      <h2>Game Finish !</h2>
      <p>{{getResultText}}</p>
    </div>
  </div>
</template>

<script>
import BackendGateway from '@/mixins/backendGateway'
import { mapActions, mapGetters } from 'vuex'
export default {
  name: 'PlayBoard',
  mixins: [BackendGateway],
  data: function () {
    return {
      userColor: "white",
      userDiscCount: 0,
      enemyDiscCount: 0,
      isUserTurn: true,
      isAutoPut: true,
      turn: 0,
      lastPut: {
        row: -1,
        col: -1
      },
      putPositions: {
        white: {
          1:[],
          2:[],
          3:[],
          4:[4],
          5:[5],
          6:[],
          7:[],
          8:[],
        },
        black: {
          1:[],
          2:[],
          3:[],
          4:[5],
          5:[4],
          6:[],
          7:[],
          8:[],
        },
      },
      nextPutPositions: {
        white: {
          1:[],
          2:[],
          3:[],
          4:[],
          5:[],
          6:[],
          7:[],
          8:[],
        },
        black: {
          1:[],
          2:[],
          3:[],
          4:[],
          5:[],
          6:[],
          7:[],
          8:[],
        },
      },
      hitDiscsPosi: []
    }
  },
  methods: {
    ...mapActions(['saveGameID']),
    getClassName(row, col) {
      return `row${row}-col${col}`
    },
    allSet() {
      this.boardInit();
      const putPosi = this.putPositions;
      Object.keys(putPosi).forEach((color) => {
        const tmpRows = putPosi[color];
        Object.keys(tmpRows).forEach((row) => {
          tmpRows[row].forEach(col => {
            color === this.userColor ? this.userDiscCount += 1 : this.enemyDiscCount += 1;
            this.updateNextPosi(row, col, color);
            const element = document.getElementById(`row${row}-col${col}`);
            element.classList.add(color);
          })
        });
      });
    },
    boardInit() {
      this.userDiscCount = 0;
      this.enemyDiscCount = 0;
      const discs = document.querySelectorAll('.white, .black, .placeable-disk');
      discs.forEach(el => {
        el.classList.remove("white");
        el.classList.remove("black");
        el.classList.remove("placeable-disk");
      });
      Object.keys(this.nextPutPositions).forEach((color) => {
        const tmpRows = this.nextPutPositions[color];
        Object.keys(tmpRows).forEach((row) => {
          tmpRows[row] = []
        });
        this.nextPutPositions[color] = tmpRows;
      });
    },
    updateNextPosi(row, col, color) {
      row = Number(row);
      // console.log(`${color}:r${row}-c${col}`);
      const enemyColor = this.getReverseColor(color);
      for (let i = -1; i < 2 ; i++) {
        const r = row + i;
        if (r <= 0 || r > 8) {continue;}
        for (let n = -1; n < 2 ; n++) {
          const c = col + n;
          if (c <= 0 || c > 8){
            continue;
          }
          if (this.isExistDisc(r,c,enemyColor)) {
            if (row === r) {
              const lastCol = this.exploreHorizontal(r, c, enemyColor, c-col);
              if(lastCol) {
                this.nextPutPositions[color][r].push(lastCol);
              }
            } else if (col === c) {
              const lastRow = this.exploreVertical(r, c, enemyColor, r-row);
              if(lastRow) {
                this.nextPutPositions[color][lastRow].push(col);
              }
            } else {
              const lastPosi = this.exploreDiagonal(r, c, enemyColor, [r-row, c-col]);
              if(lastPosi["row"] && lastPosi["col"]) {
                this.nextPutPositions[color][lastPosi["row"]].push(lastPosi["col"]);
              }
            }
          }
        }
      }
    },
    updateNextPosiUI (color) {
      for (let row in this.nextPutPositions[color]) {
        if (row in this.nextPutPositions[color]) {
          for (let col of this.nextPutPositions[color][row]) {
            const element = document.getElementById(`row${row}-col${col}`);
            element.classList.add("placeable-disk");
          }
        }
      }
    },
    isExistDisc(row, col, color) {
      const arr = this.putPositions[color][row];
      if(arr.indexOf(col) !== -1) {
        return true;
      }
      return false;
    },
    isCanPutDisc(row, col, color) {
      const arr = this.nextPutPositions[color][row];
      if(arr.indexOf(col) !== -1) {
        return true;
      }
      return false;
    },
    exploreHorizontal(row, col, color, i) {
      let lastCol = null;
      while(col > 1 && col < 8){
        col+=i;
        if (!this.isExistDisc(row, col, color)) {
          const rc = this.getReverseColor(color);
          if (!this.isExistDisc(row, col, rc)) {
            lastCol = col;
          } else {
            this.hitDiscsPosi.push([row, col])
          }
          break;
        }
      }
      return lastCol;
    },
    exploreVertical(row, col, color, i) {
      let lastRow = null;
      while(row > 1 && row < 8){
        row+=i;
        if (!this.isExistDisc(row, col, color)) {
          const rc = this.getReverseColor(color);
          if (!this.isExistDisc(row, col, rc)) {
            lastRow = row;
          } else {
            this.hitDiscsPosi.push([row, col])
          }
          break;
        }
      }
      return lastRow;
    },
    exploreDiagonal(row, col, color, i) {
      let lastRow = null;
      let lastCol = null;
      while((row > 1 && row < 8) && (col > 1 && col < 8)){
        row+=i[0];
        col+=i[1];
        if (!this.isExistDisc(row, col, color)) {
          const rc = this.getReverseColor(color);
          if (!this.isExistDisc(row, col, rc)) {
            lastRow = row;
            lastCol = col;
          } else {
            this.hitDiscsPosi.push([row, col])
          }
          break;
        }
      }
      return {row: lastRow, col: lastCol};
    },
    getReverseColor (color) {
      return color === "white" ? "black" : "white";
    },
    async putDiscByUser(row, col) {
      if (!this.isCanPutDisc(row, col, this.userColor)) {
        return;
      }
      this.putPositions[this.userColor][row].push(col);
      this.doReverse(row, col, this.userColor);
      this.lastPut = {row: row, col: col}
      this.allSet();
      if (this.isFinishGame()) {
        return;
      }
      this.turn++;
      await this.putDiscByEnemy();
      while (!this.isFinishGame() && this.isSkipTurn(this.userColor)) {
        await this.putDiscByEnemy();
      }
      if (this.isFinishGame()) {
        return;
      }
      if (this.isAutoPut) {
        await this.waitOneSecond(this.interval);
        const npp = this.nextPutPositions[this.userColor];
        const nonEmptyKeys = Object.keys(npp).filter(key => npp[key].length > 0);
        const randomKey = Number(nonEmptyKeys[Math.floor(Math.random() * nonEmptyKeys.length)]);
        const arrayForKey = npp[randomKey];
        const randomValue = arrayForKey[0];
        await this.putDiscByUser(randomKey, randomValue)
      }
    },
    async putDiscByEnemy() {
      await this.waitOneSecond(this.interval);
      const rc = this.getReverseColor(this.userColor);
      if (this.isSkipTurn(rc)) {
        console.log("enemy's turn is skip");
        this.updateNextPosiUI(this.userColor);
        return;
      }
      const nextPosi = await this.getPutDiscPosition(this.endpoint1 ,this.userColor, this.putPositions, this.gameId, this.turn, this.lastPut);
      this.turn++;
      if (!this.nextPutPositions[rc][nextPosi["row"]].includes(nextPosi["col"])) {
        console.error("response next position is invalid");
        return;
      }
      this.putPositions[rc][nextPosi["row"]].push(nextPosi["col"]);
      this.doReverse(nextPosi["row"], nextPosi["col"], rc);
      this.allSet();
      this.updateNextPosiUI(this.userColor);
    },
    doReverse(row, col, color) {
      this.hitDiscsPosi = [];
      this.updateNextPosi(row, col, color);
      this.hitDiscsPosi.forEach(hitDisc => {
        if(row === hitDisc[0]){
          let cols = this.sortBySize(col, hitDisc[1]);
          while(cols["low"]!==cols["heigh"]-1){
            cols["low"]+=1;
            this.reverseTarget(row, cols["low"], color)
          }
        } else if (col === hitDisc[1]) {
          let rows = this.sortBySize(row, hitDisc[0]);
          while(rows["low"]!==rows["heigh"]-1){
            rows["low"]+=1;
            this.reverseTarget(rows["low"], col, color)
          }
        } else {
          let low = hitDisc[1] < col ? hitDisc : [row, col];
          let heigh = hitDisc[1] > col ? hitDisc : [row, col];
          // 右上か右下か判別
          const increment = low[0] > heigh[0] ? -1: 1;
          while(low[1]!==heigh[1]-1){
            low[0]+=increment;
            low[1]+=1;
            this.reverseTarget(low[0], low[1], color);
          }
        }
      })
    },
    reverseTarget(row, col, color){
      const rc = this.getReverseColor(color);
      let index = this.putPositions[rc][row].indexOf(col);
      this.putPositions[rc][row].splice(index, 1);
      this.putPositions[color][row].push(col);
    },
    sortBySize(a, b){
      let low   = a < b ? a: b;
      let heigh = a > b ? a: b;
      return {"low": low, "heigh": heigh};
    },
    isSkipTurn(color) {
      const tmpRows = this.nextPutPositions[color];
      // forEach内でreturnできないため
      let bool = true;
      Object.keys(tmpRows).forEach((row) => {
        if (tmpRows[row].length > 0) {
            bool = false;
            // return false;
          }
      });
      if (bool) console.log(color + " payer is skip");
      return bool;
    },
    isFinishGame() {
      if(this.turn == 0) return;
      if (this.turn && this.userDiscCount + this.enemyDiscCount === 64 || 
        (this.isSkipTurn("white")&&this.isSkipTurn("black"))
        ) {
          console.log("game finished!");
          this.sendGameResult(this.userColor, this.putPositions, this.gameId, this.turn, this.lastPut);
          return true;
      }
      return false;
    },
    waitOneSecond(seconds) {
      return new Promise(resolve => setTimeout(resolve, seconds));
    }
  },
  async mounted() {
    this.allSet();
    this.updateNextPosiUI(this.userColor);
    const gameId = await this.createGameID();
    console.log(gameId);
    this.saveGameID(gameId);
  },
  computed: {
    getResultText: function () {
      if (!this.isFinishGame()) {
        return null;
      }
      if (this.userDiscCount == this.enemyDiscCount) {
        return "引き分けです。";
      }
      const winnerColor = this.userDiscCount > this.enemyDiscCount ? this.userColor : this.getReverseColor(this.userColor);
      return `${winnerColor}プレイヤーの勝利です!`
    },
    ...mapGetters([
      'endpoint1',
      'endpoint2',
      'interval',
      'gameId'
    ])
  }
}
</script>

<style scoped>
.othello-board {
  color: aliceblue;
  margin: auto;
  width: 500px;
  height: 500px;
  background-color: rgb(25, 102, 76);
  border: 8px solid black;
}
.row {
  height: 12.5%;
  display: flex;
}
.col {
  width: 12.5%;
  text-align: center;
  border: 1px solid black;
  /* 子要素でmargin-autoが効くようにするため */
  display: flex;
}
.white,.black {
  margin: auto;
  height: 90%;
  width: 90%;
  border-radius: 50%;
}
.white {
  background-color: white;
}
.black {
  background-color: black;
}
.placeable-disk {
  margin: auto;
  height: 50%;
  width: 50%;
  border-radius: 50%;
  background-color: rgba(160, 160, 160, 0.7);
}

.result-board {
  position: absolute;
  background-color: rgba(255, 255, 255, 0.9);
  color: black;
  padding: 5%;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  -webkit-transform: translate(-50%, -50%);
  -ms-transform: translate(-50%, -50%);
}
</style>
