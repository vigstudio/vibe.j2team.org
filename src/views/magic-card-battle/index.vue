<template>
  <div class="min-h-screen bg-[#0a0e1a] text-gray-100 overflow-hidden relative">
    <!-- BG Particles -->
    <div class="fixed inset-0 pointer-events-none z-0">
      <div
        v-for="n in 30"
        :key="n"
        class="absolute w-1 h-1 rounded-full bg-purple-500/20 animate-pulse"
        :style="{
          left: `${Math.random() * 100}%`,
          top: `${Math.random() * 100}%`,
          animationDelay: `${Math.random() * 3}s`,
        }"
      />
    </div>

    <!-- LOBBY SCREEN -->
    <div
      v-if="gameState.phase === 'lobby'"
      class="relative z-10 flex flex-col items-center justify-center min-h-screen px-4"
    >
      <div class="text-center animate-fade-up">
        <h1 class="font-display text-5xl md:text-7xl font-bold tracking-tight mb-2">
          <span class="text-purple-400">DEV</span>
          <span class="text-gray-100">_ARENA</span>
        </h1>
        <p class="text-purple-300/60 text-sm tracking-widest uppercase mb-8">
          ⚔️ Đấu Bài Ma Thuật Developer — 200+ Quân Bài
        </p>
      </div>

      <div class="flex flex-col gap-3 w-64 animate-fade-up" style="animation-delay: 0.2s">
        <button
          class="w-full py-3 px-6 bg-green-900/50 border border-green-500/50 text-green-300 font-display tracking-widest uppercase text-sm hover:bg-green-800 hover:border-green-400 transition-all cursor-pointer"
          @click="startGame('easy')"
        >
          ⭐ DỄ — Intern Bot
        </button>
        <button
          class="w-full py-3 px-6 bg-yellow-900/50 border border-yellow-500/50 text-yellow-300 font-display tracking-widest uppercase text-sm hover:bg-yellow-800 hover:border-yellow-400 transition-all cursor-pointer"
          @click="startGame('normal')"
        >
          ⭐⭐ THƯỜNG — Mid Dev
        </button>
        <button
          class="w-full py-3 px-6 bg-red-900/50 border border-red-500/50 text-red-300 font-display tracking-widest uppercase text-sm hover:bg-red-800 hover:border-red-400 transition-all cursor-pointer"
          @click="startGame('hard')"
        >
          🌟 KHÓ — Senior Troll
        </button>
      </div>

      <p class="text-gray-600 text-xs mt-8 max-w-sm text-center">
        200 quân bài • 6 loại bài • 8 môi trường • 9 kỹ năng đặc biệt
      </p>
    </div>

    <!-- LOADING -->
    <div v-else-if="isLoading" class="relative z-10 flex items-center justify-center min-h-screen">
      <div class="text-center">
        <Icon icon="lucide:loader-2" class="w-12 h-12 text-purple-400 animate-spin mx-auto mb-4" />
        <p class="text-purple-300 font-display tracking-widest uppercase text-sm">
          Đang xáo bài...
        </p>
      </div>
    </div>

    <!-- GAME BOARD -->
    <div v-else class="relative z-10 flex flex-col h-screen">
      <!-- Top: Opponent Info -->
      <div
        class="flex items-center justify-between px-4 py-2 bg-black/40 border-b border-red-900/30"
      >
        <div class="flex items-center gap-3">
          <div class="w-8 h-8 rounded-full bg-red-900 flex items-center justify-center text-sm">
            🤖
          </div>
          <div>
            <span class="text-xs text-red-400 font-display uppercase tracking-wider"
              >AI {{ difficultyLabel }}</span
            >
            <div class="flex items-center gap-1 mt-0.5">
              <div class="h-2 rounded-full bg-red-900/50 w-24 overflow-hidden">
                <div
                  class="h-full bg-red-500 transition-all duration-500"
                  :style="{ width: `${(gameState.opponent.hp / gameState.opponent.maxHp) * 100}%` }"
                />
              </div>
              <span class="text-xs text-red-300 min-w-[32px]"
                >{{ gameState.opponent.hp }}/{{ gameState.opponent.maxHp }}</span
              >
            </div>
          </div>
        </div>
        <div class="flex items-center gap-2">
          <span class="text-xs text-gray-500">🃏 {{ gameState.opponent.deck.length }}</span>
          <span class="text-xs text-gray-500">✋ {{ gameState.opponent.hand.length }}</span>
          <div class="flex gap-0.5">
            <div
              v-for="m in gameState.opponent.maxMana"
              :key="m"
              class="w-3 h-3 rounded-full border"
              :class="
                m <= gameState.opponent.mana
                  ? 'bg-blue-500 border-blue-400'
                  : 'bg-gray-800 border-gray-600'
              "
            />
          </div>
        </div>
      </div>

      <!-- Opponent Board -->
      <div class="flex-1 flex flex-col">
        <div class="flex justify-center items-center gap-2 py-3 min-h-[120px]">
          <div
            v-for="(slot, i) in gameState.opponent.board"
            :key="'opp-' + i"
            class="w-20 h-28 md:w-24 md:h-32 border border-gray-700/50 rounded-lg flex flex-col items-center justify-center text-xs relative transition-all"
            :class="[
              slot ? 'bg-red-950/40 border-red-700/50' : 'bg-gray-900/20 border-dashed',
              attackingIndex !== null && slot
                ? 'cursor-pointer hover:border-red-400 hover:shadow-[0_0_15px_rgba(239,68,68,0.3)]'
                : '',
            ]"
            @click="handleDefenderClick(i)"
          >
            <template v-if="slot">
              <span class="text-lg">{{
                slot.card.keywords.includes('stealth') ? '❓' : getCardEmoji(slot.card)
              }}</span>
              <span class="text-[10px] text-gray-400 truncate max-w-[70px] text-center">{{
                slot.card.keywords.includes('stealth') ? '???' : slot.card.name
              }}</span>
              <div class="flex gap-2 mt-1">
                <span class="text-yellow-400 text-[10px] font-bold">⚔️{{ slot.currentAtk }}</span>
                <span class="text-red-400 text-[10px] font-bold">❤️{{ slot.currentHp }}</span>
              </div>
              <div v-if="slot.keywords.length > 0" class="flex gap-0.5 mt-0.5">
                <span
                  v-for="kw in slot.keywords.slice(0, 3)"
                  :key="kw"
                  class="text-[8px]"
                  :title="kw"
                  >{{ keywordEmoji(kw) }}</span
                >
              </div>
              <span v-if="slot.frozen" class="absolute top-0 right-0 text-[10px]">🧊</span>
            </template>
            <template v-else>
              <span class="text-gray-700 text-lg">·</span>
            </template>
          </div>
        </div>

        <!-- Environment -->
        <div class="flex justify-center items-center py-1">
          <div
            v-if="gameState.environment.card"
            class="px-4 py-1.5 bg-gradient-to-r from-transparent via-emerald-900/30 to-transparent border-y border-emerald-500/20 text-center"
          >
            <span class="text-[10px] text-emerald-400 font-display tracking-wider uppercase">
              🌍 {{ gameState.environment.card.name }}: {{ gameState.environment.card.description }}
            </span>
          </div>
          <div
            v-else
            class="h-6 border-y border-gray-800/30 w-full flex justify-center items-center"
          >
            <span class="text-[10px] text-gray-700">— sân đấu —</span>
          </div>
        </div>

        <!-- Player Board -->
        <div class="flex justify-center items-center gap-2 py-3 min-h-[120px]">
          <div
            v-for="(slot, i) in gameState.player.board"
            :key="'plr-' + i"
            class="w-20 h-28 md:w-24 md:h-32 border rounded-lg flex flex-col items-center justify-center text-xs relative transition-all cursor-pointer"
            :class="[
              slot
                ? 'bg-blue-950/40 border-blue-700/50'
                : 'bg-gray-900/20 border-gray-700/50 border-dashed',
              attackingIndex === i
                ? 'ring-2 ring-yellow-400 shadow-[0_0_20px_rgba(234,179,8,0.3)]'
                : '',
              slot && isPlayerTurn && slot.canAttack && !slot.hasAttacked && !slot.frozen
                ? 'hover:border-yellow-400 hover:shadow-[0_0_10px_rgba(234,179,8,0.2)]'
                : '',
            ]"
            @click="handleBoardCreatureClick(i)"
          >
            <template v-if="slot">
              <span class="text-lg">{{ getCardEmoji(slot.card) }}</span>
              <span class="text-[10px] text-gray-300 truncate max-w-[70px] text-center">{{
                slot.card.name
              }}</span>
              <div class="flex gap-2 mt-1">
                <span class="text-yellow-400 text-[10px] font-bold">⚔️{{ slot.currentAtk }}</span>
                <span class="text-green-400 text-[10px] font-bold">❤️{{ slot.currentHp }}</span>
              </div>
              <div v-if="slot.keywords.length > 0" class="flex gap-0.5 mt-0.5">
                <span
                  v-for="kw in slot.keywords.slice(0, 3)"
                  :key="kw"
                  class="text-[8px]"
                  :title="kw"
                  >{{ keywordEmoji(kw) }}</span
                >
              </div>
              <span v-if="slot.frozen" class="absolute top-0 right-0 text-[10px]">🧊</span>
              <span
                v-if="isPlayerTurn && slot.canAttack && !slot.hasAttacked && !slot.frozen"
                class="absolute -top-1 -right-1 w-3 h-3 rounded-full bg-yellow-500 animate-pulse"
              />
            </template>
            <template v-else>
              <span
                v-if="selectedCardIndex !== null && canPlaySelectedToBoard"
                class="text-green-500 text-lg animate-pulse"
                >+</span
              >
              <span v-else class="text-gray-700 text-lg">·</span>
            </template>
          </div>
        </div>
      </div>

      <!-- Player Info -->
      <div
        class="flex items-center justify-between px-4 py-2 bg-black/40 border-t border-blue-900/30"
      >
        <div class="flex items-center gap-3">
          <div
            class="w-8 h-8 rounded-full bg-blue-900 flex items-center justify-center text-sm"
            :class="attackingIndex !== null ? 'cursor-pointer hover:ring-2 ring-yellow-400' : ''"
            @click="handleHeroAttack"
          >
            👤
          </div>
          <div>
            <span class="text-xs text-blue-400 font-display uppercase tracking-wider">BẠN</span>
            <div class="flex items-center gap-1 mt-0.5">
              <div class="h-2 rounded-full bg-blue-900/50 w-24 overflow-hidden">
                <div
                  class="h-full bg-blue-500 transition-all duration-500"
                  :style="{ width: `${(gameState.player.hp / gameState.player.maxHp) * 100}%` }"
                />
              </div>
              <span class="text-xs text-blue-300 min-w-[32px]"
                >{{ gameState.player.hp }}/{{ gameState.player.maxHp }}</span
              >
            </div>
          </div>
        </div>
        <div class="flex items-center gap-3">
          <div class="flex gap-0.5">
            <div
              v-for="m in gameState.player.maxMana"
              :key="m"
              class="w-3 h-3 rounded-full border transition-all"
              :class="
                m <= gameState.player.mana
                  ? 'bg-blue-500 border-blue-400'
                  : 'bg-gray-800 border-gray-600'
              "
            />
          </div>
          <span class="text-xs text-gray-500">🃏 {{ gameState.player.deck.length }}</span>
          <button
            v-if="isPlayerTurn && !isGameOver"
            class="px-3 py-1 bg-yellow-900/50 border border-yellow-500/50 text-yellow-300 text-xs font-display uppercase tracking-wider hover:bg-yellow-800 transition-all cursor-pointer"
            @click="endTurn"
          >
            Kết thúc lượt
          </button>
        </div>
      </div>

      <!-- Player Hand -->
      <div class="bg-black/60 border-t border-gray-800 px-2 py-2 overflow-x-auto">
        <div class="flex gap-1.5 justify-center min-w-max">
          <div
            v-for="(card, i) in gameState.player.hand"
            :key="'hand-' + i + '-' + card.id"
            class="w-20 md:w-24 p-1.5 border rounded-lg text-[10px] cursor-pointer transition-all flex flex-col shrink-0"
            :class="[
              selectedCardIndex === i
                ? 'ring-2 ring-purple-400 shadow-[0_0_15px_rgba(168,85,247,0.4)] -translate-y-2 bg-purple-950/60 border-purple-500/60'
                : card.cost <= gameState.player.mana && isPlayerTurn
                  ? 'bg-gray-900/60 border-gray-600/50 hover:border-purple-400 hover:-translate-y-1'
                  : 'bg-gray-900/30 border-gray-800 opacity-50',
            ]"
            @click="handleHandCardClick(i)"
          >
            <div class="flex justify-between items-center mb-0.5">
              <span class="bg-blue-900 text-blue-200 rounded px-1 font-bold text-[9px]">{{
                card.cost
              }}</span>
              <span class="text-[9px]">{{ cardTypeEmoji(card.type) }}</span>
            </div>
            <span
              class="text-[9px] text-gray-200 font-semibold leading-tight mb-0.5 line-clamp-2"
              >{{ card.name }}</span
            >
            <span class="text-[8px] text-gray-500 leading-tight line-clamp-2 flex-1">{{
              card.description
            }}</span>
            <div v-if="card.type === 'creature'" class="flex gap-2 mt-0.5">
              <span class="text-yellow-400 text-[9px]">⚔️{{ card.atk }}</span>
              <span class="text-red-400 text-[9px]">❤️{{ card.hp }}</span>
            </div>
            <div class="flex justify-end mt-0.5">
              <span class="text-[8px]" :class="rarityColor(card.rarity)">{{
                rarityLabel(card.rarity)
              }}</span>
            </div>
          </div>
        </div>
      </div>

      <!-- Turn Indicator -->
      <div
        v-if="!isPlayerTurn && !isGameOver"
        class="absolute inset-0 bg-black/30 flex items-center justify-center z-30 pointer-events-none"
      >
        <div
          class="bg-red-900/80 px-6 py-3 border border-red-500/50 text-red-200 font-display tracking-widest uppercase text-sm animate-pulse"
        >
          🤖 AI đang suy nghĩ...
        </div>
      </div>

      <!-- Game Over Modal -->
      <div
        v-if="isGameOver"
        class="absolute inset-0 bg-black/70 flex items-center justify-center z-40"
      >
        <div class="text-center p-8 border border-gray-600 bg-gray-900/95 max-w-sm">
          <h2
            class="font-display text-3xl font-bold mb-2"
            :class="gameState.winner === 'player' ? 'text-yellow-400' : 'text-red-400'"
          >
            {{ gameState.winner === 'player' ? '🏆 CHIẾN THẮNG!' : '💀 THẤT BẠI!' }}
          </h2>
          <p class="text-gray-400 text-sm mb-4">
            {{
              gameState.winner === 'player' ? 'Bạn đã nghiền nát con Bot!' : 'Bot đã xử đẹp bạn!'
            }}
          </p>
          <p class="text-gray-500 text-xs mb-6">
            Lượt: {{ gameState.turnNumber }} • HP còn:
            {{ gameState.winner === 'player' ? gameState.player.hp : gameState.opponent.hp }}
          </p>
          <div class="flex gap-3 justify-center">
            <button
              class="px-4 py-2 bg-purple-900/50 border border-purple-500/50 text-purple-300 font-display uppercase text-xs tracking-wider hover:bg-purple-800 transition-all cursor-pointer"
              @click="startGame(difficulty)"
            >
              Chơi lại
            </button>
            <button
              class="px-4 py-2 bg-gray-800 border border-gray-600 text-gray-300 font-display uppercase text-xs tracking-wider hover:bg-gray-700 transition-all cursor-pointer"
              @click="gameState.phase = 'lobby'"
            >
              Về Lobby
            </button>
          </div>
        </div>
      </div>

      <!-- Game Log (collapsible) -->
      <div
        v-if="showLog"
        class="absolute bottom-[140px] right-2 w-72 max-h-48 overflow-y-auto bg-black/90 border border-gray-700 rounded-lg p-2 z-20 text-[10px]"
      >
        <div
          v-for="(log, i) in reversedLog"
          :key="i"
          class="py-0.5 border-b border-gray-800/50 last:border-0"
        >
          <span :class="log.side === 'player' ? 'text-blue-400' : 'text-red-400'">{{
            log.message
          }}</span>
        </div>
      </div>
      <button
        class="absolute bottom-[140px] right-2 z-20 w-6 h-6 bg-gray-800 border border-gray-600 rounded text-[10px] text-gray-400 hover:text-white cursor-pointer"
        :class="showLog ? 'hidden' : ''"
        @click="showLog = !showLog"
        title="Game Log"
      >
        📜
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import { Icon } from '@iconify/vue'
import type { CardType, Rarity, Keyword, CardData } from './types'
import { useGameEngine } from './composables/useGameEngine'

