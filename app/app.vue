<script setup lang="ts">
const gameName = ref('')
const tagLine = ref('')
const account = ref<any | null>(null)
const errorMessage = ref('')
const loading = ref(false)

async function searchPlayer() {
  errorMessage.value = ''
  account.value = null
  loading.value = true

  try {
    const data = await $fetch('/api/account', {
      query: {
        gameName: gameName.value,
        tagLine: tagLine.value,
      },
    })

    account.value = data
  } catch (error: any) {
    errorMessage.value = error?.data?.message || error?.statusMessage || 'Search failed'
  } finally {
    loading.value = false
  }
}
</script>

<template>
  <div class="min-h-screen bg-zinc-950 text-zinc-100">
    <div class="mx-auto max-w-3xl px-6 py-12">
      <header class="mb-8">
        <p class="text-sm text-zinc-400">Nuxt 3 + Riot API Demo</p>
        <h1 class="mt-2 text-3xl font-bold">League Player Lookup</h1>
        <p class="mt-3 text-zinc-300">
          Search a League of Legends player by Riot ID.
        </p>
      </header>

      <div class="rounded-2xl border border-zinc-800 bg-zinc-900/60 p-5 shadow-lg">
        <div class="grid gap-4 md:grid-cols-2">
          <div>
            <label class="mb-2 block text-sm text-zinc-300">Game Name</label>
            <input
              v-model="gameName"
              type="text"
              placeholder="Faker"
              class="w-full rounded-xl border border-zinc-700 bg-zinc-950 px-4 py-3 outline-none ring-0 placeholder:text-zinc-500 focus:border-zinc-500"
            />
          </div>

          <div>
            <label class="mb-2 block text-sm text-zinc-300">Tag Line</label>
            <input
              v-model="tagLine"
              type="text"
              placeholder="KR1"
              class="w-full rounded-xl border border-zinc-700 bg-zinc-950 px-4 py-3 outline-none ring-0 placeholder:text-zinc-500 focus:border-zinc-500"
            />
          </div>
        </div>

        <button
          class="mt-4 rounded-xl bg-indigo-500 px-5 py-3 font-medium text-white transition hover:bg-indigo-400 disabled:cursor-not-allowed disabled:opacity-60"
          :disabled="loading"
          @click="searchPlayer"
        >
          {{ loading ? 'Searching...' : 'Search Player' }}
        </button>

        <p v-if="errorMessage" class="mt-4 text-sm text-red-400">
          {{ errorMessage }}
        </p>
      </div>

      <div
        v-if="account"
        class="mt-8 rounded-2xl border border-zinc-800 bg-zinc-900/60 p-5"
      >
        <h2 class="text-xl font-semibold">Account Result</h2>

        <div class="mt-4 grid gap-3 text-sm text-zinc-300">
          <div class="rounded-xl border border-zinc-800 bg-zinc-950/60 p-4">
            <span class="text-zinc-400">Game Name:</span>
            <span class="ml-2 font-medium text-zinc-100">{{ account.gameName }}</span>
          </div>

          <div class="rounded-xl border border-zinc-800 bg-zinc-950/60 p-4">
            <span class="text-zinc-400">Tag Line:</span>
            <span class="ml-2 font-medium text-zinc-100">#{{ account.tagLine }}</span>
          </div>

          <div class="rounded-xl border border-zinc-800 bg-zinc-950/60 p-4">
            <span class="text-zinc-400">PUUID:</span>
            <span class="ml-2 break-all font-medium text-zinc-100">{{ account.puuid }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>