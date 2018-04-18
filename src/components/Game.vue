<template>
  <section class="hero is-fullheight">
    <div class="hero-body">
      <div class="container">
        <div class="columns is-centered">
          <div class="column is-8">
            <div class="columns game-row" v-for="row in rows">
              <div class="column game-col" :key="cell.row + '-' + cell.column" v-for="cell in row">
                <game-cell :cell="cell"></game-cell>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
  import GameCell from './GameCell';
  import { EventBus } from "../EventBus";

  export default {
    name: "game",
    components: { GameCell },
    data() {
      return {
        mines: [],
        rows: [],
        rowsNumber: 8,
        columnsNumber: 8,
        minesNumber: 10
      }
    },
    mounted() {
      EventBus.$on('reveal-adjacent', this.revealAdjacent);
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
              value: 0,
              row: i,
              column: j,
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
              cell.value++;
            }
          });
        });
      },
      revealAdjacent(mine) {
        const adjacent = this.getAdjacentCells(mine);

        adjacent.forEach( cell => {
          if ( !cell.isMine ) {

            EventBus.$emit('reveal', cell);

            // if ( cell.value === 0 ) {
            //   // this.revealAdjacent(cell);
            // } else {
            //
            // }
          }
        });
      }
    }
  }
</script>

<style lang="scss" scoped>
  .game-col {
    padding: 0.2rem;
  }
</style>
