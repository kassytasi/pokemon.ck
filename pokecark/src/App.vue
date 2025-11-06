<template>
  <div class="app">
    <header class="header">
      <img 
        src="https://upload.wikimedia.org/wikipedia/commons/9/98/International_Pok%C3%A9mon_logo.svg" 
        class="logo" 
        alt="Pokémon Logo" 
      />
      <div class="search-box">
        <input v-model="search" @keyup.enter="fetchPokemon" placeholder="Buscar Pokémon.." />
        <button @click="fetchPokemon">Buscar</button>
      </div>
    </header>

    <main v-if="pokemon" class="card-container">
      <div class="pokemon-card">
        <img :src="pokemon.image" :alt="pokemon.name" class="pokemon-img" />
        <h2>{{ pokemon.name.toUpperCase() }}</h2>
        <p><strong>Altura:</strong> {{ pokemon.height / 10 }} m</p>
        <p><strong>Peso:</strong> {{ pokemon.weight / 10 }} kg</p>
      </div>

      <div class="pokemon-info">
        <h1>#{{ pokemon.id }}</h1>
        <h3>Tipos del Pokémon</h3>
        <div class="types">
          <span v-for="t in pokemon.types" :key="t" class="type">{{ t }}</span>
        </div>

        <h3>Debilidades</h3>
        <div class="weaknesses">
          <span v-for="w in weaknesses" :key="w" class="weak">{{ w }}</span>
        </div>

        <h3>Estadísticas</h3>
        <div v-for="stat in pokemon.stats" :key="stat.name" class="stat">
          <p>{{ stat.name }}: {{ stat.value }}/255</p>
          <div class="progress-bar">
            <div class="progress" :style="{ width: stat.value + 'px' }"></div>
          </div>
        </div>
      </div>
    </main>
  </div>
</template>

<script setup>
import { ref } from "vue";

const search = ref("");
const pokemon = ref(null);
const weaknesses = ref([]);

async function fetchPokemon() {
  if (!search.value) return;

  try {
    const res = await fetch(`https://pokeapi.co/api/v2/pokemon/${search.value.toLowerCase()}`);
    if (!res.ok) throw new Error("No encontrado");
    const data = await res.json();

    const types = data.types.map((t) => t.type.name);
    pokemon.value = {
      id: data.id,
      name: data.name,
      height: data.height,
      weight: data.weight,
      image: data.sprites.other["official-artwork"].front_default,
      types,
      stats: data.stats.map((s) => ({
        name: s.stat.name,
        value: s.base_stat,
      })),
    };

    const typeData = await fetch(data.types[0].type.url).then((r) => r.json());
    weaknesses.value = typeData.damage_relations.double_damage_from.map((t) => t.name);
  } catch (e) {
    alert("Pokémon no encontrado ⚠️");
    pokemon.value = null;
  }
}
</script>

<style>
body {
  margin: 0;
  font-family: "Poppins", sans-serif;
  background: radial-gradient(circle at top, #a0c4ff, #cdb4db, #ffc8dd);
  min-height: 100vh;
}

.app {
  text-align: center;
  padding: 30px 10px;
}

/* HEADER */
.header {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 15px;
  background: rgba(255, 255, 255, 0.8);
  border-radius: 16px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  padding: 15px;
  margin-bottom: 25px;
}

.logo {
  width: 200px;
  height: auto;
  filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.2));
}

.search-box {
  display: flex;
  justify-content: center;
  gap: 10px;
}

.search-box input {
  padding: 10px 15px;
  border-radius: 10px;
  border: 2px solid #0077b6;
  font-size: 1rem;
  width: 220px;
  transition: 0.3s;
}

.search-box input:focus {
  border-color: #ffb703;
  outline: none;
}

.search-box button {
  background-color: #ffb703;
  color: #023047;
  border: none;
  padding: 10px 16px;
  border-radius: 10px;
  font-weight: bold;
  cursor: pointer;
  transition: 0.3s;
}

.search-box button:hover {
  background-color: #fb8500;
}

/* MAIN CONTENT */
.card-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: start;
  gap: 25px;
  max-width: 900px;
  margin: 0 auto;
}

.pokemon-card,
.pokemon-info {
  background: #e1c0c0b3;
  border-radius: 18px;
  padding: 20px;
  width: 280px;
  box-shadow: 0 5px 12px rgba(0, 0, 0, 0.15);
  transition: transform 0.2s ease-in-out;
}

.pokemon-card:hover,
.pokemon-info:hover {
  transform: scale(1.03);
}

.pokemon-img {
  width: 180px;
  height: 180px;
  object-fit: contain;
  margin-bottom: 10px;
}

.pokemon-card h2 {
  color: #023047;
  margin-bottom: 10px;
}

/* TYPES & WEAKNESSES */
.types .type {
  background: #06d6a0;
  color: white;
  padding: 6px 12px;
  margin: 4px;
  border-radius: 10px;
  display: inline-block;
  font-weight: 600;
}

.weaknesses .weak {
  background: #ef476f;
  color: white;
  padding: 6px 12px;
  margin: 4px;
  border-radius: 10px;
  display: inline-block;
  font-weight: 600;
}

/* STATS */
.stat {
  text-align: left;
  margin-bottom: 10px;
}

.progress-bar {
  background: #e9ecef;
  border-radius: 10px;
  height: 8px;
}

.progress {
  background: linear-gradient(90deg, #219ebc, #00b4d8);
  height: 8px;
  border-radius: 10px;
}

/* RESPONSIVE */
@media (max-width: 700px) {
  .card-container {
    flex-direction: column;
    align-items: center;
  }

  .pokemon-card,
  .pokemon-info {
    width: 90%;
  }

  .pokemon-img {
    width: 160px;
    height: 160px;
  }
}
</style>
