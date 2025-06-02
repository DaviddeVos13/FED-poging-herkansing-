<script>

export default {
  data() {
    return {
      selectedPokemon: null,
      pokemonList: [],
      searchQuery: '',
      selectedSort: 'number-asc',
      selectedType: 'all'
    };
  },
  mounted() {
    fetch('https://pokeapi.co/api/v2/pokemon?limit=649&offset=0')
      .then(response => response.json())
      .then(data => {
        const fetches = data.results.map(pokemon =>
          fetch(pokemon.url).then(res => res.json())
        );

        Promise.all(fetches)
          .then(pokemonDataArray => {
            pokemonDataArray.sort((a, b) => a.id - b.id);
            this.pokemonList = pokemonDataArray;
          })
          .catch(error => {
            console.error('Fout bij ophalen van Pokemon data:', error);
          });
      })
      .catch(error => {
        console.error('Fout bij ophalen van lijst:', error);
      });
  },
  computed: {
    allTypes() {
      const types = new Set();
      this.pokemonList.forEach(pokemon => {
        pokemon.types.forEach(t => types.add(t.type.name));
      });
      return Array.from(types).sort();
    },
    filteredPokemon() {
      let filtered = this.pokemonList;

      if (this.searchQuery) {
        filtered = filtered.filter(p =>
          p.name.toLowerCase().includes(this.searchQuery.toLowerCase())
        );
      }

      if (this.selectedType !== 'all') {
        filtered = filtered.filter(p =>
          p.types.some(t => t.type.name === this.selectedType)
        );
      }

      switch (this.selectedSort) {
        case 'name-asc':
          filtered.sort((a, b) => a.name.localeCompare(b.name));
          break;
        case 'name-desc':
          filtered.sort((a, b) => b.name.localeCompare(a.name));
          break;
        case 'number-asc':
          filtered.sort((a, b) => a.id - b.id);
          break;
        case 'number-desc':
          filtered.sort((a, b) => b.id - a.id);
          break;
      }

      return filtered;
    }
  },
  methods: {
    selectPokemon(pokemon) {
      this.selectedPokemon = pokemon;
    },
    closeModal() {
      this.selectedPokemon = null;
    }
  },
  methods: {
    selectPokemon(pokemon) {
      this.selectedPokemon = pokemon;
    },
    closeModal() {
      this.selectedPokemon = null;
    },
    getStatColor(stat) {
      if (stat <= 49) return 'red';
      if (stat <= 99) return 'orange';
      return 'green';
    }
  }

};
</script>

<template>
  <div class="app-wrapper">

    <div class="logo-container">
      <img class="Logo" src="@/assets/Pokemon.png" alt="Pokemon Logo" />
    </div>

    <div class="controls">
      <input type="text" v-model="searchQuery" placeholder="Zoek een Pokémon..." />

      <select v-model="selectedSort">
        <option value="number-asc">Nummer oplopend</option>
        <option value="number-desc">Nummer aflopend</option>
        <option value="name-asc">Naam A-Z</option>
        <option value="name-desc">Naam Z-A</option>
      </select>

      <select v-model="selectedType">
        <option value="all">Alle types</option>
        <option v-for="type in allTypes" :key="type" :value="type">
          {{ type }}
        </option>
      </select>
    </div>

    <ul class="pokemon-grid">
      <li v-for="pokemon in filteredPokemon" :key="pokemon.id" class="pokemon-item" @click="selectPokemon(pokemon)">
        <img class="sprite" :src="pokemon.sprites.front_default" alt="Pokemon Sprite" />
        <p class="pokemon-number">#{{ pokemon.id }}</p>
        <h1>{{ pokemon.name }}</h1>
      </li>

    </ul>

  </div>
  <div v-if="selectedPokemon" class="modal-overlay" @click.self="closeModal">
    <div class="pokemon-modal">
      <button class="close-btn" @click="closeModal">X</button>

      <img :src="selectedPokemon.sprites.other['official-artwork'].front_default" alt="Big sprite" class="big-sprite" />

      <h2>{{ selectedPokemon.name }} <span>#{{ selectedPokemon.id.toString().padStart(3, '0') }}</span></h2>

      <div class="types">
        <span v-for="type in selectedPokemon.types" :key="type.type.name" :class="['type', 'type-' + type.type.name]">
          {{ type.type.name }}
        </span>

      </div>

      <div class="about-section">
        <h3>About</h3>
        <div class="about-stats">
          <div><strong>Weight:</strong> {{ (selectedPokemon.weight / 10).toFixed(1) }} kg</div>
          <div><strong>Height:</strong> {{ (selectedPokemon.height / 10).toFixed(1) }} m</div>
          <div><strong>Ability:</strong> {{ selectedPokemon.abilities[0].ability.name }}</div>
        </div>
      </div>

      <div class="stats-section">
        <h3>Base Stats</h3>
        <div v-for="stat in selectedPokemon.stats" :key="stat.stat.name" class="stat-bar">
          <div class="stat-name">{{ stat.stat.name.toUpperCase() }}</div>
          <div class="stat-value">{{ stat.base_stat }}</div>
          <div class="bar">
            <div class="bar-fill" :style="{
              width: (stat.base_stat / 200 * 100) + '%',
              backgroundColor: getStatColor(stat.base_stat)
            }"></div>
          </div>
        </div>
      </div>
    </div>
  </div>

  <li v-for="pokemon in filteredPokemonList" :key="pokemon.id" class="pokemon-item" @click="selectPokemon(pokemon)">
    <img class="sprite" :src="pokemon.sprites.front_default" alt="Pokemon Image" />
    <h1>{{ capitalizeName(pokemon.name) }}</h1>
    <p>#{{ pokemon.id }}</p>
  </li>



