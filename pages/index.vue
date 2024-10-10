<script setup>
import { ref, computed } from "vue"

// State Management
const cards = ref([])
const allowRollDice = ref(true)
const allowSelectCard = ref(false)
const gameOverModel = ref({})
const chosenNumbers = ref([0, 0])

// Computed Properties
const chosenNumber = computed(() =>
  chosenNumbers.value.reduce((a, b) => a + b, 0)
)

const selectedNumbersSum = computed(() => {
  return cards.value
    .filter((c) => c.status === "chosen")
    .reduce((sum, c) => sum + c.id, 0)
})

// Game Functions
const playGame = () => {
  cards.value = Array.from({ length: 12 }, (_, index) => ({
    id: index + 1,
    status: "active",
  }))

  resetGameState()
}

const resetGameState = () => {
  allowRollDice.value = true
  allowSelectCard.value = false
  gameOverModel.value = { show: false, message: "", img: "" }
  chosenNumbers.value = [0, 0]
}

const rollDice = () => {
  if (!allowRollDice.value) return

  allowRollDice.value = false
  chosenNumbers.value = chosenNumbers.value.map(
    () => Math.floor(Math.random() * 6) + 1
  )

  examineGameOver()
  allowSelectCard.value = true
}

const selectCard = (cardId) => {
  const card = cards.value[cardId - 1]
  if (
    card.status === "inactive" ||
    !allowSelectCard.value ||
    allowRollDice.value
  )
    return

  card.status = card.status === "chosen" ? "active" : "chosen"
  examineResults()
  checkIfGameWon()
}

const examineResults = () => {
  if (selectedNumbersSum.value === chosenNumber.value) {
    cards.value.forEach((card) => {
      if (card.status === "chosen") card.status = "inactive"
    })
    resetGameState()
  }
}

const checkIfGameWon = () => {
  if (cards.value.every((card) => card.status === "inactive")) {
    showModel("win")
  }
}

const examineGameOver = () => {
  const activeCards = cards.value.filter((c) => c.status === "active")

  if (activeCards.length === 1 && activeCards[0].id !== chosenNumber.value) {
    showModel("lost")
  } else if (
    activeCards.length > 1 &&
    !activeCards.some((c) => c.id === chosenNumber.value)
  ) {
    const combinationsSum = getCombinationSums(activeCards)
    if (!combinationsSum.some((sum) => sum === chosenNumber.value)) {
      showModel("lost")
    }
  }
}

const getCombinationSums = (activeCards) => {
  const selectionCombinations = getCombinations(activeCards, 2)
    .concat(getCombinations(activeCards, 3))
    .concat(getCombinations(activeCards, 4))

  return selectionCombinations.map((arr) =>
    arr.reduce((sum, id) => sum + id, 0)
  )
}

const getCombinations = (array, length) => {
  const result = []

  const combine = (start, combo) => {
    if (combo.length === length) {
      result.push(combo)
      return
    }
    for (let i = start; i < array.length; i++) {
      combine(i + 1, combo.concat(array[i].id))
    }
  }

  combine(0, [])
  return result
}

const showModel = (arg) => {
  gameOverModel.value = {
    show: true,
    message:
      arg === "win"
        ? "Congrats! You Won the GAME !!"
        : "GG! You Lost the GAME !!",
    img: arg === "win" ? "victory" : "lost",
  }
}

// Initialize the game
playGame()
</script>

<template>
  <div class="Index-page">
    <div class="game-over-modal" v-if="gameOverModel.show">
      <div class="modal-content">
        <h4>{{ gameOverModel.message }}</h4>
        <NuxtImg
          class="game-over-img"
          :src="`./images/${gameOverModel.img}.gif`"
          :alt="gameOverModel.message"
        />
        <div class="play-again-btn" @click="playGame">Play Again</div>
      </div>
    </div>

    <h1 class="game-name">Dice Game</h1>

    <div class="chosen-number-container">
      <p class="chosen-number">
        {{ chosenNumber === 0 ? "...." : chosenNumber }}
      </p>
    </div>

    <div class="dices-container">
      <NuxtImg
        v-for="(number, index) in chosenNumbers"
        :key="index"
        class="dice-img"
        :src="`./images/dices/dice${number === 0 ? 1 : number}.png`"
        :alt="`Dice Number ${number === 0 ? 1 : number}`"
      />
    </div>

    <div class="game-btn-container">
      <div
        :class="`game-btn ${!allowRollDice ? 'disable-cursor' : ''}`"
        @click="rollDice"
      >
        Roll the Dice
      </div>
    </div>

    <div class="cards-container">
      <div class="cards">
        <div
          v-for="card in cards"
          :key="card.id"
          :class="`card ${card.status} ${
            !allowSelectCard ? 'disable-cursor' : ''
          }`"
          @click="selectCard(card.id)"
        >
          {{ card.id }}
        </div>
      </div>
    </div>
  </div>
</template>

<style>
.Index-page {
  min-height: 100vh;
}

.game-name {
  font-size: 50px;
  text-align: center;
  padding: 16px 0;
}

.cards-container,
.chosen-number-container,
.dices-container,
.game-btn-container {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 16px 8px;
}

.cards {
  max-width: 350px;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 8px;
  flex-wrap: wrap;
}

.card {
  width: 48px;
  aspect-ratio: 1 / 1;
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 4px;
  font-size: 18px;
  font-weight: 700;
}

.active {
  background-color: #cb997e;
  cursor: pointer;
}

.inactive {
  background-color: #81b29a;
}

.chosen {
  background-color: #8a817c;
  cursor: pointer;
}

.disable-cursor {
  cursor: not-allowed !important;
}

.chosen-number,
.game-btn,
.play-again-btn {
  width: 200px;
  height: 24px;
  font-size: 18px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-color: #cb997e;
  border-radius: 4px;
}

.game-btn,
.play-again-btn {
  cursor: pointer;
}

.dices-container {
  gap: 8px;
}

.dice-img {
  display: block;
  width: 46px;
  aspect-ratio: 1 / 1;
}

.game-over-modal {
  position: fixed;
  inset: 0;
  margin: auto;
  width: 250px;
  aspect-ratio: 1 / 1;
  background: rgba(157, 78, 221, 0.25);
  box-shadow: 0 8px 32px rgba(31, 38, 135, 0.37);
  backdrop-filter: blur(2.5px);
  border-radius: 10px;
  border: 1px solid rgba(255, 255, 255, 0.18);
  display: flex;
  align-items: center;
  justify-content: center;
}

.game-over-img {
  display: block;
  width: 128px;
  aspect-ratio: 1 / 1;
  margin: 12px auto;
}
</style>
