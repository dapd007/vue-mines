<template>
  <div class="box">
    <div class="columns">
      <div class="column">
        <h3 class="title">Choose difficulty</h3>
      </div>
    </div>
    <div class="columns">
      <div class="column" v-for="difficulty in choices">
        <div class="box difficulty-box" :class="{ 'is-selected': difficulty === selectedDifficulty }" @click="selectDifficulty(difficulty)">
          <span class="size"><b>{{ difficulty.columnsNumber }} x {{ difficulty.rowsNumber }}</b></span>
          <span class="mines">{{ difficulty.minesNumber }} mines</span>
        </div>
      </div>
    </div>
    <div class="columns">
      <div class="column" style="text-align: right;">
        <button class="button is-primary" @click="$emit('picked')">Play</button>
      </div>
    </div>
  </div>
</template>

<script>
  export default {
    name: "difficulty-picker",
    props: {
      choices: {
        type: Array,
        required: true
      },
      value: {
        type: Object
      }
    },
    data() {
      return {
        selectedDifficulty: {}
      }
    },
    mounted() {
      this.selectDifficulty( this.choices[0] );
    },
    methods: {
      selectDifficulty(difficulty) {
        this.selectedDifficulty = difficulty;
        this.$emit('input', this.selectedDifficulty);
      }
    }
  }
</script>

<style lang="scss" scoped>
  .difficulty-box {
    cursor: pointer;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    font-size: 18px;

    &.is-selected {
      background-color: #7957d5;
      color: #fff;
    }

    .size {
      margin-bottom: 10px;
    }
  }
</style>
