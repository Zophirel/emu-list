<template>
  <div :class="theme">
    <button @click="toggleTheme" class="theme-toggle">
      Switch to {{ theme === 'light' ? 'Dark' : 'Light' }} Mode
    </button>

    <input type="text" v-model="searchQuery" class="search-bar" placeholder="Search lobby name..." @keyup="fetchLobbies" />

    <div class="games-dropdown">
      <button @click="dropdownOpen = !dropdownOpen" class="dropdown-toggle">
        {{ selectedGame || 'Filter by Game' }}
      </button>
      <div v-if="dropdownOpen" class="chips-container">
        <span v-for="game in uniqueGames" :key="game" class="game-chip" @click="filterByGame(game)">
          {{ game }}
        </span>
        <button v-if="selectedGame" @click="clearGameFilter" class="clear-filter">Clear Filter</button>
      </div>
    </div>

    <div class="lobby-list">
      <div v-if="filteredLobbies.length > 0" v-for="lobby in filteredLobbies" :key="lobby.id" class="lobby-tile">
        <div class="lobby-title">{{ lobby.name }}</div>
        <div class="lobby-description">
          <span v-if="lobby.hasPassword" class="lock-icon">🔒</span>
          {{ lobby.description }}
        </div>
        <div class="players-count">{{ lobby.players.length }} / {{ lobby.maxPlayers }} Players</div>
      </div>
      <p v-else class="no-lobbies">No lobbies found.</p>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';

const searchQuery = ref('');
const allLobbies = ref([]);
const loading = ref(false);
const error = ref(null);
const selectedGame = ref('');
const dropdownOpen = ref(false);
const theme = ref(localStorage.getItem('theme') || 'light'); // carica tema salvato

const filteredLobbies = computed(() => {
  if (!selectedGame.value) return allLobbies.value;
  return allLobbies.value.filter(lobby => lobby.players.some(player => player.gameName === selectedGame.value));
});

const uniqueGames = computed(() => {
  const games = new Set();
  allLobbies.value.forEach(lobby => {
    lobby.players.forEach(player => {
      if (player.gameName) games.add(player.gameName);
    });
  });
  return Array.from(games);
});

async function fetchLobbies() {
  loading.value = true;
  error.value = null;
  try {
    const response = await fetch(`${import.meta.env.VITE_BACKEND_URL}/api/lobbies?search=${searchQuery.value}`);
    const data = await response.json();
    allLobbies.value = data || [];
  } catch (err) {
    error.value = err;
    console.error('Error fetching lobbies:', err);
  } finally {
    loading.value = false;
  }
}

function filterByGame(game) {
  selectedGame.value = game;
  dropdownOpen.value = false;
}

function clearGameFilter() {
  selectedGame.value = '';
}

function toggleTheme() {
  theme.value = theme.value === 'light' ? 'dark' : 'light';
  localStorage.setItem('theme', theme.value); // salva tema selezionato
}

onMounted(() => {
  fetchLobbies();
});
</script>


<style scoped>
/* Common styles */
.search-bar {
  margin: 10px 0;
  padding: 8px;
  width: 100%;
}

.games-dropdown {
  position: relative;
  margin: 10px 0;
}

.dropdown-toggle {
  padding: 8px 12px;
  background-color: var(--btn-bg);
  border: none;
  border-radius: 5px;
  cursor: pointer;
  color: var(--text-color);
}

.chips-container {
  position: absolute;
  background: var(--dropdown-bg);
  border: 1px solid #ccc;
  padding: 10px;
  margin-top: 5px;
  border-radius: 5px;
  max-height: 200px;
  overflow-y: auto;
  z-index: 1;
}

.game-chip {
  display: inline-block;
  margin: 5px;
  padding: 5px 10px;
  background: var(--chip-bg);
  border-radius: 15px;
  cursor: pointer;
  color: var(--text-color);
}

.clear-filter {
  display: block;
  margin-top: 10px;
  background: #f88;
  border: none;
  padding: 5px 10px;
  border-radius: 5px;
  cursor: pointer;
}

.lobby-list {
  margin-top: 20px;
}

.lobby-tile {
  background: var(--tile-bg);
  border: 1px solid #ccc;
  border-radius: 10px;
  padding: 10px;
  margin-bottom: 10px;
}

.theme-toggle {
  margin-bottom: 10px;
  padding: 8px 12px;
  background-color: var(--btn-bg);
  border: none;
  border-radius: 5px;
  cursor: pointer;
  color: var(--text-color);
}

/* Light theme variables */
.light {
  --btn-bg: #ddd;
  --text-color: #000;
  --dropdown-bg: #fff;
  --chip-bg: #eee;
  --tile-bg: #f9f9f9;
  background-color: #fff;
  color: #000;
}

/* Dark theme variables */
.dark {
  --btn-bg: #333;
  --text-color: #fff;
  --dropdown-bg: #444;
  --chip-bg: #555;
  --tile-bg: #222;
  background-color: #111;
  color: #fff;
}
</style>