const {
  gameState,
  isLoading,
  isPlayerTurn,
  isGameOver,
  selectedCardIndex,
  attackingIndex,
  difficulty,
  startGame,
  playCard,
  attackWithCreature,
  endTurn,
} = useGameEngine()

const showLog = ref(false)

const difficultyLabel = computed(() => {
  switch (difficulty.value) {
    case 'easy':
      return 'Intern'
    case 'normal':
      return 'Mid Dev'
    case 'hard':
      return 'Senior'
    default:
      return ''
  }
})

const reversedLog = computed(() => [...gameState.value.log].reverse())

const canPlaySelectedToBoard = computed(() => {
  if (selectedCardIndex.value === null) return false
  const card = gameState.value.player.hand[selectedCardIndex.value]
  return card?.type === 'creature' || card?.type === 'weapon'
})

function handleHandCardClick(index: number) {
  if (!isPlayerTurn.value || isGameOver.value) return
  const card = gameState.value.player.hand[index]
  if (!card || card.cost > gameState.value.player.mana) return

  // Cancel attack mode
  attackingIndex.value = null

  if (selectedCardIndex.value === index) {
    // Deselect
    selectedCardIndex.value = null
    return
  }

  selectedCardIndex.value = index

  // Auto-play non-targeted cards
  if (card.type === 'spell' && !needsTarget(card)) {
    playCard(index)
    selectedCardIndex.value = null
  } else if (card.type === 'support' && !needsCreatureTarget(card)) {
    playCard(index)
    selectedCardIndex.value = null
  } else if (card.type === 'trap') {
    playCard(index)
    selectedCardIndex.value = null
  } else if (card.type === 'environment') {
    playCard(index)
    selectedCardIndex.value = null
  }
}

