<template>
  <div class="h-container">
    <div class="v-container">
      <main :style="cssVars">
        <div class="status">
          <div>
            <b-button-group>
              <b-dropdown right :text="level">
                <b-dropdown-item @click="setBeginnerDifficulty">Beginner</b-dropdown-item>
                <b-dropdown-item @click="setIntermediateDifficulty">Intermediate</b-dropdown-item>
                <b-dropdown-item @click="setExpertDifficulty">Expert</b-dropdown-item>
              </b-dropdown>
            </b-button-group>
          </div>
          <button @click="resetGame">{{ gameStatus }}</button>
          <div>
            <b-icon-flag></b-icon-flag>
            {{ bombsRemaining | addLeadingZeros }}
          </div>

          <div>
            <b-icon-alarm></b-icon-alarm>
            {{ secondsPassed | addLeadingZeros }}
          </div>
        </div>
        <div class="board">
          <div
            class="tile"
            v-for="(tile, i) in tiles"
            :key="i"
            :class="{
                          revealed: tile.revealed,
                          // If the game failed mark tile as a wrong pick                          
                          'wrong-pick': 
                          gameFailed &amp;&amp; ((tile.bomb &amp;&amp; tile.revealed) || (!tile.bomb &amp;&amp; tile.flagged))
                        }"
            :data-surrounding-bombs="tile.surroundingBombs"
            @click="reveal(i)"
            @contextmenu.prevent="flag(i)"
          >
            <template v-if="tile.flagged">
              <b-icon icon="flag-fill" style="color: red"></b-icon>
            </template>
            <template v-else-if="tile.revealed &amp;&amp; tile.bomb">
              <b-icon icon="circle-fill"></b-icon>
            </template>
            <template
              v-else-if="tile.revealed &amp;&amp; tile.surroundingBombs"
            >{{ tile.surroundingBombs }}</template>
          </div>
        </div>
      </main>
    </div>
  </div>
</template>

<script>
const getIndex = (row, column, config) => {
  // out of field
  if (row < 0) return;
  if (column < 0) return;
  if (row >= config.height) return;
  if (column >= config.width) return;

  // Return index
  return row * config.width + column;
};

const getTileCoordinates = (index, config) => ({
  row: Math.floor(index / config.width),
  column: index % config.width,
});

const generateTiles = (config) => {
  // Main array
  const bombs = Array.from({ length: config.width * config.height });

  // Put bombs to random positions
  let bombsPlanted = 0;
  while (bombsPlanted != config.totalNumberOfBombs) {
    // Random index
    const index = Math.floor(Math.random() * config.width * config.height);

    // If tile doesn't already contain a bomb then plant it
    if (!bombs[index]) {
      bombs[index] = true;
      bombsPlanted++;
    }
  }

  return bombs.map((bomb, i, array) => {
    const { row, column } = getTileCoordinates(i, config);

    // Count number of bombs around a tile
    let surroundingBombs = 0;
    if (array[getIndex(row - 1, column - 1, config)]) surroundingBombs++;
    if (array[getIndex(row - 1, column - 0, config)]) surroundingBombs++;
    if (array[getIndex(row - 1, column + 1, config)]) surroundingBombs++;
    if (array[getIndex(row - 0, column - 1, config)]) surroundingBombs++;
    if (array[getIndex(row - 0, column + 1, config)]) surroundingBombs++;
    if (array[getIndex(row + 1, column - 1, config)]) surroundingBombs++;
    if (array[getIndex(row + 1, column - 0, config)]) surroundingBombs++;
    if (array[getIndex(row + 1, column + 1, config)]) surroundingBombs++;

    return {
      bomb, // The tile contains a bomb
      flagged: false, // The tile is flagged
      revealed: false, // The tile has been revealed
      surroundingBombs, // The number of bombs in the surrounding fields
    };
  });
};

