<template>
  <div id="pokedex">
    <Card id="poke-card">
      <template #title>
        <div id="card-title">
          <span>{{displayName}}</span>
          <span>{{hitPoints}} HP</span>
        </div>
      </template>
      <template #subtitle>
        <div id="pokemon-type">{{pokemonType}}</div>
      </template>
      <template #content>
        <img v-bind:src="pokemon.picture"/>
          <div class="poke-info">
            <div class="poke-info-section" style="margin-right:25px">
              <span>Height: {{pokemon.height}}</span>
              <span>Weight: {{pokemon.weight}}</span>
            </div>
            <div class="poke-info-section">
              <span>Move: {{move}}</span>
              <span>Damage: {{attackDamage}}</span>
            </div>
          </div>
      </template>
      <template #footer>
        <div id="card-footer">
          <Button v-bind:disabled="!previous" @click="retrievePokedexListing(previous)">prev</Button>
          <span>{{pokemon.id}}/{{totalCount}}</span>
          <Button v-bind:disabled="!next" @click="retrievePokedexListing(next)">next</Button>
        </div>
      </template>
    </Card>
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue'
import * as axios from "axios"

export default defineComponent({
  name: 'App',
  data() {
    let pokemon = new Pokemon();
    let next: string = "https://pokeapi.co/api/v2/pokemon?limit=1&offset=0";
    let previous: string = "";
    return {
      pokemon,
      next,
      previous,
      totalCount: 0
    }
  },
  computed: {
    displayName(): string {
      return this.pokemon.name?.charAt(0).toUpperCase() + this.pokemon.name?.slice(1);
    },
    pokemonType(): string {
      return this.pokemon.types ? this.pokemon.types[0].type.name : "";
    },
    hitPoints(): number {
      return this.pokemon.stats?.find(x => x.stat.name === "hp").base_stat;
    },
    move(): string {
      return this.pokemon.moves ? this.pokemon.moves[0].move.name : "";
    },
    attackDamage(): number {
      return this.pokemon.stats?.find(x => x.stat.name === "attack").base_stat;
    }
  },
  methods: {
    async retrievePokemonListing(url: string) {
      return (await axios.default.get(url)).data;
    },
    async retrievePokemonDetails(pokemonListing: any) {
      return (await axios.default.get(pokemonListing.url)).data;
    },
    async retrievePokedexListing(listing: string) {
      var response = await this.retrievePokemonListing(listing);
      this.next = response.next;
      this.previous = response.previous;
      this.totalCount = response.count;

      for (let result of response.results) {
        var pokemonData = await this.retrievePokemonDetails(result);
        this.pokemon = {
          name: result.name,
          id: pokemonData.id,
          picture: pokemonData.sprites.front_default,
          weight: pokemonData.weight,
          height: pokemonData.height,
          types: pokemonData.types,
          moves: pokemonData.moves,
          stats: pokemonData.stats,
        };
      }
    }
  },
  async mounted() {
    this.retrievePokedexListing(this.next);
  }
})

export class Pokemon {
  name!: string;
  id!: number;
  picture!: string;
  weight!: number;
  types!: any[];
  moves!: any[];
  height!: number;
  stats!: any[];
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
#pokedex {
  display:flex; 
  flex-direction: column; 
  align-items: center;
}
#poke-card {
  width: 25rem; 
  margin-bottom: 2em
}
#card-title {
  display:flex;
  justify-content: space-between;
}
img {
  height: 15rem
}
#card-footer {
  display:flex;
  justify-content:space-between;
  align-items:center;
  margin-left:1rem;
  margin-right:1rem;
}
.poke-info {
  display: flex; 
  flex-direction: row;
  justify-content:center;
}
.poke-info-section {
  display:flex; 
  flex-direction: column; 
  align-items: flex-start;  
}
#pokemon-type {
  text-align: left;
  font-weight:bold;
}
</style>
