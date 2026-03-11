<script setup lang="ts">
const gameName = ref('')
const tagLine = ref('')
const account = ref<any | null>(null)
const matches = ref<any[]>([])
const matchDetails = ref<any[]>([])
const selectedMatchId = ref<string | null>(null)
const errorMessage = ref('')
const loading = ref(false)
const loadingMatches = ref(false)

async function searchPlayer() {
  errorMessage.value = ''
  account.value = null
  matches.value = []
  matchDetails.value = []
  selectedMatchId.value = null
  loading.value = true

  try {
    // 1. Get account
    const accountData = await $fetch<any>('/api/account', {
      query: {
        gameName: gameName.value,
        tagLine: tagLine.value,
      },
    })
    account.value = accountData

    // 2. Get match IDs
    loadingMatches.value = true
    const matchIds = await $fetch<string[]>('/api/matches', {
      query: { puuid: accountData.puuid, count: 15 },
    })
    matches.value = matchIds

    // 3. Fetch match details (parallel, batched)
    const details: any[] = []
    const batchSize = 5
    for (let i = 0; i < matchIds.length; i += batchSize) {
      const batch = matchIds.slice(i, i + batchSize)
      const results = await Promise.all(
        batch.map(id =>
          $fetch(`/api/match/${id}`).catch(() => null),
        ),
      )
      details.push(...results.filter(Boolean))
      matchDetails.value = [...details]
    }
  } catch (error: any) {
    errorMessage.value = error?.data?.message || error?.statusMessage || 'Search failed'
  } finally {
    loading.value = false
    loadingMatches.value = false
  }
}

function selectMatch(matchId: string) {
  selectedMatchId.value = selectedMatchId.value === matchId ? null : matchId
}
</script>

<template>
  <div class="flex h-screen flex-col overflow-hidden bg-zinc-950 text-zinc-100">
    <!-- Top Bar -->
    <header class="flex shrink-0 items-center gap-4 border-b border-zinc-800/60 px-6 py-4">
      <div>
        <h1 class="text-lg font-bold tracking-tight">LoL Profile Graph</h1>
        <p class="text-xs text-zinc-500">Obsidian-style champion visualization</p>
      </div>

      <div class="ml-auto flex items-center gap-3">
        <input
          v-model="gameName"
          type="text"
          placeholder="Game Name"
          class="w-40 rounded-lg border border-zinc-800 bg-zinc-900 px-3 py-2 text-sm outline-none placeholder:text-zinc-600 focus:border-zinc-600"
          @keydown.enter="searchPlayer"
        />
        <input
          v-model="tagLine"
          type="text"
          placeholder="Tag"
          class="w-24 rounded-lg border border-zinc-800 bg-zinc-900 px-3 py-2 text-sm outline-none placeholder:text-zinc-600 focus:border-zinc-600"
          @keydown.enter="searchPlayer"
        />
        <button
          class="rounded-lg bg-indigo-600 px-4 py-2 text-sm font-medium transition hover:bg-indigo-500 disabled:opacity-50"
          :disabled="loading || !gameName || !tagLine"
          @click="searchPlayer"
        >
          {{ loading ? 'Loading...' : 'Search' }}
        </button>
      </div>
    </header>

    <p v-if="errorMessage" class="shrink-0 bg-red-500/10 px-6 py-2 text-sm text-red-400">
      {{ errorMessage }}
    </p>

    <!-- Loading indicator for matches -->
    <div v-if="loadingMatches" class="shrink-0 border-b border-zinc-800/60 px-6 py-2">
      <div class="flex items-center gap-2 text-sm text-zinc-400">
        <svg class="h-4 w-4 animate-spin" viewBox="0 0 24 24" fill="none">
          <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4" />
          <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4z" />
        </svg>
        Loading matches... ({{ matchDetails.length }}/{{ matches.length }})
      </div>
    </div>

    <!-- Main Content -->
    <div class="flex min-h-0 flex-1">
      <!-- Left: Match History -->
      <aside
        v-if="account"
        class="w-72 shrink-0 border-r border-zinc-800/60 p-4"
      >
        <div class="mb-4 rounded-xl border border-zinc-800 bg-zinc-900/40 p-3">
          <p class="text-sm font-semibold text-zinc-100">
            {{ account.gameName }}<span class="text-zinc-500">#{{ account.tagLine }}</span>
          </p>
        </div>

        <MatchHistory
          :matches="matchDetails"
          :selected-match-id="selectedMatchId"
          :player-puuid="account.puuid"
          @select="selectMatch"
        />
      </aside>

      <!-- Right: Champion Graph -->
      <main class="relative min-h-0 flex-1">
        <ChampionGraph
          v-if="account"
          :matches="matchDetails"
          :player-puuid="account.puuid"
          :player-name="account.gameName"
          :selected-match-id="selectedMatchId"
        />

        <!-- Empty state -->
        <div v-else class="flex h-full items-center justify-center">
          <div class="text-center">
            <div class="mx-auto mb-4 h-16 w-16 rounded-full bg-zinc-900 p-4 text-zinc-700">
              <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
                <circle cx="11" cy="11" r="8" />
                <path d="m21 21-4.35-4.35" />
              </svg>
            </div>
            <p class="text-sm text-zinc-500">Search a player to explore their champion graph</p>
          </div>
        </div>
      </main>
    </div>
  </div>
</template>