function handleBoardCreatureClick(boardIndex: number) {
  if (!isPlayerTurn.value || isGameOver.value) return

  const slot = gameState.value.player.board[boardIndex]

  // If we have a card selected, play it to this board slot
  if (selectedCardIndex.value !== null) {
    const card = gameState.value.player.hand[selectedCardIndex.value]
    if (card) {
      if (card.type === 'creature' && !slot) {
        playCard(selectedCardIndex.value, boardIndex)
        selectedCardIndex.value = null
        return
      }
      if ((card.type === 'weapon' || card.type === 'support') && slot) {
        playCard(selectedCardIndex.value, boardIndex)
        selectedCardIndex.value = null
        return
      }
      // Spell targeting friendly creature
      if (card.type === 'spell' && slot) {
        playCard(selectedCardIndex.value, boardIndex)
        selectedCardIndex.value = null
        return
      }
    }
  }

  // Select creature for attacking
  if (slot && slot.canAttack && !slot.hasAttacked && !slot.frozen) {
    if (attackingIndex.value === boardIndex) {
      attackingIndex.value = null
    } else {
      attackingIndex.value = boardIndex
      selectedCardIndex.value = null
    }
  }
}

function handleDefenderClick(defenderIdx: number) {
  if (attackingIndex.value === null) {
    // Check if we have a spell selected targeting enemy
    if (selectedCardIndex.value !== null) {
      const card = gameState.value.player.hand[selectedCardIndex.value]
      if (card && card.type === 'spell') {
        playCard(selectedCardIndex.value, defenderIdx)
        selectedCardIndex.value = null
      }
    }
    return
  }
  attackWithCreature(attackingIndex.value, defenderIdx)
  attackingIndex.value = null
}

