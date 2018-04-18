<template>
    <div class="ms-box" @click.left="reveal" @click.right.prevent="setFlag" :class="{ 'is-flag' : isFlagged, 'is-exploded' : isExploded, 'is-revealed': isRevealed }">
      <div class="cell-content" v-if="isRevealed">
        <span class="cell-number" v-if="cell.value > 0">{{ cell.value }}</span>
      </div>
    </div>
</template>

<script>
  import {EventBus} from "../EventBus";

  const STATUS_HIDDEN = 0;
  const STATUS_REVEALED = 1;
  const STATUS_FLAGGED = 2;
  const STATUS_FLAGGED_DOUBT = 3;
  const STATUS_EXPLODED = 3;

  export default {
    name: "game-cell",
    props: {
      cell: {
        type: Object,
        required: true
      }
    },
    data() {
      return {
        status: STATUS_HIDDEN
      }
    },
    mounted() {
      EventBus.$on('reveal', cell => {
        if ( this.cell.row === cell.row && this.cell.column === cell.column ) {
          this.reveal(false);
        }
      });
    },
    methods: {
      setFlag: function () {
        // if ( this.status === STATUS_HIDDEN )
        if ( this.status !== STATUS_FLAGGED )
          this.status = STATUS_FLAGGED;
        else
          this.status = STATUS_HIDDEN;
      },
      reveal: function ( includeAdjacent = true ) {
        this.status = STATUS_REVEALED;
        if ( this.cell.isMine ) {
          //  todo - Handle loss
          console.log('U LOSE');
          this.status = STATUS_EXPLODED;
        } else if ( includeAdjacent ) {
          EventBus.$emit('reveal-adjacent', this.cell);
        }
      }
    },
    computed: {
      isRevealed: function () {
        return this.status === STATUS_REVEALED;
      },
      isFlagged: function () {
        return this.status === STATUS_FLAGGED;
      },
      isExploded: function () {
        return this.status === STATUS_EXPLODED;
      }
    }
  }
</script>

<style lang="scss">
  .ms-box {
    cursor: pointer;
    border-radius: 3px;
    position: relative;
    background-color:hsl(0, 0%, 86%);

    &.is-revealed {
      opacity: .6;
    }

    &.is-exploded {
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