export default {
  data: () => ({
    config: {}, // Initialized in created lifecycle hook(?!?)
    tiles: [],

    // Timer
    secondsPassed: 0, // Seconds passed since the game started
    timerIntervalID: undefined,

    //level, default í Beginner
    levelS: "Beginner",
  }),
  created() {
    // At initialization set the game difficulty to beginner
    this.setBeginnerDifficulty();
  },
  computed: {
    cssVars() {
      return {
        "--width": this.config.width,
        "--height": this.config.height,
        "--size": `${this.config.size}px`,
      };
    },
    bombsRemaining() {
      // Reduce total number of bombs by the number of flagged tiles
      const numberOfFlaggedTiles = this.tiles.filter((t) => t.flagged).length;
      return this.config.totalNumberOfBombs - numberOfFlaggedTiles;
    },
    gameIsProgress() {
      // game has ended already
      if (this.gameStatus == "😞" || this.gameStatus == "😎") return false;
      // game haven't even started
      if (!this.tiles.find((tile) => tile.revealed)) return false;
      // Otherwise the game is in progress
      return true;
    },
    gameWon() {
      const numberOfRevealedTiles = this.tiles.filter((t) => t.revealed).length;

      // To win the game you need to reveal all the tiles that do not contain a bomb
      const numberOfTilesThatNeedsToBeRevealed =
        this.config.width * this.config.height - this.config.totalNumberOfBombs;

      return numberOfRevealedTiles == numberOfTilesThatNeedsToBeRevealed;
    },
    gameFailed() {
      // Once you reveal a tile that contains a bomb you failed
      if (this.tiles.find((tile) => tile.bomb && tile.revealed)) {
        //SHow all bombs that are not flagged
        this.tiles.forEach((tile) => {
          if (tile.bomb && !tile.flagged) {
            tile.revealed = true;
          }
        });
        return true;
      }
      return false;
    },
    gameStatus() {
      if (this.gameFailed) return "😞";
      if (this.gameWon) return "😎";
      return "😣";
    },
    level() {
      if (this.levelS == "Beginner") {
        return "Beginner";
      }
      if (this.levelS == "Intermediate") {
        return "Intermediate";
      }
      return "Expert";
    },
  },
  watch: {
    // Start / stop clock
    gameIsProgress(value) {
      if (value) {
        // Once the game started start counting the seconds
        this.timerIntervalID = setInterval(() => {
          this.secondsPassed++;
        }, 1000);
      } else {
        // Once the game ended stop the timer
        clearInterval(this.timerIntervalID);
      }
    },
  },
  methods: {
    setBeginnerDifficulty() {
      this.config = {
        width: 10,
        height: 8,
        totalNumberOfBombs: 10,
        size: 50,
      };
      this.levelS = "Beginner";
      this.resetGame();
    },
    setIntermediateDifficulty() {
      this.config = {
        width: 18,
        height: 14,
        totalNumberOfBombs: 40,
        size: 35,
      };
      this.levelS = "Intermediate";
      this.resetGame();
    },
    setExpertDifficulty() {
      this.config = {
        width: 24,
        height: 20,
        totalNumberOfBombs: 99,
        size: 25,
      };
      this.levelS = "Expert";
      this.resetGame();
    },

    resetGame() {
      // Reset tiles based on the config
      this.tiles = generateTiles(this.config);

      // Reset timer
      this.secondsPassed = 0;
    },
    reveal(i) {
      // Do nothing if the game has already failed
      if (this.gameFailed) return;

      // Do nothing in case of wrong input
      if (i == undefined) return;

      const tile = this.tiles[i];

      // You can't reveal a flagged tile, you need to unflag it first
      if (tile.flagged) return;

      // Only reveal a tile if it is not revealed already
      if (!tile.revealed) {
        // Reveal tile
        tile.revealed = true;

        // If the tile is empty, also reveal the neighbour tiles
        if (!tile.bomb && tile.surroundingBombs == 0) {
          const { row, column } = getTileCoordinates(i, this.config);

          this.reveal(getIndex(row - 1, column - 1, this.config)); // Reveal top left neighbour
          this.reveal(getIndex(row - 1, column - 0, this.config)); // Reveal top neighbour
          this.reveal(getIndex(row - 1, column + 1, this.config)); // Reveal top right neighbour
          this.reveal(getIndex(row - 0, column - 1, this.config)); // Reveal left neighbour
          this.reveal(getIndex(row - 0, column + 1, this.config)); // Reveal right neighbour
          this.reveal(getIndex(row + 1, column - 1, this.config)); // Reveal bottom left neighbour
          this.reveal(getIndex(row + 1, column - 0, this.config)); // Reveal bottom neighbour
          this.reveal(getIndex(row + 1, column + 1, this.config)); // Reveal bottom right neighbour
        }
      }
    },
    flag(i) {
      // Do nothing if the game has already failed
      if (this.gameFailed) return;

      // Do nothing if the tile is alerady revealed
      if (this.tiles[i].revealed) return;

      // Flag or unflag tile
      this.tiles[i].flagged = !this.tiles[i].flagged;
    },
  },
  filters: {
    addLeadingZeros: function (value) {
      // Add leading zeros. E.g. 1 -> 001, 10 -> 010, 100 -> 100
      return ("00" + value).slice(-3);
    },
  },
};
</script>