function handleHeroAttack() {
  if (attackingIndex.value === null) return
  attackWithCreature(attackingIndex.value, 'hero')
  attackingIndex.value = null
}

function needsTarget(card: CardData): boolean {
  return card.effects.some((e) =>
    ['enemy_creature', 'friendly_creature', 'any_creature'].includes(e.target),
  )
}

function needsCreatureTarget(card: CardData): boolean {
  return card.effects.some((e) => e.target === 'friendly_creature')
}

function cardTypeEmoji(type: CardType): string {
  const map: Record<CardType, string> = {
    creature: '⚔️',
    spell: '✨',
    trap: '🪤',
    support: '💊',
    weapon: '🗡️',
    environment: '🌍',
  }
  return map[type]
}

function rarityColor(rarity: Rarity): string {
  const map: Record<Rarity, string> = {
    common: 'text-gray-500',
    rare: 'text-blue-400',
    epic: 'text-purple-400',
    legendary: 'text-yellow-400',
  }
  return map[rarity]
}

function rarityLabel(rarity: Rarity): string {
  const map: Record<Rarity, string> = {
    common: '⭐',
    rare: '⭐⭐',
    epic: '⭐⭐⭐',
    legendary: '🌟',
  }
  return map[rarity]
}

function keywordEmoji(kw: Keyword): string {
  const map: Record<Keyword, string> = {
    rush: '⚡',
    shield: '🛡️',
    stealth: '👻',
    taunt: '🏰',
    lifesteal: '🧛',
    poison: '☠️',
    deathrattle: '💀',
    battlecry: '📯',
    overflow: '💥',
  }
  return map[kw]
}

