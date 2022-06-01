<template>
  <div class="game-message">
    <!-- <div>
      {{ storyState }}
    </div>
    <div>
      {{ subStoryState }}
    </div> -->

    <p>
      {{ message }}
    </p>
  </div>
  <div v-if="gameState === 'IN_PROGRESS'">
    {{ prompt }}
  </div>
  <input
    v-if="gameState === 'IN_PROGRESS'"
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
  <div class="hide" :class="{'end': gameState === 'END'}">
    <button @click="startGame">Try Again?</button>
  </div>
</template>

<script>
// character state: tired, scared, angry, idle, active
const CHARACTER_STATE = { IDLE: "IDLE", ACTION: "ACTION" };
const GAME_STATE = { START: "START", IN_PROGRESS: "IN_PROGRESS", END: "END" };
const STORY_STATE = {
  DOOR: {
    value: "DOOR",
    subStates: {
      beforeKnock: { value: "before the knock" },
      afterKnock: { value: "after the knock" },
      someoneAtDoor: {
        value: "someone is at the door, but the door has not opened",
      },
    },
  },
};
// const STORY_STATE = {
//     DOOR: {
//         initState: 'beforeKnock',
//         actions: {
//             beforeKnock: {
//                 knock() {
//                     this.message = 'You knocked'
//                 },
//                 inspect() {
//                     this.message = 'You looked around, but found nothing';
//                 },
//             },
//             afterKnock: {
//                 lookAround: 'look around',
//                 runAway: 'run away',
//             },
//         },
//     },
//     dispatch(actionName) {
//         const action = this.storyStates[this.gameState].actions[this.subStoryState][actionName];

//         if (action) {
//             action.call(this);
//         } else {
//             this.prompt = "Unsure what you're trying to do";
//         }
//     },
// };
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
      }, 2000);
      //   this.character_action = "";
    },
    dispatch(e) {
      this.disabled = true;
      let actionName = e.target.value;
      console.log(actionName);
      this.characterAction = "";
      this.stateReducer(actionName);
      // const action =
      //     this.storyStates[this.gameState].actions[this.subStoryState][actionName];
      // if (action) {
      //     action.call(this);
      // } else {
      //     this.prompt = "Unsure what you're trying to do";
      // }
    },
    stateReducer(actionName) {
      if ((this.storyState = STORY_STATE.DOOR.value)) {
        if (
          this.subStoryState === STORY_STATE.DOOR.subStates.beforeKnock.value
        ) {
          switch (true) {
            case actionName.includes("knock"):
              this.message = "You knocked";
              setTimeout(() => {
                this.message = "There was no answer";
                setTimeout(() => {
                  this.message = "You hear a scream";
                  this.subStoryState =
                    STORY_STATE.DOOR.subStates.afterKnock.value;
                  this.disabled = false;
                  this.focus();
                  // new if statements available
                }, 3000);
              }, 2000);
              break;
            case actionName.includes("run"):
              this.message = "You ran away...";
              setTimeout(() => {
                this.gameState = GAME_STATE.END;
              }, 1000);
              break;
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
              this.message = "You knocked again...";
              setTimeout(() => {
                this.message = "The sound of footsteps down a hallway";
                setTimeout(() => {
                  this.message = "The door handle jiggles";
                  // new if statements available
                  setTimeout(() => {
                    this.message = "And jiggles...";
                    this.subStoryState =
                      STORY_STATE.DOOR.subStates.someoneAtDoor.value;
                    this.disabled = false;
                    this.focus();
                  }, 2000);
                }, 3000);
              }, 3000);
              break;

            default:
              this.disabled = false;
              break;
          }
        }

        if (
          this.subStoryState === STORY_STATE.DOOR.subStates.someoneAtDoor.value
        ) {
          switch (true) {
            case actionName.includes("run"):
              this.message = "You ran away...";
              setTimeout(() => {
                this.gameState = GAME_STATE.END;
              }, 1000);
              break;

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
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  color: #2c3e50;
  margin-top: 60px;
}
.game-message {
  color: white;
  width: 100%;
  padding: 50px;
  margin: 20px;
  background: black;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}
.hide {
    opacity: 0;
}
.end {
    opacity: 1;
    transition: opacity 2.2s ease-in-out;
}
button {
    color: white;
    font-weight: 700;
    background: linear-gradient(40deg, slateblue, firebrick);
    border: none;
    padding: 10px 20px;
    border-radius: 3px;
    box-shadow: 2px 4px 4px #2c3e506b;
}
input {
    margin-top: 5px;
    padding: 8px 20px;
}
</style>