<style lang="scss">
$background-color: #006989;
$tile-color: #d4d4d4;
$border-radius: calc(var(--size) / 10);

.h-container {
  // margin:20px;
  display: flex;
  flex-direction: row;
  justify-content: center;
  height: 100%;
}

.v-container {
  margin: 30px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  height: 100%;
}

button {
  background-color: inherit;
  border: none;
  cursor: pointer;
  font-family: inherit;
  font-size: inherit;

  &:focus {
    outline: none;
  }
}

a:visited {
  color: inherit;
}

@mixin add-shadow($offset) {
  $opposite: calc(#{$offset} * -1);
  box-shadow: inset $offset $offset 0px 0px rgba(255, 255, 255, 0.45),
    inset $opposite $opposite 0px 0px rgba(0, 0, 0, 0.25);
}

main {
  background-color: rgb(190, 255, 215);
  border-radius: $border-radius;
  font-family: "Ubuntu Mono", monospace;
  font-weight: 700;
  padding: 0;
}

.status {
  align-items: center;
  color: #abb926;
  background-color: rgb(74, 117, 44);
  display: flex;
  flex-direction: row;
  font-size: 2em;
  justify-content: space-between;
  padding: 5px 15px;

  button {
    box-shadow: 0 2.8px 7.1px rgba(0, 0, 0, 0.124),
      0 5px 14px rgba(0, 0, 0, 0.2);

    border-radius: 5px;
  }
}

.board {
  display: grid;
  grid-template-columns: repeat(var(--width), auto);
  grid-template-rows: repeat(var(--height), auto);
  user-select: none;
}

.tile {
  width: var(--size);
  height: var(--size);
  line-height: var(--size);

  &:not(.revealed) {
    $shadow: calc(var(--size) / 12.5);
    @include add-shadow($shadow);
    border-radius: $border-radius;
    cursor: pointer;
  }

  &.revealed {
    border: 1px solid #707070;
    box-sizing: border-box;
    background-color: rgb(215, 184, 153);
  }

  &.wrong-pick {
    background-color: rgb(255, 100, 100);
  }

  &[data-surrounding-bombs="1"] {
    color: blue;
  }
  &[data-surrounding-bombs="2"] {
    color: green;
  }
  &[data-surrounding-bombs="3"] {
    color: red;
  }
  &[data-surrounding-bombs="4"] {
    color: purple;
  }
  &[data-surrounding-bombs="5"] {
    color: maroon;
  }
  &[data-surrounding-bombs="6"] {
    color: turquoise;
  }
  &[data-surrounding-bombs="7"] {
    color: black;
  }
  &[data-surrounding-bombs="8"] {
    color: gray;
  }
}
</style>
