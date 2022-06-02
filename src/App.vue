<template>
  <div class="game">
    <div class="title" v-if="gameState === 'IN_PROGRESS'">
      <h2>
        <!-- {{ storyState }} -->
      </h2>
      <div>
        <!-- {{ subStoryState }} -->
      </div>
    </div>
    <div class="game-message">
      <p>
        {{ message }}
      </p>
    </div>
    <div class="prompt" v-if="gameState === 'IN_PROGRESS' && !disabled">
      {{ prompt }}
    </div>
    <input
      v-if="gameState === 'IN_PROGRESS' && !disabled"
      autofocus
      ref="input"
      type="text"
      :disabled="disabled"
      v-model="characterAction"
      @keyup.enter="dispatch"
    />
    <div v-if="gameState === 'START'">
      <button @click="startGame">Start Game?</button>
    </div>
    <div class="hide" :class="{ end: gameState === 'END' }">
      <button @click="startGame">Try Again?</button>
    </div>
  </div>
</template>

<script>
// character state: tired, scared, angry, idle, active
const CHARACTER_STATE = { IDLE: "IDLE", ACTION: "ACTION" };
const GAME_STATE = { START: "START", IN_PROGRESS: "IN_PROGRESS", END: "END" };
const STORY_STATE = {
  DOOR: {
    value: "At the door",
    subStates: {
      beforeKnock: { value: "You haven't done anything yet" },
      afterKnock: { value: "After the knock" },
      someoneAtDoor: {
        value: "Someone is at the door, but the door has not opened",
      },
    },
  },
};

