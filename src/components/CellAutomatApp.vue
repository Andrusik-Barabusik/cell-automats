<script>
import animalPalette from './AnimalPalette.vue';
import patternSelector from './PatternSelector';
import Slider from '@vueform/slider';
import '@vueform/slider/themes/default.css';

export default {
  name: 'CellAutomatApp',
  components: { animalPalette, Slider, patternSelector },
  props: {
    msg: String,
  },
  data() {
    return {
      numTransforms: 0,
      fieldSize: 100,
      cellSize: 5,
      currentGeneration: [],
      nextGeneration: [],
      runmodus: 'flow',
      startButtonMsg: 'resume',
      generationCounter: 0,
      numberColors: 6,
      setNumberColors: 6,
      minColors: 3,
      maxColors: 9,
      similarity: 0.5,
      automatName: 'nonTransitiveAutomat',
      setAutomatName: 'nonTransitiveAutomat',
      selectedPatternIndex: -1,
      lifeNeighbours: {
        value: [2, 3],
      },
      generationTimeout: 0,
      //      values: [0]
    };
  },
  computed: {},
  watch: {
    setAutomatName: function (newName, oldName) {
      this.reset();
    },
  },

  mounted() {
    this.currentGeneration = Array.from(
      new Array(this.fieldSize),
      () => new Array(this.fieldSize)
    );
    this.nextGeneration = Array.from(
      new Array(this.fieldSize),
      () => new Array(this.fieldSize)
    );
    this.resetField();
  },
  methods: {
    //    getMessage: function(mes){
    //      return tmessages.tmsg(mes)
    getMsg(name) {
      return this.$store.state.messages[name];
    },
    getRunmodus() {},
    changeGoButton() {
      if (this.runmodus === 'flow') {
        if (this.$store.state.running === 'paused') {
          this.$store.commit('changeRunningState', 'running');
          this.startButtonMsg = 'pause';
          this.nextGenerationRun();
        } else {
          this.$store.commit('changeRunningState', 'paused');
          this.startButtonMsg = 'resume';
        }
      } else {
        this.nextGenerationRun();
      }
    },
    changeRunModus(modus) {
      if (modus === 'flow') {
        this.startButtonMsg = 'resume';
        this.$store.commit('changeRunningState', 'paused');
      } else {
        this.startButtonMsg = 'nextGeneration';
      }
    },
    nextGenerationRun() {
      this.generationCounter++;
      this.createNewGeneration();
      if (this.runmodus === 'flow' && this.$store.state.running === 'running') {
        this.generationTimeout = setTimeout(this.nextGenerationRun, 1);
      }
    },
    reset() {
      clearTimeout(this.generationTimeout);
      this.generationCounter = 0;
      this.$store.commit('reset');
      if (this.runmodus === 'flow') {
        this.$store.commit('changeRunningState', 'paused');
        this.startButtonMsg = 'start';
      }
      this.resetField();
    },
    clearLifeField() {
      this.reset();
      this.resetGenerationCtx();
      for (var i = 0; i < this.fieldSize; i++) {
        for (var j = 0; j < this.fieldSize; j++) {
          this.fillCell(9, i, j);
          this.currentGeneration[i][j] = 0;
        }
      }
      //        this.convertPattern(this.$store.state.lifePatterns[1])
      //        alert(JSON.stringify(this.values))
    },
    resetGenerationCtx() {
      this.currentGeneration = Array.from(
        new Array(this.fieldSize),
        () => new Array(this.fieldSize)
      );
      this.ctx = this.$refs.field.getContext('2d');
      this.ctx.clearRect(
        0,
        0,
        this.fieldSize * this.cellSize,
        this.fieldSize * this.cellSize
      );
    },
    resetField() {
      this.numberColors = this.setNumberColors;
      this.automatName = this.setAutomatName;
      this.resetGenerationCtx();
      this.currentGeneration = Array.from(
        new Array(this.fieldSize),
        () => new Array(this.fieldSize)
      );
      this.ctx = this.$refs.field.getContext('2d');
      this.ctx.clearRect(
        0,
        0,
        this.fieldSize * this.cellSize,
        this.fieldSize * this.cellSize
      );
      if (this.automatName === 'nonTransitiveAutomat') {
        this.resetNonTransitionalField();
      } else {
        this.resetGameOfLife();
      }
    },
    resetNonTransitionalField() {
      var previousColor = 0;
      for (var i = 0; i < this.fieldSize; i++) {
        for (var j = 0; j < this.fieldSize; j++) {
          if (typeof this.currentGeneration[i][j] !== 'undefined') {
            continue;
          }
          if (i > 45 && j > 30) {
            var b = 1;
          }
          var colorToFill = Math.floor(Math.random() * this.numberColors);
          previousColor = colorToFill;
          this.fillCell(colorToFill, i, j);
          this.currentGeneration[i][j] = colorToFill;
          if (Math.random() < this.similarity) {
            var stepNr = 1;
            whileLoop: while (true) {
              var i1 = i + stepNr;
              for (var j1 = j; j1 < j + stepNr; j1++) {
                if (
                  i1 >= this.fieldSize ||
                  j1 >= this.fieldSize ||
                  typeof this.currentGeneration[i1][j1] !== 'undefined'
                ) {
                  break whileLoop;
                }
                this.fillCell(colorToFill, i1, j1);
                this.currentGeneration[i1][j1] = colorToFill;
              }
              j1 = j + stepNr;
              for (i1 = i; i1 <= i + stepNr; i1++) {
                if (
                  i1 >= this.fieldSize ||
                  i1 >= this.fieldSize ||
                  typeof this.currentGeneration[i1][j1] !== 'undefined'
                ) {
                  break whileLoop;
                }
                this.fillCell(colorToFill, i1, j1);
                this.currentGeneration[i1][j1] = colorToFill;
              }
              stepNr++;
              if (Math.random() > this.similarity) {
                break whileLoop;
              }
            }
          }
        }
      }
    },
    resetGameOfLife() {
      var probability =
        ((this.lifeNeighbours.value[0] + this.lifeNeighbours.value[1]) / 2) *
        0.125;
      probability = 0.5;

      for (var i = 0; i < this.fieldSize; i++) {
        for (var j = 0; j < this.fieldSize; j++) {
          if (Math.random() < probability) {
            this.currentGeneration[i][j] = 1;
            this.fillCell(0, i, j);
          } else {
            this.currentGeneration[i][j] = 0;
            this.fillCell(9, i, j);
          }
        }
      }
    },
    fillCell(color, x, y) {
      this.ctx.fillStyle = 'black';
      this.ctx.strokeRect(
        x * this.cellSize,
        y * this.cellSize,
        this.cellSize,
        this.cellSize
      );
      this.ctx.fillStyle = this.$store.state.colors[color];
      this.ctx.fillRect(
        x * this.cellSize,
        y * this.cellSize,
        this.cellSize,
        this.cellSize
      );
    },
    createNewGeneration() {
      if (this.automatName === 'nonTransitiveAutomat') {
        this.createNonTransitionalFieldGeneration();
      } else {
        this.createGameOfLifeGeneration();
      }
      this.currentGeneration = this.nextGeneration;
      this.nextGeneration = Array.from(
        new Array(this.fieldSize),
        () => new Array(this.fieldSize)
      );
    },
    createNonTransitionalFieldGeneration() {
      var transform = false;
      this.numTransforms = 0;

      for (var i = 0; i < this.fieldSize; i++) {
        for (var j = 0; j < this.fieldSize; j++) {
          this.nextGeneration[i][j] = this.currentGeneration[i][j];
          checkCell: {
            for (var k = i - 1; k <= i + 1; k++) {
              var k1 = k;
              if (k == -1) {
                k1 = this.fieldSize - 1;
              }
              if (k == this.fieldSize) {
                k1 = 0;
              }
              for (var l = j - 1; l <= j + 1; l++) {
                if (k == i && l == j) continue;
                var l1 = l;
                if (l == -1) {
                  l1 = this.fieldSize - 1;
                }
                if (l == this.fieldSize) {
                  l1 = 0;
                }
                transform = false;
                if (
                  this.currentGeneration[i][j] + 1 ==
                  this.currentGeneration[k1][l1]
                ) {
                  transform = true;
                } else if (
                  this.currentGeneration[k1][l1] == 0 &&
                  this.currentGeneration[i][j] == this.numberColors - 1
                ) {
                  transform = true;
                }
                if (transform) {
                  this.numTransforms++;
                  this.nextGeneration[i][j] = this.currentGeneration[k1][l1];
                  this.fillCell(this.currentGeneration[k1][l1], i, j);
                  break checkCell;
                }
              }
            }
          }
        }
      }
    },
    createGameOfLifeGeneration() {
      for (var i = 0; i < this.fieldSize; i++) {
        for (var j = 0; j < this.fieldSize; j++) {
          var neighboursCount = this.countNeighbours(i, j);
          if (this.currentGeneration[i][j] === 0) {
            if (neighboursCount === this.lifeNeighbours.value[1]) {
              this.nextGeneration[i][j] = 1;
              this.fillCell(0, i, j);
            } else {
              this.nextGeneration[i][j] = 0;
            }
          } else {
            // this.currentGeneration[i][j] > 0
            if (
              neighboursCount < this.lifeNeighbours.value[0] ||
              neighboursCount > this.lifeNeighbours.value[1]
            ) {
              this.nextGeneration[i][j] = 0;
              this.fillCell(9, i, j);
            } else {
              this.nextGeneration[i][j] = this.calculateNextLifeAge(i, j);
              this.fillCell(this.nextGeneration[i][j] - 1, i, j);
            }
          }
        }
      }
    },
    calculateNextLifeAge(i, j) {
      var age = this.currentGeneration[i][j] + 1;
      if (age > 8) age = 8;
      return age;
    },
    countNeighbours(i, j) {
      var count = 0;
      var iminus = i - 1;
      if (iminus === -1) {
        iminus = this.fieldSize - 1;
      }
      var iplus = i + 1;
      if (iplus == this.fieldSize) {
        iplus = 0;
      }
      var jminus = j - 1;
      if (jminus === -1) {
        jminus = this.fieldSize - 1;
      }
      var jplus = j + 1;
      if (jplus == this.fieldSize) {
        jplus = 0;
      }
      count += this.currentGeneration[iminus][jminus] > 0 ? 1 : 0;
      count += this.currentGeneration[iminus][j] > 0 ? 1 : 0;
      count += this.currentGeneration[iminus][jplus] > 0 ? 1 : 0;
      count += this.currentGeneration[i][jminus] > 0 ? 1 : 0;
      count += this.currentGeneration[i][jplus] > 0 ? 1 : 0;
      count += this.currentGeneration[iplus][jminus] > 0 ? 1 : 0;
      count += this.currentGeneration[iplus][j] > 0 ? 1 : 0;
      count += this.currentGeneration[iplus][jplus] > 0 ? 1 : 0;

      return count;
    },
    toggleLifeCell(e) {
      if (this.automatName === 'gameOfLife') {
        var canvas = this.$refs.field.getBoundingClientRect();
        const mousePos = {
          x: Math.floor(e.clientX - canvas.left),
          y: Math.floor(e.clientY - canvas.top),
        };
        var i = Math.floor(mousePos.x / this.cellSize);
        var j = Math.floor(mousePos.y / this.cellSize);
        if (i >= this.fieldSize) {
          i = this.fieldSize - 1;
        }
        if (j >= this.fieldSize) {
          j = this.fieldSize - 1;
        }
        if (i < 0) {
          i = 0;
        }
        if (j < 0) {
          j = 0;
        }
        if (this.selectedPatternIndex === -1) {
          if (this.currentGeneration[i][j] > 0) {
            this.currentGeneration[i][j] = 0;
            this.fillCell(9, i, j);
          } else {
            this.currentGeneration[i][j] = 1;
            this.fillCell(0, i, j);
          }
        } else {
          this.$store.state.lifePatterns[
            this.selectedPatternIndex
          ].values.forEach((coordinate) => {
            var x = i + coordinate[0];
            var y = j + coordinate[1];
            if (x >= this.fieldSize) {
              x = x - this.fieldSize;
            }
            if (y >= this.fieldSize) {
              y = y - this.fieldSize;
            }
            this.currentGeneration[x][y] = 1;
            this.fillCell(0, x, y);
          });
        }
      }
    },
  },
};
</script>

