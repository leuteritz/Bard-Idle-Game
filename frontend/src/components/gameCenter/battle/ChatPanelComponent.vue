<template>
  <div class="flex flex-col items-center justify-center w-full h-full">
    <div class="flex items-center gap-2 mb-2 text-lg font-bold text-black">
      <svg class="w-5 h-5 text-amber-500" fill="currentColor" viewBox="0 0 20 20">
        <path d="M2 5a2 2 0 012-2h12a2 2 0 012 2v10a2 2 0 01-2 2H6l-4 3V5z" />
      </svg>
      Chat
    </div>
    <div
      id="battle-chat-box"
      class="h-48 p-2 mb-2 space-y-1 overflow-y-auto text-sm border rounded w-80 opacity-70 bg-white/70 border-amber-100"
    >
      <div v-for="(msg, idx) in chatMessages" :key="'msg-' + idx" class="flex items-start gap-2">
        <span class="mr-2 text-xs text-gray-400">{{ msg.time }}</span>
        <span
          class="font-bold"
          :class="{
            'text-amber-500': msg.user === 'Bard',
            'text-blue-600': msg.team === 1 && msg.user !== 'Bard',
            'text-red-600': msg.team === 2 && msg.user !== 'Bard',
          }"
        >
          {{ msg.user }}:
        </span>
        <span class="break-words">{{ msg.text }}</span>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, PropType, watch, nextTick, ref, onMounted } from 'vue'
import { useGameStore } from '../../../stores/gameStore'
import { battleMessages } from '../../../config/messages'

export default defineComponent({
  name: 'ChatPanelComponent',
  props: {
    team1: {
      type: Array as PropType<any[]>,
      required: true,
    },
    team2: {
      type: Array as PropType<any[]>,
      required: true,
    },
    battleId: {
      type: [String, Number],
      default: 0,
    },
  },
  emits: ['gameTimeUpdate'],
  setup(props, { emit }) {
    const gameStore = useGameStore()
    const gameTime = ref(0)
    const chatMessages = ref<any[]>([])
    const currentTimeoutId = ref<any>(null)

    function showRandomChatMessagesSequentially() {
      // Alten Timeout abbrechen falls vorhanden
      if (currentTimeoutId.value) {
        clearTimeout(currentTimeoutId.value)
        currentTimeoutId.value = null
      }

      chatMessages.value = []

      if (!props.team1.length || !props.team2.length) {
        console.log('No team1 or team2')
        currentTimeoutId.value = setTimeout(() => showRandomChatMessagesSequentially(), 100)
        return
      }

      const messages = [...battleMessages]

      function showNext() {
        if (messages.length === 0) {
          currentTimeoutId.value = null // Reset when done
          return
        }

        const idx = Math.floor(Math.random() * messages.length)
        const msg = messages[idx]
        let chatMsg

        if (typeof msg === 'string') {
          const allChampions = [
            ...props.team1.map((champ) => ({ name: champ.name, team: 1 })),
            ...props.team2.map((champ) => ({ name: champ.name, team: 2 })),
          ]

          const randomChampion = allChampions[Math.floor(Math.random() * allChampions.length)]

          // Zeit zuerst erhöhen, damit Chat und MiniMap synchron sind
          gameTime.value += getRandomTimeIncrement()

          chatMsg = {
            user: randomChampion.name,
            text: msg,
            time: formatTime(gameTime.value),
            team: randomChampion.team,
          }
        }

        chatMessages.value.push(chatMsg)
        messages.splice(idx, 1)

        if (messages.length > 0) {
          // Timeout-ID speichern
          currentTimeoutId.value = setTimeout(showNext, gameStore.gameSpeed)
        } else {
          currentTimeoutId.value = null
        }
      }
      showNext()
    }

    function formatTime(seconds: number) {
      const min = Math.floor(seconds / 60)
      const sec = seconds % 60
      return `${min.toString().padStart(2, '0')}:${sec.toString().padStart(2, '0')}`
    }
    function getRandomTimeIncrement() {
      return Math.floor(Math.random() * 471) + 30
    }

    watch(
      () => chatMessages.value.length,
      async () => {
        await nextTick()
        const chatBox = document.getElementById('battle-chat-box')
        if (chatBox) {
          chatBox.scrollTop = chatBox.scrollHeight
        }
      },
    )

    watch(
      () => props.battleId,
      () => {
        console.log('battleId changed')
        gameTime.value = 120 // Reset game time
        showRandomChatMessagesSequentially()
      },
    )

    // Emit gameTime changes to parent component
    watch(
      () => gameTime.value,
      (newTime) => {
        // Emit the gameTime to parent component
        // This will be used by MiniMapComponent
        emit('gameTimeUpdate', newTime)
      },
    )

    return {
      chatMessages,
      showRandomChatMessagesSequentially,
      gameTime,
    }
  },
})
</script>
