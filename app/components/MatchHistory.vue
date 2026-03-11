<script setup lang="ts">
const props = defineProps<{
  matches: any[]
  selectedMatchId: string | null
  playerPuuid: string
}>()

const emit = defineEmits<{
  select: [matchId: string]
}>()

function getPlayerData(match: any) {
  const participant = match.info.participants.find(
    (p: any) => p.puuid === props.playerPuuid,
  )
  return participant
}

function formatDuration(seconds: number) {
  const mins = Math.floor(seconds / 60)
  const secs = seconds % 60
  return `${mins}:${secs.toString().padStart(2, '0')}`
}

function timeAgo(timestamp: number) {
  const diff = Date.now() - timestamp
  const hours = Math.floor(diff / 3600000)
  if (hours < 1) return 'Just now'
  if (hours < 24) return `${hours}h ago`
  const days = Math.floor(hours / 24)
  if (days < 7) return `${days}d ago`
  return `${Math.floor(days / 7)}w ago`
}
</script>

<template>
  <div class="flex h-full flex-col">
    <h2 class="mb-3 px-1 text-sm font-semibold uppercase tracking-wider text-zinc-400">
      Match History
    </h2>

    <div class="custom-scrollbar flex-1 space-y-2 overflow-y-auto pr-1">
      <button
        v-for="match in matches"
        :key="match.metadata.matchId"
        class="group w-full rounded-xl border p-3 text-left transition-all duration-200"
        :class="[
          selectedMatchId === match.metadata.matchId
            ? 'border-indigo-500/50 bg-indigo-500/10'
            : 'border-zinc-800 bg-zinc-900/40 hover:border-zinc-700 hover:bg-zinc-900/80',
        ]"
        @click="emit('select', match.metadata.matchId)"
      >
        <template v-if="getPlayerData(match)">
          <div class="flex items-center justify-between">
            <div class="flex items-center gap-2">
              <img
                :src="`https://ddragon.leagueoflegends.com/cdn/15.5.1/img/champion/${getPlayerData(match).championName}.png`"
                :alt="getPlayerData(match).championName"
                class="h-8 w-8 rounded-lg"
              />
              <div>
                <p class="text-sm font-medium text-zinc-100">
                  {{ getPlayerData(match).championName }}
                </p>
                <p class="text-xs text-zinc-500">
                  {{ getPlayerData(match).kills }}/{{ getPlayerData(match).deaths }}/{{ getPlayerData(match).assists }}
                </p>
              </div>
            </div>

            <div class="text-right">
              <span
                class="inline-block rounded-md px-2 py-0.5 text-xs font-semibold"
                :class="getPlayerData(match).win ? 'bg-emerald-500/20 text-emerald-400' : 'bg-red-500/20 text-red-400'"
              >
                {{ getPlayerData(match).win ? 'WIN' : 'LOSS' }}
              </span>
              <p class="mt-1 text-xs text-zinc-600">
                {{ formatDuration(match.info.gameDuration) }}
              </p>
            </div>
          </div>

          <div class="mt-2 flex items-center gap-3 text-xs text-zinc-500">
            <span>{{ getPlayerData(match).totalMinionsKilled + getPlayerData(match).neutralMinionsKilled }} CS</span>
            <span>{{ getPlayerData(match).visionScore }} Vision</span>
            <span class="ml-auto">{{ timeAgo(match.info.gameCreation) }}</span>
          </div>
        </template>
      </button>

      <div v-if="matches.length === 0" class="py-8 text-center text-sm text-zinc-600">
        No matches found
      </div>
    </div>
  </div>
</template>

<style scoped>
.custom-scrollbar::-webkit-scrollbar {
  width: 4px;
}
.custom-scrollbar::-webkit-scrollbar-track {
  background: transparent;
}
.custom-scrollbar::-webkit-scrollbar-thumb {
  background: rgb(63 63 70);
  border-radius: 999px;
}
</style>