<template>
  <div>
    <h1 align="left">{{ getMsg(automatName) }}</h1>

    <div class="app">
      <div class="automat-controls">
        <div class="general-controls">
          <button @click="reset">{{ getMsg('reset') }}</button>
          <button @click="changeGoButton">{{ getMsg(startButtonMsg) }}</button>
          <span>
            <input
              type="radio"
              name="runstep"
              value="flow"
              checked
              v-model="runmodus"
              @change="changeRunModus('flow')"
            />
            {{ getMsg('runflow') }}
            <input
              type="radio"
              name="runstep"
              value="step"
              v-model="runmodus"
              @change="changeRunModus('step')"
            />
            {{ getMsg('runstep') }}
          </span>
        </div>

        <div
          class="nontransitional-controls"
          v-if="setAutomatName === 'nonTransitiveAutomat'"
        >
          <div>
            {{ getMsg('similarity') }}
            <div />
            <div>
              <Slider
                class="slider-blue"
                :min="0"
                :max="0.99"
                :range="0.01"
                v-model="similarity"
                :step="0.01"
                :format="{ decimals: 2 }"
              />
            </div>
            <div>
              <animalPalette
                v-model:numberOfColors="setNumberColors"
                @changeNumberOfColors="setNumberColors = $event"
                :minColors="minColors"
                :maxColors="maxColors"
                >{{ getMsg('numberOfSpeciesInput') }}
              </animalPalette>
            </div>
          </div>
        </div>
        <div class="gameoflife-controls" v-if="setAutomatName === 'gameOfLife'">
            <div>
              <animalPalette
                :numberOfColors="maxColors"
                :minColors="minColors"
                :maxColors="maxColors"
                :showPlusMinusButtons= false
                >{{ getMsg('generationColors') }}
              </animalPalette>
            </div>
            <br>
            <div>
              {{ getMsg('neigboursToLive') }}
            <div />
            <div>
              <Slider
                class="slider-blue"
                :min="1"
                :max="8"
                :range="1"
                v-model="lifeNeighbours.value"
                :step="1"
              />
            </div>
            <div>
              <button @click="clearLifeField">{{ getMsg('clear') }}</button>
            </div>
            <patternSelector
              @selectPatternIndex="selectedPatternIndex = $event"
              >{{ getMsg('selectPattern') }}</patternSelector
            >
          </div>
        </div>
        <div class="automatselection-controls">
          {{ getMsg('automatSelection') }}
          <div>
            <span
              ><input
                type="radio"
                name="automat-selection"
                value="nonTransitiveAutomat"
                checked
                v-model="setAutomatName"
              />
              {{ getMsg('nonTransitiveAutomat') }}</span
            >
            <span
              ><input
                type="radio"
                name="automat-selection"
                value="gameOfLife"
                v-model="setAutomatName"
              />
              {{ getMsg('gameOfLife') }}</span
            >
          </div>
        </div>
      </div>
      <div class="automat-field">
        <div>
          {{ $store.state.messages['generationnumber'] }}
          {{ generationCounter }}
        </div>
        <canvas
          ref="field"
          :width="fieldSize * cellSize"
          :height="fieldSize * cellSize"
          @click="toggleLifeCell"
        ></canvas>
      </div>
    </div>
  </div>
</template>

<!-- Add "scoped" attribute to limit CSS to this component only -->

<style scoped>
.app {
  display: flex;
  flex-wrap: wrap;
}
canvas {
  border: 1px solid;
}
h3 {
  margin: 40px 0 0;
}

a {
  color: #42b983;
}

button {
  background-color: LightSalmon;
}
canvas{
  margin-top: 0;
}
.automat-field {
  border: 1px solid;
  margin-right: 10px;
  padding: 3px;
  padding-bottom: 7px;

}
.automat-controls {
  width: 500px;
  margin-right: 10px;
}
.automat-controls,
.automatselection-controls {
  display: inline-block;
  padding: 10px;
  border: 1px solid;
}

.automatselection-controls {
  background-color: LemonChiffon;
}
.general-controls {
  padding-top: 5px;
  border-bottom: 1px solid;
  padding-bottom: 3px;
  background-color: lavender;
}
.slider-blue {
  margin-top: 50px;
  width: 100%;
  --slider-connect-bg: #3b82f6;
  --slider-tooltip-bg: #3b82f6;
  --slider-handle-ring-color: #3b82f630;
}
</style>
