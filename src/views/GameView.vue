<script setup lang="ts">
import { useGameStore } from '../stores/useGameStore'
import Board from '@/components/Board.vue'
import { ref, onMounted } from 'vue'

const gameStore = useGameStore()
const boardRef = ref()
const interactable = ref(true)
const currentPlayerSymbol = ref<string | null>(null)
const gameResult = ref<string | null>(null)

const resetGame = (): void => {
  gameStore.initializeGame()
  boardRef.value?.resetStones()
  interactable.value = true
  currentPlayerSymbol.value = getPlayerSymbol(gameStore.currentPlayer)
  gameResult.value = null
}

const handleCellClicked = async (position: number): Promise<void> => {
  if (interactable.value === false) {
    return
  }

  if (boardRef.value?.isCellEmpty(position) === false) {
    console.log('該位置已被佔用，請選擇其他位置。')
    return
  }

  gameStore.placeStone(position)
  boardRef.value?.addStone(position)
  interactable.value = false
  await new Promise((resolve) => setTimeout(resolve, 200))

  boardRef.value?.moveStones('clockwise')
  await new Promise((resolve) => setTimeout(resolve, 200))

  currentPlayerSymbol.value = getPlayerSymbol(gameStore.currentPlayer)
  if (gameStore.winner !== null) {
    gameResult.value = getWinnerSymbol(gameStore.winner)
    interactable.value = false
  } else {
    interactable.value = true
  }
}

const getPlayerSymbol = (player: string): string => (player === 'X' ? '⚫' : '⚪')

const getWinnerSymbol = (winner: string): string => {
  switch (winner) {
    case 'X':
      return '⚫'
    case 'O':
      return '⚪'
    default:
      return '😐'
  }
}

onMounted(() => {
  resetGame()
})
</script>

<template>
  <div class="about">
    <div class="status">
      <div v-if="gameResult === null" class="current-player">
        當前玩家: {{ currentPlayerSymbol }}
      </div>
      <div v-else class="game-result">遊戲結果: {{ gameResult }}</div>
    </div>
    <Board ref="boardRef" @cellClicked="handleCellClicked" />
    <div class="buttons">
      <button class="reset-button" @click="resetGame">重置遊戲</button>
    </div>
  </div>
</template>

<style>
.about {
  min-height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
.status {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin-bottom: 20px;
}
.current-player,
.game-result {
  font-size: 20px;
  margin: 5px 0;
}
.game-result {
  font-weight: bold;
  color: #ff5722;
}
.buttons {
  display: flex;
  gap: 10px;
}
.move-button {
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
}
.reset-button {
  padding: 10px 20px;
  font-size: 16px;
  cursor: pointer;
  background-color: #f44336;
  color: white;
}
</style>