</template>



<style scoped>
.app-wrapper {
  position: absolute;
  left: 300px;
  top: -70px;
  padding: 20px;
}

h3 {
  color: blue;
  font-weight: bold;
}

h2 {
  color: blue;
  text-transform: capitalize;
}

.logo-container {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}

.Logo {
  width: 350px;
  height: auto;
}

.controls {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-bottom: 30px;
  flex-wrap: wrap;
}

.controls input,
.controls select {
  padding: 10px;
  font-size: 1rem;
  border-radius: 8px;
  border: 2px solid gray;
}

.pokemon-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
  list-style: none;
  padding: 0;
  margin: 0 auto;
  max-width: 900px;

}


.pokemon-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  background-color: rgb(23, 191, 247);
  padding: 15px;
  border-radius: 15px;
  border: 2px solid red;

}

.sprite {
  height: 150px;
  width: auto;
}

.pokemon-number {
  margin: 10px 0 0;
  font-weight: bold;
  color: lightgray;
  font-size: 1rem;
}

h1 {
  color: black;
  margin-top: 10px;
  font-size: 1rem;
  font-weight: bold;
  text-transform: capitalize;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 999;
}

.pokemon-modal {
  background: white;
  border-radius: 20px;
  padding: 30px;
  width: 600px;
  max-width: 95%;
  max-height: 90vh;
  overflow-y: auto;
  position: relative;
  text-align: center;
}

.close-btn {
  position: absolute;
  right: 20px;
  top: 20px;
  background: red;
  color: white;
  border: none;
  font-size: 1.2rem;
  border-radius: 50%;
  width: 30px;
  height: 30px;
  cursor: pointer;
}

.big-sprite {
  height: 200px;
  margin-bottom: 20px;
}

.types {
  margin: 10px 0;
}

.type-label {
  background: orange;
  color: white;
  border-radius: 20px;
  padding: 5px 10px;
  margin: 0 5px;
  display: inline-block;
  text-transform: capitalize;
}

.about-section,
.stats-section {
  text-align: left;
  margin-top: 20px;
}

.about-stats {
  display: flex;
  justify-content: space-between;
  padding: 10px 0;
  color: blue;
  text-transform: capitalize;
}

.stat-bar {
  color: blue;
  margin: 10px 0;
}

.stat-name {
  font-weight: bold;
  display: inline-block;
  width: 130px;
}

.stat-value {
  font-weight: bold;
  display: inline-block;
  width: 40px;
  padding-left: 25px;

}

.bar {
  background: #cecaca;
  height: 8px;
  border-radius: 10px;
  overflow: hidden;
  display: inline-block;
  width: 100%;
  vertical-align: middle;
  margin-left: 20px;

}

.bar-fill {
  height: 100%;
  background: orange;
  border-radius: 10px;
}

.type {
  padding: 6px 12px;
  border-radius: 20px;
  color: white;
  font-weight: bold;
  text-transform: capitalize;
  margin: 4px;
  display: inline-block;
  font-size: 0.9rem;
}

.type-fire {
  background-color: #f08030;
}

.type-water {
  background-color: #6890f0;
}

.type-grass {
  background-color: #78c850;
}

.type-electric {
  background-color: #f8d030;
  color: #333;
}

.type-ice {
  background-color: #98d8d8;
  color: #333;
}

.type-fighting {
  background-color: #c03028;
}

.type-poison {
  background-color: #a040a0;
}

.type-ground {
  background-color: #e0c068;
  color: #333;
}

.type-flying {
  background-color: #a890f0;
}

.type-psychic {
  background-color: #f85888;
}

.type-bug {
  background-color: #a8b820;
}

.type-rock {
  background-color: #b8a038;
}

.type-ghost {
  background-color: #705898;
}

.type-dragon {
  background-color: #7038f8;
}

.type-dark {
  background-color: #705848;
}

.type-steel {
  background-color: #b8b8d0;
  color: #333;
}

.type-fairy {
  background-color: #f0b6bc;
  color: #333;
}

.type-normal {
  background-color: #a8a878;
}

.pokemon-item {
  text-align: center;
  padding: 10px;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
  border-radius: 10px;
  cursor: pointer;
}

.pokemon-item:hover {
  transform: scale(1.05);
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
  background-color: #ffffff;
}
</style>
