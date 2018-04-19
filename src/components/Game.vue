<template>
  <section class="game-page" :class="selectedTheme.class">
    <b-modal class="difficulty-modal" :active.sync="isDifficultyPickerActive" :can-cancel="false">
      <difficulty-picker v-model="game.settings" :choices="difficulties" v-on:picked="startGame"></difficulty-picker>
    </b-modal>
    <main class="game-container">
        <div class="columns is-centered">
          <div class="column is-10">
            <div class="columns game-row" v-for="row in game.rows" @click.right.prevent="() => {}">
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
      <template v-if="game.isSetUp">
        <p>Total mines: {{game.settings.minesNumber}}</p>
        <p>Flags: {{game.flagsCount}}</p>
        <div class="game-sidebar--actions">
          <button class="button is-info" @click="startGame">Start over</button>
          <button class="button" @click="changeDifficulty">Change difficulty</button>
          <b-field
            label="Theme">
            <b-select v-model="selectedTheme" placeholder="Pick a color theme" expanded>
              <option :value="theme" v-for="theme in themes">{{ theme.label }}</option>
            </b-select>
        </b-field>
        </div>
      </template>
    </aside>
  </section>
</template>

<script>
  import { EventBus } from "../EventBus";
  import DifficultyPicker from './DifficultyPicker';

  export default {
    name: "game",
    components: { DifficultyPicker },
    data() {
      return {
        isDifficultyPickerActive: false,
        themes: [
          {
            label: 'Dark',
            class: 'dark-theme'
          },
          {
            label: 'Light',
            class: ''
          }
        ],
        difficulties: [
          {
            rowsNumber: 7,
            columnsNumber: 10,
            minesNumber: 15
          },
          {
            rowsNumber: 12,
            columnsNumber: 20,
            minesNumber: 40
          },
          {
            rowsNumber: 16,
            columnsNumber: 30,
            minesNumber: 99
          }
        ],
        game: {
          settings: {},
          mines: [],
          rows: [],
          flagsCount: 0,
          isCreated: false,
          isOver: false,
        },
        selectedTheme: null
      }
    },
    mounted() {
      this.selectedTheme = this.themes[0];
      this.isDifficultyPickerActive = true;
    },
    methods: {
      changeDifficulty() {
        this.game.isSetUp = false;
        this.isDifficultyPickerActive = true;
      },
      startGame() {
        this.game.isOver = false;
        //  Create basic board
        this.createBoard();
        //  Set random mines
        this.setMinesRandomly();
        //  Calculate mines
        this.calculateCellMines();

        this.game.isSetUp = true;
        this.isDifficultyPickerActive = false;
      },
      createBoard() {
        //  Reset board
        this.game.mines = [];
        this.game.rows = [];
        this.game.flagsCount = 0;

        //  Create rows
        for ( let i = 0; i < this.game.settings.rowsNumber; i++ ) {

          let row = [];

          for ( let j = 0; j < this.game.settings.columnsNumber; j++ ) {

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

          this.game.rows.push(row);
        }
      },
      setMinesRandomly() {
        let added = 0;
        while (added < this.game.settings.minesNumber) {
          const randomRow = Math.floor((Math.random() * this.game.settings.rowsNumber));
          const randomCol = Math.floor((Math.random() * this.game.settings.columnsNumber));

          if ( this.game.mines.filter( mine => mine.row === randomRow && mine.column === randomCol  ).length === 0 ) {
            this.game.rows[randomRow][randomCol].isMine = true;
            this.game.mines.push(this.game.rows[randomRow][randomCol]);
            added++;
          }
        }
      },
      getAdjacentCells( cell ) {
        const cells = [];
        for (let dRow = -1; dRow <= 1; ++dRow) {
          for (let dCol = -1; dCol <= 1; ++dCol) {
            if (dRow !== 0 || dCol !== 0) {
              if ( this.game.rows[ cell.row + dRow ] !== undefined && this.game.rows[ cell.row + dRow ][ cell.column + dCol ] !== undefined ) {
                cells.push(this.game.rows[ cell.row + dRow ][ cell.column + dCol ]);
              }
            }
          }
        }
        return cells;
      },
      calculateCellMines() {
        this.game.mines.forEach(mine => {
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
      checkWin() {
        if ( this.game.flagsCount === this.game.mines.length ) {
          const flaggedMines = this.game.mines.filter( mine => mine.isFlagged = true );

          if ( flaggedMines.length === this.game.flagsCount ) {
            this.gameWon();
          }
        }
      },
      revealCell(cell) {
        if ( !this.game.isOver && !cell.isRevealed && !cell.isFlagged ) {
          cell.isRevealed = true;
          if ( cell.isMine && !cell.isFlagged ) {
            this.gameLost();
          } else {
            this.revealAdjacent(cell);
            this.checkWin();
          }
        }
      },
      revealAdjacent(cell) {
        if ( !this.game.isOver && cell.isRevealed ) {
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
        if ( !this.game.isOver ) {
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
            this.game.flagsCount++;
          else
            this.game.flagsCount--;

          this.checkWin();
        }
      },
      gameWon() {
        this.game.isOver = true;
        this.revealAll();

        this.$snackbar.open({
          message: 'You won, yay!',
          type: 'is-success',
          position: 'is-top',
          actionText: 'New game',
          duration: 5000,
          onAction: () => {
            this.startGame();
          }
        });
      },
      gameLost() {
        this.game.isOver = true;
        this.revealAll();

        this.$snackbar.open({
          message: ':( you just blew yourself up!',
          type: 'is-danger',
          position: 'is-top',
          actionText: 'try again',
          duration: 5000,
          onAction: () => {
            this.startGame();
          }
        });
      },
      revealAll() {
        this.game.rows.forEach( row => {
          row.forEach( cell => {
            cell.isRevealed = true;
          });
        });
      }
    }
  }
</script>

<style lang="scss">
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

        &:not(:last-child) {
          margin-bottom: 10px;
        }
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

  .difficulty-modal .modal-content {
    width: 960px;
    padding: .75rem;
  }

  .game-page.dark-theme {
    background-color: #16111c;

    .ms-box {
      background-color: #292034;
      color: #d0d0d0;

      &.is-mine.is-revealed {
        background-color: #ff3860;

        &.is-flag {
          background-color: hsl(141, 71%, 48%);
        }
      }

      &.is-revealed.is-blank {
        opacity: .3;
      }
    }

    aside.game-sidebar {
      background-color: #292034;
      border-color: #292034;
      color: #d0d0d0;

      label {
        color: #d0d0d0;
      }

      select, button:not(.is-info) {
        background: #16111c;
        color: #d0d0d0;
        border-color: #16111c;
      }
    }

    .box {
      background-color: #292034;
      color: #d0d0d0;

      .box:not(.is-selected) {
        background-color: #16111c;
      }
    }

    .title {
      color: #d0d0d0;
    }
  }
</style>
