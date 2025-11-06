<template>
  <div class="app">
    <!-- 🎨 Pokémon decorativos de fondo -->
    <div class="background-pokemons">
      <img v-for="(p, i) in bgPokemons" :key="i" :src="p" class="bg-pokemon" />
    </div>

    <!-- HEADER -->
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

    <!-- MAIN -->
    <main v-if="pokemon" class="card-container">
      <div 
        class="pokemon-card"
        :style="{
          background: `linear-gradient(135deg, ${typeColors[pokemon.types[0]]} 40%, ${typeColors[pokemon.types[1]] || '#d8d8d8'})`
        }"
      >
        <h2>{{ pokemon.name.toUpperCase() }}</h2>
        <img :src="pokemon.image" :alt="pokemon.name" class="pokemon-img" />
        <p><strong>Altura:</strong> {{ pokemon.height / 10 }} m</p>
        <p><strong>Peso:</strong> {{ pokemon.weight / 10 }} kg</p>
      </div>

      <div class="pokemon-info">
        <h1>#{{ pokemon.id }}</h1>
        <h3>Tipos del Pokémon</h3>
        <div class="types">
          <span
            v-for="t in pokemon.types"
            :key="t"
            class="type"
            :style="{ backgroundColor: typeColors[t] }"
          >
            {{ t }}
          </span>
        </div>

        <h3>Debilidades</h3>
        <div class="weaknesses">
          <span
            v-for="w in weaknesses"
            :key="w"
            class="weak"
            :style="{ backgroundColor: typeColors[w] || '#999' }"
          >
            {{ w }}
          </span>
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

// 🎨 Colores oficiales
const typeColors = {
  normal: "#A8A77A",
  fire: "#EE8130",
  water: "#6390F0",
  electric: "#F7D02C",
  grass: "#7AC74C",
  ice: "#96D9D6",
  fighting: "#C22E28",
  poison: "#A33EA1",
  ground: "#E2BF65",
  flying: "#A98FF3",
  psychic: "#F95587",
  bug: "#A6B91A",
  rock: "#B6A136",
  ghost: "#735797",
  dragon: "#6F35FC",
  dark: "#705746",
  steel: "#B7B7CE",
  fairy: "#D685AD"
};

// 🌟 Imágenes de fondo
const bgPokemons = [
  "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/25.png", // Pikachu
  "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/133.png", // Eevee
  "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/1.png",   // Bulbasaur
  "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/4.png",   // Charmander
  "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/7.png",   // Squirtle
  "https://raw.githubusercontent.com/PokeAPI/sprites/master/sprites/pokemon/39.png"   // Jigglypuff
];

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

    // Debilidades
    const weaknessesSet = new Set();
    for (const type of data.types) {
      const typeData = await fetch(type.type.url).then((r) => r.json());
      typeData.damage_relations.double_damage_from.forEach((t) =>
        weaknessesSet.add(t.name)
      );
    }
    weaknesses.value = Array.from(weaknessesSet);
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
  overflow-y: scroll;
  scrollbar-width: none;
}

body::-webkit-scrollbar {
  display: none;
}

.app {
  text-align: center;
  padding: 30px 10px;
  position: relative;
  min-height: 100vh;
  overflow: hidden;
}

/* 🎨 FONDO DE POKÉMONS */
.background-pokemons {
  position: fixed;
  inset: 0;
  z-index: 0;
  overflow: hidden;
}

.bg-pokemon {
  position: absolute;
  width: 90px;
  opacity: 0.25;
  animation: floaty 10s ease-in-out infinite alternate;
}

.bg-pokemon:nth-child(1) { top: 10%; left: 5%; animation-delay: 0s; }
.bg-pokemon:nth-child(2) { bottom: 15%; right: 10%; animation-delay: 2s; }
.bg-pokemon:nth-child(3) { top: 20%; right: 20%; animation-delay: 4s; }
.bg-pokemon:nth-child(4) { bottom: 10%; left: 15%; animation-delay: 6s; }
.bg-pokemon:nth-child(5) { top: 50%; left: 40%; animation-delay: 8s; }
.bg-pokemon:nth-child(6) { bottom: 40%; right: 35%; animation-delay: 10s; }

@keyframes floaty {
  0% { transform: translateY(0px) rotate(0deg); }
  50% { transform: translateY(-20px) rotate(10deg); }
  100% { transform: translateY(0px) rotate(-10deg); }
}

/* HEADER */
.header {
  position: relative;
  z-index: 2;
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
  position: relative;
  z-index: 2;
}

.pokemon-card,
.pokemon-info {
  border-radius: 18px;
  padding: 20px;
  width: 280px;
  box-shadow: 0 5px 12px rgba(0, 0, 0, 0.15);
  color: #023047;
  background: #f8f9fa;
}

/* ✨ Imagen animada */
.pokemon-img {
  width: 180px;
  height: 180px;
  object-fit: contain;
  margin: 10px 0;
  animation: floatAndSpin 4s ease-in-out infinite;
}

@keyframes floatAndSpin {
  0%, 100% { transform: translateY(0) rotate(0deg); }
  25% { transform: translateY(-8px) rotate(4deg); }
  75% { transform: translateY(8px) rotate(-4deg); }
}

/* TYPES & WEAKNESSES */
.type, .weak {
  color: white;
  padding: 6px 12px;
  margin: 4px;
  border-radius: 10px;
  display: inline-block;
  font-weight: 600;
  text-transform: capitalize;
}

/* STATS */
.stat { text-align: left; margin-bottom: 10px; }

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
</style>
