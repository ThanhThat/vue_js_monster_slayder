<template>
  <div>
    <header>
      <h1>Monster Slayder {{ turn }}</h1>
    </header>
    <div id="game">
      <section id="monster" class="container">
        <h2>Monster Health</h2>
        <span class="health__value">{{ monsterHealth }}%</span>

        <div class="healthbar">
          <div class="healthbar__value" :style="monsterBarHealthStyle"></div>
        </div>
      </section>

      <section id="player" class="container">
        <h2>Your Health</h2>
        <span class="health__value">{{ playerHealth }}%</span>

        <div class="healthbar">
          <div class="healthbar__value" :style="playerBarHealthStyle"></div>
        </div>
      </section>

      <section id="controls" v-if="!winner">
        <button @click="handleAttackMonster">ATTACK</button>
        <button :disabled="canUseSpecialAttack" @click="handleSpecialAttack">SPECIAL ATTACK</button>
        <button :disabled="canUseHeal" @click="handleHeal">HEAL</button>
        <button :disabled="canUserSurrender" @click="handleSurrender">SURRENDER</button>
      </section>

      <section class="container" v-if="winner">
        <h2>Game Over</h2>
        <h3 v-if="winner === 'player'">You win!</h3>
        <h3 v-else-if="winner === 'monster'">You close!</h3>
        <h3 v-else-if="winner === 'draw'">It's a draw!</h3>
        <button @click="playAgain">Play again!</button>
      </section>

      <section id="log" class="container">
        <h2>Battle Log</h2>
        <ul>
          <!-- <li>
            <span class="log--player">Player</span>&nbsp;<span>Attack</span>&nbsp;<span
              class="log--damage"
              >and deal 20</span
            >
          </li>

          <li>
            <span class="log--monster">Monster</span>&nbsp;<span>Attack</span>&nbsp;<span
              class="log--damage"
              >and deal 25</span
            >
          </li>
          <li>
            <span class="log--player">Player</span>&nbsp;<span>Health</span>&nbsp;<span
              class="log--heal"
              >20</span
            >
          </li> -->

          <li v-for="log in logs" :key="log.id">
            <span>{{ log.id }} - </span>
            <span
              :class="{
                'log--player': log.who === 'player',
                'log--monster': log.who === 'monster'
              }"
              >{{ log.who }}</span
            >&nbsp;

            <span class="log--heal" v-if="log.actionType === 'health'"
              >Yourself for {{ log.actionValue }}</span
            >
            <span class="log--damage" v-else-if="log.actionType === 'attack'"
              >Attack {{ log.actionValue }}</span
            >
            <span class="log--damage" v-else-if="log.actionType === 'special attack'"
              >Special Attack {{ log.actionValue }}</span
            >
            <span v-else-if="log.actionType === 'surrender'">Surrender</span>
          </li>
        </ul>
      </section>
    </div>
  </div>
</template>

<script>
function getRandomValue(min, max) {
  return Math.round(Math.random() * (max - min) + min)
}