function getCardEmoji(card: CardData): string {
  if (card.name.includes('Docker')) return '🐳'
  if (card.name.includes('React')) return '⚛️'
  if (card.name.includes('Bug')) return '🐛'
  if (card.name.includes('Coffee') || card.name.includes('Cà phê')) return '☕'
  if (card.name.includes('AI') || card.name.includes('Bot') || card.name.includes('ChatGPT'))
    return '🤖'
  if (card.name.includes('Intern') || card.name.includes('Junior')) return '👶'
  if (card.name.includes('Senior') || card.name.includes('Lead')) return '🧙'
  if (card.name.includes('CEO') || card.name.includes('Manager')) return '👔'
  if (card.name.includes('QA') || card.name.includes('Test')) return '🔍'
  if (card.name.includes('Null') || card.name.includes('Error')) return '💥'
  if (card.name.includes('Memory')) return '🧠'
  if (card.name.includes('Hack')) return '👾'
  if (card.name.includes('Database') || card.name.includes('SQL')) return '🗄️'
  if (card.name.includes('Git') || card.name.includes('GitHub')) return '🐙'
  if (card.name.includes('Kubernetes') || card.name.includes('K8s')) return '☸️'
  if (card.name.includes('Stack Overflow')) return '📚'
  if (card.name.includes('Fire') || card.name.includes('Production')) return '🔥'
  if (card.name.includes('Shield') || card.name.includes('Firewall')) return '🛡️'
  if (card.name.includes('Token') || card.name.includes('Container')) return '📦'
  if (card.rarity === 'legendary') return '🌟'
  if (card.type === 'creature') return '👤'
  return '🃏'
}
</script>

<style scoped>
.line-clamp-2 {
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
</style>
