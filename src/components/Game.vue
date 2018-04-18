<template>
  <section class="hero is-fullheight">
    <div class="hero-body">
      <div class="container">
        <div class="columns is-centered">
          <div class="column is-10">
            <div class="columns game-row" v-for="row in rows" @click.right.prevent="() => {}">
              <div class="column game-col" :key="cell.row + '-' + cell.column" v-for="cell in row">
                <div class="ms-box"
                     @dblclick="revealAdjacent(cell)"
                     @click.left="revealCell(cell)"
                     @click.right.prevent="flagCell(cell)"
                     :class="{
                       'is-blank': cell.mineCount === 0,
                       'is-flag' : cell.isFlagged,
                       'is-mine': cell.isMine,
                       'is-revealed': cell.isRevealed
                     }"
                >
                  <div class="cell-content" v-if="cell.isRevealed">
                    <span class="cell-number" v-if="cell.mineCount > 0">{{ cell.mineCount }}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
  import { EventBus } from "../EventBus";

  export default {
    name: "game",
    data() {
      return {
        mines: [],
        rows: [],
        rowsNumber: 16,
        columnsNumber: 30,
        minesNumber: 99
      }
    },
    mounted() {
      //  Create basic board
      this.createBoard();
      //  Set random mines
      this.setMinesRandomly();
      //  Calculate mines
      this.calculateCellMines();
    },
    methods: {
      createBoard() {

        //  Create rows
        for ( let i = 0; i < this.rowsNumber; i++ ) {

          let row = [];

          for ( let j = 0; j < this.columnsNumber; j++ ) {

            const cell = {
              mineCount: 0,
              row: i,
              column: j,
              isMine: false,
              isFlagged: false,
              isRevealed: false
              // isMine: mines.filter( mine => mine.row === i && mine.col === j).length > 0
            };

            row.push(cell);
          }

          this.rows.push(row);
        }
      },
      setMinesRandomly() {
        let added = 0;
        while (added < this.minesNumber) {
          const randomRow = Math.floor((Math.random() * this.rowsNumber));
          const randomCol = Math.floor((Math.random() * this.columnsNumber));

          if ( this.mines.filter( mine => mine.row === randomRow && mine.column === randomCol  ).length === 0 ) {
            this.rows[randomRow][randomCol].isMine = true;
            this.mines.push(this.rows[randomRow][randomCol]);
            added++;
          }
        }
      },
      getAdjacentCells( cell ) {
        const cells = [];
        for (let dRow = -1; dRow <= 1; ++dRow) {
          for (let dCol = -1; dCol <= 1; ++dCol) {
            if (dRow !== 0 || dCol !== 0) {
              if ( this.rows[ cell.row + dRow ] !== undefined && this.rows[ cell.row + dRow ][ cell.column + dCol ] !== undefined ) {
                cells.push(this.rows[ cell.row + dRow ][ cell.column + dCol ]);
              }
            }
          }
        }
        return cells;
      },
      calculateCellMines() {

        this.mines.forEach(mine => {
          const adjacentCells = this.getAdjacentCells(mine);
          adjacentCells.forEach( cell => {
            if ( !cell.isMine ) {
              cell.mineCount++;
            }
          });
        });
      },
      //  Game actions
      revealCell(cell) {
        if ( !cell.isRevealed ) {
          cell.isRevealed = true;
          if ( cell.isMine && !cell.isFlagged ) {
            console.log('gg');
          } else {
            this.revealAdjacent(cell);
          }
        }
      },
      revealAdjacent(cell) {
        if ( cell.isRevealed ) {
          const adjacent = this.getAdjacentCells(cell);
          adjacent.forEach( adj => {
            if ( !adj.isMine && !adj.isRevealed ) {

              adj.isRevealed = true;

              if ( adj.mineCount === 0 ) {
                this.revealAdjacent(adj);
              }
            }
          });
        }
      },
      handleDoubleClick() {
        //  todo
        //  if adjacentFlags === adjacentMines
        //    if adjacentFlags are indeed mines, revealAdjacent
        //    else GG
      },
      flagCell(cell) {
        if ( !cell.isRevealed ) {
          cell.isFlagged = !cell.isFlagged;
        }
      }
    }
  }
</script>

<style lang="scss" scoped>
  .game-col {
    padding: 0.2rem;
  }

  .ms-box {
    cursor: pointer;
    border-radius: 3px;
    position: relative;
    background-color:hsl(0, 0%, 75%);

    -webkit-user-select: none; /* Safari 3.1+ */
    -moz-user-select: none; /* Firefox 2+ */
    -ms-user-select: none; /* IE 10+ */
    user-select: none; /* Standard syntax */

    &.is-revealed {
      background-color:hsl(0, 0%, 90%);
    }

    &.is-revealed.is-blank {
      opacity: .6;
    }

    &.is-mine.is-revealed {
      background-color: hsl(348, 100%, 61%);
    }

    &.is-flag {
      background-color: hsl(204, 86%, 53%);
    }

    &:after {
      content: '';
      display: block;
      padding-bottom: 100%;
    }

    .cell-content {
      position: absolute;
      left: 0;
      right: 0;
      top: 0;
      bottom: 0;
      width: 100%;
      height: 100%;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 26px;
    }
  }
</style>
