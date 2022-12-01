<template>
  <h1>Xdd</h1>
  <div>
    <h3>Size</h3>
    <input type="number" v-model="size.w">
    x
    <input type="number" v-model="size.h">
  </div>
  <div>
    <h3>Bombs</h3>
    <input type="number" v-model="nBombs">  
  </div>
  <button @click="init">Generate</button>
  <h2 v-if="state === WON">Yay!</h2>
  <h2 v-if="state === LOST">Sad chet</h2>
  <div id="game" :key="gamekey" :class="`${state === LOST ? 'game--lost':''}`" v-if="initialized">
    <div v-for="row in size.h" :key="`row_${row}`">
      <div v-for="col in size.w" :key="`col_${col}`"
        class="cell"
        :class="bombs[row-1][col-1] ? 'cell--bomb' : ''"
        @click="onClick(col-1, row-1)"
        @contextmenu.prevent="onRightClick(col-1, row-1)" 
        :data-cell="`${col-1}_${row-1}`">
      </div>
    </div>
  </div>
</template>

<script>
const _generateMatrix = (w, h) => {
  let result = []
  Array(h).fill(0).forEach(row => {
    const newRow = [...Array(w).keys()].map(k => false)
    result.push(newRow)
  })
  return result
}
const [IDLE, PLAYING, WON, LOST] = [...Array(4).keys()]

export default {
  name: 'Minesweeper',
  data () {
    return {
      // const
      IDLE, PLAYING, WON, LOST,
      // settings
      size: {
        h: 20,
        w: 40
      },
      nBombs: Math.floor(20*40*0.2),

      // game
      bombs: null,
      checked: null,
      checkedsum: 0,
      initialized: false,
      gamekey: Date.now(),
      state: IDLE,
    }
  },
  methods: {
    init() {
      this.initialized = false
      const MAX_BOMBS = Math.floor(this.size.h*this.size.w*0.8)
      const n = Math.min(MAX_BOMBS, this.nBombs)
      if (this.nBombs != n) console.log("Bombs capped to", n)
      this.nBombs = n
      this.bombs = _generateMatrix(this.size.w, this.size.h)
      this.checked = _generateMatrix(this.size.w, this.size.h)
      this.checkedsum = 0
      this.gamekey = Date.now()
      this.state = PLAYING      
      this.initialized = true
    },
    placeBombs(x, y) {
      console.log("Placing", this.nBombs, "bombs")
      let placed = 0
      const MAXITERS = this.w * this.h
      let iters = 0
      while (placed < this.nBombs){
        if (iters++ > MAXITERS) {
          console.error("Max iterations reached. Seems like an infinite loop!")
          break
        }
        const newx = Math.floor(Math.random()*this.size.w)
        const newy = Math.floor(Math.random()*this.size.h)
        if (newx === x && newy === y) continue
        this.bombs[newy][newx] = true
        placed++
      }
      console.log("Placed all bombs")
    },
    onClick(x, y) {
      console.log("Clicked on", x, y)
      if (this.checkedsum === 0) {
        this.placeBombs(x, y)
      }
      if (this.bombs[y][x]){
        this.state = LOST
        return
      }
      this.getElement(x, y).classList.remove('cell--flag')
      this.checkBombs(x, y)
      // if no more cells to check, you won
      this.checkWin()
    },
    getElement(x, y) {
      return document.querySelector(`[data-cell="${x}_${y}"]`)
    },
    onRightClick(x, y) {
      console.log("Right click")
      if (!this.checked[y][x])
        this.getElement(x, y).classList.add(`cell--flag`)
    },
    checkWin() {
      // Sum all checked
      if (this.checkedsum === this.size.w*this.size.h - this.nBombs)
        this.state = WON
    },
    applyAround (px, py, method) {
      for (var j = Math.max(0, py - 1); j <= Math.min(this.size.h - 1, py + 1); j++) {
        for (var i = Math.max(0, px - 1); i <= Math.min(this.size.w - 1, px + 1); i++) {
          if (i === px && j === py) continue
          method(i, j)
        }
      }
    },
    checkBombs(x, y) {
      // if already checked, just return
      if (this.checked[y][x]) return
      this.checked[y][x] = true
      this.checkedsum ++
      // count bombs around
      let sum = 0
      this.applyAround(x, y, (i, j) => sum += this.bombs[j][i] ? 1 : 0)
      this.getElement(x, y).classList.add(`cell--${sum}`)
      if (sum === 0){
        // Recursively check around
        this.applyAround(x, y, (i, j) => this.checkBombs(i, j))
      }
    },
    print() {
      let s = ''
      this.bombs.forEach((row) => {
        console.log("row", row)
        s = `${s}|${row.map(c => c ? 'x' : ' ').join('|')}|\n`
      })
      console.log(s)
    }
  }
}
</script>