export default {
  data() {
    return {
      playerHealth: 100,
      monsterHealth: 100,
      winner: null,
      turn: 0,
      logs: []
    }
  },

  methods: {
    handleAttackMonster() {
      ++this.turn

      const userDame = getRandomValue(4, 8)
      this.monsterHealth -= userDame

      this.handleAttackUser()

      if (this.monsterHealth < 0) this.monsterHealth = 0

      this.addLog('player', 'attack', userDame)
    },

    handleAttackUser() {
      const monsterDame = getRandomValue(5, 10)
      this.playerHealth -= monsterDame

      if (this.playerHealth < 0) this.playerHealth = 0

      this.addLog('monster', 'attack', monsterDame)
    },

    handleSpecialAttack() {
      this.turn++
      const userDame = getRandomValue(10, 15)
      this.monsterHealth -= userDame

      if (this.monsterHealth < 0) this.monsterHealth = 0

      this.handleAttackUser()

      this.addLog('player', 'special attack', userDame)
    },

    handleSurrender() {
      this.turn++
      this.winner = 'monster'

      this.addLog('player', 'surrender')
    },

    handleHeal() {
      this.turn++
      const health = getRandomValue(5, 10)

      if (this.playerHealth + health > 100) {
        this.playerHealth = 100
      } else {
        this.playerHealth += health
      }

      this.addLog('player', 'health', health)

      this.handleAttackUser()
    },

    addLog(who, actionType, actionValue) {
      const logItem = {
        id: this.turn,
        who,
        actionType,
        actionValue
      }

      this.logs.unshift(logItem)
    },

    playAgain() {
      this.playerHealth = 100
      this.monsterHealth = 100
      this.winner = null
      this.turn = 0
      this.logs = []
    }
  },

  computed: {
    playerBarHealthStyle() {
      if (this.playerHealth < 0) {
        return { width: '0%' }
      }

      return { width: `${this.playerHealth}%` }
    },

    monsterBarHealthStyle() {
      if (this.monsterHealth < 0) {
        return { width: '0%' }
      }

      return {
        width: `${this.monsterHealth}%`
      }
    },
    canUseSpecialAttack() {
      return this.turn === 0 || this.turn % 3 !== 0
    },

    canUseHeal() {
      return this.playerHealth === 100
    },

    canUserSurrender() {
      return this.turn === 0
    }
  },

  watch: {
    playerHealth: {
      handler(newPlayerHealth) {
        if (newPlayerHealth === 0 && newPlayerHealth === this.monsterHealth) {
          this.winner = 'draw'
        } else if (newPlayerHealth === 0) {
          this.winner = 'monster'
        }
      }
    },

    monsterHealth: {
      handler(newMonsterHealth) {
        if (newMonsterHealth === 0 && newMonsterHealth === this.playerHealth) {
          this.winner = 'draw'
        } else if (newMonsterHealth === 0) {
          this.winner = 'player'
        }
      }
    }
  }
}
</script>

<style lang="scss" scoped>
* {
  box-sizing: border-box;
}

html {
  font-family: 'Jost', sans-serif;
}

body {
  margin: 0;
}

header {
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.26);
  padding: 0.5rem;
  background-color: #880017;
  color: white;
  text-align: center;
  margin-bottom: 2rem;
}

section {
  width: 90%;
  max-width: 40rem;
  margin: auto;
}

.healthbar {
  width: 100%;
  height: 40px;
  border: 1px solid #575757;
  margin: 1rem 0;
  background: #fde5e5;
}

.healthbar__value {
  background-color: #00a876;
  width: 100%;
  height: 100%;
}

.container {
  text-align: center;
  padding: 0.5rem;
  margin: 1rem auto;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.26);
  border-radius: 12px;

  position: relative;
}

.health__value {
  color: red;
  font-size: 1.5rem;

  position: absolute;
  top: 15px;
  right: 15px;
}

#monster h2,
#player h2 {
  margin: 0.25rem;
}

#controls {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  align-items: center;
  justify-content: center;
}

button {
  font: inherit;
  border: 1px solid #88005b;
  background-color: #88005b;
  color: white;
  padding: 1rem 2rem;
  border-radius: 12px;
  margin: 1rem;
  width: 12rem;
  cursor: pointer;
  box-shadow: 1px 1px 4px rgba(0, 0, 0, 0.26);
}

button:focus {
  outline: none;
}

button:hover,
button:active {
  background-color: #af0a78;
  border-color: #af0a78;
  box-shadow: 1px 1px 8px rgba(0, 0, 0, 0.26);
}

button:disabled {
  background-color: #ccc;
  border-color: #ccc;
  box-shadow: none;
  color: #3f3f3f;
  cursor: not-allowed;
}

#log ul {
  list-style: none;
  margin: 0;
  padding: 0;
}

#log li {
  margin: 0.5rem 0;
}

.log--player {
  color: #7700ff;
}

.log--monster {
  color: #da8d00;
}

.log--damage {
  color: red;
}

.log--heal {
  color: green;
}
</style>