export default {
  data() {
    return {
      gameState: GAME_STATE.START,
      characterState: CHARACTER_STATE.IDLE,
      storyState: STORY_STATE.DOOR.value,
      subStoryState: STORY_STATE.DOOR.subStates.beforeKnock.value,
      message: "Monster House",
      prompt: "What will you do?",
      characterAction: "",
      disabled: false,
      //   stateMachine: { ...GAME_STATE },
    };
  },
  methods: {
    submitAction(e) {
      this.stateMachine.dispatch(e.target.value);
      this.characterAction = "";
    },
    focus() {
      this.$nextTick(function () {
        this.$refs.input.focus();
      });
    },
    startGame() {
      this.gameState = "IN_PROGRESS";
      this.storyState = STORY_STATE.DOOR.value;
      this.subStoryState = STORY_STATE.DOOR.subStates.beforeKnock.value;
      (this.message = "You see a door"), (this.disabled = false);
      this.focus();
    },
    knock() {
      console.log("reached");
      //   this.gameSubState = "afterKnock";
      this.message = "You knocked";
      setTimeout(() => {
        this.message = "There was no answer";
        setTimeout(() => {
          this.message = "You hear a scream";
          this.subStoryState = STORY_STATE.DOOR.subStates.afterKnock.value;
          // new if statements available
        }, 3000);
      }, 1800);
      //   this.character_action = "";
    },
    dispatch(e) {
      this.disabled = true;
      let actionName = e.target.value;
      actionName = actionName.toLowerCase();

      this.characterAction = "";
      this.stateReducer(actionName);
    },
    async stateReducer(actionName) {
      /**
       *
       * @param {string} storyState Story state to advance to
       * @param {number} time time of message in seconds
       */
      const advanceStory = (storyState, time = 1.2) => {
        return new Promise((res) => {
          setTimeout(() => {
            res(
              (() => {
                this.storyState = storyState;
                this.disabled = false;
                this.focus();
              })()
            );
          }, time * 1000);
        });
      };
      /**
       *
       * @param {string} subStoryState Sub story state to advance to
       * @param {number} time time of message in seconds
       */
      const advanceSubStory = (subStoryState, time = 1.2) => {
        return new Promise((res) => {
          setTimeout(() => {
            res(
              (() => {
                this.subStoryState = subStoryState;
                this.disabled = false;
                this.focus();
              })()
            );
          }, time * 1000);
        });
      };
      /**
       *
       * @param {string} gameState Game state to change to
       * @param {number} time time till state change in seconds
       */
      const setGameState = (gameState, time = 1.2) => {
        return new Promise((res) => {
          setTimeout(() => {
            res(
              (() => {
                this.gameState = gameState;
              })()
            );
          }, time * 1000);
        });
      };
      /**
       *
       * @param {string} msg Game message
       * @param {number} time time of message in seconds
       * @param {boolean} end sets whether this is the end of the section or not
       */
      const gameMessage = (msg, time = 1, end = false) => {
        return new Promise((res) => {
          setTimeout(() => {
            res(
              (() => {
                this.message = msg;
                this.disabled = !end;
                end && this.focus();
              })()
            );
          }, time * 1000);
        });
      };

      if ((this.storyState = STORY_STATE.DOOR.value)) {
        if (
          this.subStoryState === STORY_STATE.DOOR.subStates.beforeKnock.value
        ) {
          switch (true) {
            case actionName.includes("kick" && "door"):
              await gameMessage("You kicked the door", 0);
              await gameMessage("You kicked the door again", 1.5);
              await gameMessage("And again", 1.5);
              await gameMessage("No response", 1.5, true);
              return;

            case actionName.includes("knock"):
              await gameMessage("You knock", 0);
              await gameMessage("There is no answer", 1.8);
              await gameMessage("You hear a scream", 2.8);
              await advanceSubStory(
                STORY_STATE.DOOR.subStates.afterKnock.value
              );
              return;

            // TODO: use regex to make case for ruuuun or rrrruuuunnn
            case actionName.includes("run"):
              await gameMessage("You ran away...", 0);
              await setGameState(GAME_STATE.END);
              return;

            default:
              this.disabled = false;
              break;
          }
        }

        if (
          this.subStoryState === STORY_STATE.DOOR.subStates.afterKnock.value
        ) {
          switch (true) {
            case actionName.includes("knock"):
              await gameMessage("You knock again...", 0);
              await gameMessage("The sound of footsteps down a hallway", 2.3);
              await gameMessage("The door handle jiggles", 2.7);
              await gameMessage("And jiggles...", 2);
              await advanceSubStory(
                STORY_STATE.DOOR.subStates.someoneAtDoor.value
              );
              return;

            case actionName.includes("run"):
              await gameMessage("You ran away...", 0);
              return await setGameState(GAME_STATE.END);

            default:
              this.disabled = false;
              break;
          }
        }

        if (
          this.subStoryState === STORY_STATE.DOOR.subStates.someoneAtDoor.value
        ) {
          switch (true) {
            case actionName.includes("knock"):
              await gameMessage("The jiggling stops...", 0);
              await gameMessage("The door handle jiggles again...", 1.8);
              await advanceSubStory(
                STORY_STATE.DOOR.subStates.someoneAtDoor.value
              );
              return;
            case actionName.includes("run"):
              await gameMessage("You ran away...", 0);
              return await setGameState(GAME_STATE.END);

            default:
              this.disabled = false;
              break;
          }
        }
      }
    },
  },
};
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}
body {
  background: rgb(11, 11, 11);
  /* background-image: linear-gradient(to bottom, rgba(0, 0, 0, 0.706), rgba(3, 2, 2, 0.697)), url('/Eggener-Haunted.jpeg'); */
  background-repeat: no-repeat;
  height: 100vh;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  display: grid;
  place-content: center;
  color: #2c3e50;
  padding-top: 100px;
  margin: 0;
  padding: 0;
}
.game {
  width: 300px;
  padding-top: 200px;
  display: flex;
  flex-direction: column;
  align-items: center;
}

.title {
  color: rgb(164, 164, 164);
  display: flex;
  flex-direction: column;
  align-items: center;
}

.game-message {
  color: white;
  width: 100%;
  height: 150px;
  padding: 50px;
  margin: 20px;
  background: black;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  box-shadow: 0px 0px 50px rgba(229, 229, 229, 0.271);
}
.prompt {
  color: rgb(197, 197, 197);
}
.hide {
  opacity: 0;
}
.end {
  opacity: 1;
  transition: opacity 2.2s ease-in-out;
}
button {
  color: rgb(220, 220, 220);
  font-weight: 700;
  background: linear-gradient(40deg, rgb(0, 0, 0), rgb(67, 1, 1));
  border: none;
  padding: 10px 20px;
  border-radius: 3px;
  box-shadow: 0px 0px 10px #85858548;
}
input {
  font-size: 16px;
  color: rgb(0, 0, 0);
  font-weight: 700;
  margin-top: 8px;
  padding: 8px 20px;
  background-color: #929292;
  border: none;
  border-radius: 3px;
  outline: none;
}
input[type="text"]:disabled {
  background-color: rgb(11, 11, 11);
}
</style>