<style lang="scss" scoped>
#game {
  display: grid;
  grid-template-rows: 1fr;
  > div {
    display: flex;
    grid-template-columns: 1fr;
  }
}
$size: 25px;
$cell-flag: "https://upload.wikimedia.org/wikipedia/commons/thumb/8/83/Minesweeper_flag.svg/76px-Minesweeper_flag.svg.png";
$cell-mine: "https://apprecs.org/ios/images/app-icons/256/e7/451931111.jpg";
$cell-mine-wrong: "http://pixelartmaker-data-78746291193.nyc3.digitaloceanspaces.com/image/cdd35af4eaef3da.png";
$cell-tile: "https://encrypted-tbn0.gstatic.com/images?q=tbn:ANd9GcRZQsW6Z5gruT_2-HiSrx5lkFE0LwhGwY5kP0Mor3LX2C22gGd5JEIWbzC0VFk92latMJ0&usqp=CAU";
$cell-1: "https://upload.wikimedia.org/wikipedia/commons/thumb/c/ca/Minesweeper_1.svg/76px-Minesweeper_1.svg.png";
$cell-2: "https://upload.wikimedia.org/wikipedia/commons/thumb/4/44/Minesweeper_2.svg/76px-Minesweeper_2.svg.png";
$cell-3: "https://upload.wikimedia.org/wikipedia/commons/thumb/0/08/Minesweeper_3.svg/76px-Minesweeper_3.svg.png";
$cell-4: "https://upload.wikimedia.org/wikipedia/commons/thumb/4/4f/Minesweeper_4.svg/76px-Minesweeper_4.svg.png";
$cell-5: "https://upload.wikimedia.org/wikipedia/commons/thumb/4/46/Minesweeper_5.svg/76px-Minesweeper_5.svg.png";
$cell-6: "https://upload.wikimedia.org/wikipedia/commons/thumb/c/cc/Minesweeper_6.svg/76px-Minesweeper_6.svg.png";
$cell-7: "https://upload.wikimedia.org/wikipedia/commons/thumb/5/56/Minesweeper_7.svg/76px-Minesweeper_7.svg.png";
$cell-8: "https://upload.wikimedia.org/wikipedia/commons/thumb/0/0d/Minesweeper_8.svg/76px-Minesweeper_8.svg.png";
$cell-0: "https://upload.wikimedia.org/wikipedia/commons/thumb/8/80/Minesweeper_0.svg/76px-Minesweeper_0.svg.png";

.cell {
  width: $size;
  height: $size;
  transition: opacity 0.5 ease;

  background-size: 100% 100%;
  background-image: url($cell-tile);
  &--flag { background-image: url($cell-flag); }
  &--1 { background-image: url($cell-1); }
  &--2 { background-image: url($cell-2); }
  &--3 { background-image: url($cell-3); }
  &--4 { background-image: url($cell-4); }
  &--5 { background-image: url($cell-5); }
  &--6 { background-image: url($cell-6); }
  &--7 { background-image: url($cell-7); }
  &--8 { background-image: url($cell-8); }
  &--0 { background-image: url($cell-0); }

  &--bomb {
    opacity: 1;
  }

  opacity: 1;
}
.game--lost {
  .cell {
    pointer-events: none;
    &--bomb {
      background-image: url($cell-mine);
    }
    &--flag:not(&--bomb) {
      background-image: url($cell-mine-wrong);
    }
  }
}

</style>