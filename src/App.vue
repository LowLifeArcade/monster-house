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
  {{ prompt }}
  <input
    autofocus
    ref="input"
    type="text"
    :disabled="disabled"
    v-model="characterAction"
    @keyup.enter="dispatch"
  />
</template>

<script>
// character state: tired, scared, angry, idle, active
const CHARACTER_STATE = { IDLE: "IDLE", ACTION: "ACTION" };
const STORY_STATE = {
  DOOR: {
    value: "DOOR",
    subStates: {
      beforeKnock: { value: "before the knock" },
      afterKnock: { value: "after the knock" },
      someoneAtDoor: {value: "someone is at the door, but the door has not opened"}
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
      characterState: CHARACTER_STATE.IDLE,
      storyState: STORY_STATE.DOOR.value,
      subStoryState: STORY_STATE.DOOR.subStates.beforeKnock.value,
      message: "You see a door",
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
    focus(){
        this.$nextTick(function() {
            this.$refs.input.focus()
        })
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
        this.disabled = true
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
                    this.focus()
                  // new if statements available
                }, 3000);
              }, 2000);
              break;

            default:
                this.disabled = false
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
                      this.message = "And jiggles..."
                      this.subStoryState = STORY_STATE.DOOR.subStates.someoneAtDoor.value
                      this.disabled = false
                      this.focus()
                  }, 2000);
                }, 3000);
              }, 3000);
              break;

            default:
                this.disabled = false
              break;
          }
        }

        if (this.subStoryState === STORY_STATE.DOOR.subStates.someoneAtDoor.value) {
            switch (true) {
                case actionName.includes('run'):
                    this.message = "you ran away..."
                    break;
            
                default:
                    this.disabled = false
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
</style>
