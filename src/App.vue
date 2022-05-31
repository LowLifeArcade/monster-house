<template>
    <div class="game-message">
        <div>
            {{ gameState }}
        </div>
        <div>
            {{ gameSubState }}
        </div>
        <p>
            {{ message }}
        </p>
    </div>
    {{ prompt }}
    <input type="text" v-model="characterAction" @keyup.enter="submitAction" />
</template>

<script>
    const CHARACTER_STATE = { IDLE: 'IDLE', ACTION: 'ACTION' };
    // game state : title, active, game over
    // story state: episodes and how they branch
    // episode state: DOOR episode has certain actions and messages it displays
    // character state: tired, scared, angry, idle, active
    const STORY_STATE = {
        initialState: 'DOOR',
        DOOR: {
            actions: {
                enterCeler() {
                    this.storyState = 'CELER';
                },
            },
        },
    };
    const GAME_STATE = {
        state: 'DOOR',
        actions: {
            DOOR: {
                state: 'beforeKnock',
                actions: {
                    beforeKnock: {
                        knock() {
                            console.log(this.knock);

                            this.knock();
                        },
                        inspect() {
                            this.message = 'You looked around, but found nothing';
                        },
                    },
                    afterKnock: {
                        lookAround: 'look around',
                        runAway: 'run away',
                    },
                },
            },
        },
        dispatch(actionName) {
            // const action = this.state[this.gameState].actions[this.gameSubState][actionName];
            const action =
                this.actions[this.state].actions[this.actions[this.state].state][actionName];
            // console.log(this.actions[this.state].actions[this.actions[this.state].state][actionName]);
            if (action) {
                action.call(this);
            } else {
                this.prompt = "Unsure what you're trying to do";
            }
        },
    };
    export default {
        data() {
            return {
                characterState: CHARACTER_STATE.IDLE,
                gameState: GAME_STATE.state,
                gameSubState: GAME_STATE.actions.DOOR.state,
                message: 'You see a door',
                prompt: 'What will you do?',
                characterAction: '',
                stateMachine: { ...GAME_STATE },
            };
        },
        methods: {
            submitAction(e) {
                if (this.gameState === GAME_STATE.state)
                    if ((this.characterState = CHARACTER_STATE.IDLE)) {
                        if (this.characterAction == 'knock') {
                            this.stateMachine.dispatch('knock');
                        }
                        this.characterAction = '';
                    }
                this.characterAction = '';
            },
            knock() {
                console.log('reached');
                this.gameSubState = 'afterKnock';
                this.message = 'You knocked';
                setTimeout(() => {
                    this.message = 'There was no answer';
                    setTimeout(() => {
                        this.message = 'You hear a scream';
                        // new if statements available
                    }, 3000);
                }, 2000);
                this.character_action = '';
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
