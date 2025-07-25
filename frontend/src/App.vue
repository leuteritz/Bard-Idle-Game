<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'
import { useGameStore } from './stores/gameStore'
import { useBattleStore } from './stores/battleStore'
import LevelComponent from './components/LevelComponent.vue'
import { titleMessages } from './config/messages'
import StatsPanelComponent from './components/StatsPanelComponent.vue'
import AbilityBarComponent from './components/bottom/AbilityBarComponent.vue'
import BardHudComponent from './components/bottom/BardHudComponent.vue'
import GameCenterComponent from './components/gameCenter/GameCenterComponent.vue'

const gameStore = useGameStore()
const battleStore = useBattleStore()

const title = ref('Bardle')

// AutoBattle result state
const showAutoBattleResult = ref(false)
const autoBattleResult = ref(null)
const autoBattleMmrChange = ref(0)
const autoBattleLpChange = ref(0)
const autoBattleResultKey = ref(0)

const autoBattleResultComponentRef = ref(null)

const currentMsg = ref('')

function getRandomMsg() {
  const randomMsg = titleMessages[Math.floor(Math.random() * titleMessages.length)]
  currentMsg.value = randomMsg
}

function handleAutoBattleResult() {
  if (battleStore.lastAutoBattleResult && battleStore.showAutoBattleResult) {
    // Verwende die im Store gespeicherten alten Werte
    const oldMMR = battleStore.autoBattleOldMMR
    const oldLP = battleStore.autoBattleOldLP

    autoBattleMmrChange.value = gameStore.mmr - oldMMR
    autoBattleLpChange.value = gameStore.currentRank.lp - oldLP

    autoBattleResult.value = battleStore.lastAutoBattleResult
    showAutoBattleResult.value = true
    autoBattleResultKey.value++

    // Champions neu generieren
    if (autoBattleResultComponentRef.value && autoBattleResultComponentRef.value.refreshTeams) {
      autoBattleResultComponentRef.value.refreshTeams()
    }

    // Reset battleStore state
    battleStore.lastAutoBattleResult = null
    battleStore.showAutoBattleResult = false
  }
}

function closeAutoBattleResult() {
  showAutoBattleResult.value = false
  autoBattleResult.value = null
}

onMounted(() => {
  getRandomMsg()
  setInterval(() => {
    gameStore.gernerateMeeps()
  }, 1000)

  setInterval(() => {
    getRandomMsg()
  }, 5000)
})

// Watch for AutoBattle results
watch(
  () => battleStore.showAutoBattleResult,
  (newValue) => {
    if (newValue) {
      handleAutoBattleResult()
    }
  },
)
</script>

<template>
  <div
    class="flex flex-col justify-between p-4 w-full min-h-screen bg-gradient-to-br from-amber-100 via-yellow-200 to-orange-200 font-['MedievalSharp']"
  >
    <!-- Top -->
    <div class="grid w-full h-32 grid-cols-3 border-amber-400">
      <!-- Linke Spalte: LevelComponent -->
      <div class="flex items-center justify-start col-span-1">
        <LevelComponent />
      </div>

      <!-- Titel -->
      <h1
        class="flex items-center justify-center col-span-1 font-bold text-8xl text-amber-800 drop-shadow-lg"
      >
        {{ title }}
      </h1>

      <!-- Rechte Spalte: Message -->
      <div class="flex items-center justify-end col-span-1">
        <div
          class="flex items-center justify-center w-full h-full gap-2 rounded-xl border-amber-400/50"
        >
          <p
            class="text-xl cursor-default select-none text-amber-700 drop-shadow-lg hover:text-amber-800 hover:scale-105 animate-bounce"
          >
            {{ currentMsg }}
          </p>
        </div>
      </div>
    </div>

    <!-- Mitte -->
    <div class="flex justify-center w-full h-full">
      <div class="w-2/3">
        <GameCenterComponent />
      </div>
    </div>

    <!-- Bottom Panel -->
    <div class="flex justify-center w-full h-full">
      <div
        className="flex w-1/3 bg-gradient-to-br rounded-2xl shadow-2xl from-amber-600 via-amber-700 to-orange-700 border-2 border-amber-400 items-center"
      >
        <!-- Bard Hud -->
        <div className="w-64 flex items-center justify-center">
          <BardHudComponent />
        </div>

        <!-- Ability Bar -->
        <div className=" flex justify-center items-center">
          <AbilityBarComponent />
        </div>

        <!-- Stats Panel -->
        <div className="w-64 flex items-center justify-center">
          <StatsPanelComponent />
        </div>
      </div>
    </div>
  </div>
</template>
