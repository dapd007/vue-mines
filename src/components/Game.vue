<template>
  <section class="game-page">
    <main class="game-container">
        <div class="columns is-centered">
          <div class="column is-10">
            <div class="columns game-row" v-for="row in rows" @click.right.prevent="() => {}">
              <div class="column game-col" :key="cell.row + '-' + cell.column" v-for="cell in row">
                <div class="ms-box"
                     @dblclick="handleDoubleClick(cell)"
                     @click.left="revealCell(cell)"
                     @click.right.prevent="flagCell(cell)"
                     :class="{
                       'is-blank': cell.mineCount === 0 && !cell.isMine,
                       'is-flag' : cell.isFlagged,
                       'is-mine': cell.isMine,
                       'is-revealed': cell.isRevealed,
                   }"
                     :style="getCellStyles(cell)"
                >
                  <div class="cell-content" v-if="cell.isRevealed">
                    <span class="cell-number" v-if="cell.mineCount > 0">{{ cell.mineCount }}</span>
                    <i class="fas fa-bomb" v-else-if="cell.isMine"></i>
                  </div>
                  <div class="cell-content" v-else-if="cell.isFlagged">
                    <i class="fas fa-flag"></i>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
    </main>
    <aside class="game-sidebar">
      <p>Total mines: {{minesNumber}}</p>
      <p>Flags: {{flagsCount}}</p>
      <div class="game-sidebar--actions">
        <button class="button is-primary" @click="startGame">Start over</button>
      </div>
    </aside>
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
        rowsNumber: 10,
        columnsNumber: 16,
        minesNumber: 45,
        flagsCount: 0,
        gameOver: false,
      }
    },
    mounted() {
      this.startGame();
    },
    methods: {
      startGame() {
        this.gameOver = false;
        //  Create basic board
        this.createBoard();
        //  Set random mines
        this.setMinesRandomly();
        //  Calculate mines
        this.calculateCellMines();
      },
      createBoard() {
        //  Reset board
        this.mines = [];
        this.rows = [];
        this.flagsCount = 0;

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
      getCellStyles(cell) {
        const colors = [
          'rgba(84,175,96,.35)',
          'rgba(103,162,96,.35)',
          'rgba(255,166,90,.35)',
          'rgba(255,148,91,.35)',
          'rgba(160,122,96,.35)',
          'rgba(179,109,96,.35)',
          'rgba(198,96,96,.35)',
          'rgba(217,82,96,.35)',
          'rgba(236,69,96,.35)',
        ];

        if ( cell.mineCount === 0 || !cell.isRevealed )
          return {};
        return {
          'backgroundColor': colors[cell.mineCount - 1]
        };
      },
      //  Game actions
      revealCell(cell) {
        if ( !this.gameOver && !cell.isRevealed && !cell.isFlagged ) {
          cell.isRevealed = true;
          if ( cell.isMine && !cell.isFlagged ) {
            this.gameLost();
          } else {
            this.revealAdjacent(cell);
          }
        }
      },
      revealAdjacent(cell) {
        if ( !this.gameOver && cell.isRevealed ) {
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
      handleDoubleClick(cell) {
        if ( !this.gameOver ) {
          const adjacent = this.getAdjacentCells(cell);
          const flags = adjacent.filter( cell => cell.isFlagged );

          if ( flags.length === cell.mineCount ) {
            const result = flags.filter( flag => flag.isMine );

            if ( result.length === flags.length ) {
              this.revealAdjacent(cell);
            } else {
              this.gameLost();
            }
          }
        }
      },
      flagCell(cell) {
        if ( !cell.isRevealed ) {
          cell.isFlagged = !cell.isFlagged;

          if ( cell.isFlagged )
            this.flagsCount++;
          else
            this.flagsCount--;
        }
      },
      gameLost() {
        this.gameOver = true;
        this.revealMines();
      },
      revealMines() {
        this.mines.forEach( mine => {
          mine.isRevealed = true;
        });
      }
    }
  }
</script>

<style lang="scss" scoped>
  .game-page {
    height: 100vh;
    display: flex;
  }

  main.game-container {
    width: 80%;
    display: flex;
    align-items: center;
    justify-content: center;

    & > .columns {
      width: 100%;
    }
  }

  aside.game-sidebar {
    position: fixed;
    width: 20%;
    height: 100vh;
    top: 0;
    right: 0;
    background-color: #fdfdfd;
    border-left: 2px solid #f0f0f0;
    padding: 20px;
    display: flex;
    flex-direction: column;

    .game-sidebar--actions {
      margin-top: auto;

      .button {
        width: 100%;
      }
    }
  }

  .game-col {
    padding: 0.2rem;
  }

  .ms-box {
    color: #000;
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

      &.is-flag {
        background-color: hsl(141, 71%, 48%);
        color: #000;
      }
    }

    &.is-flag {
      background-color:hsl(0, 0%, 85%);
      color: hsl(204, 86%, 53%);
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
