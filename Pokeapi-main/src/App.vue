<template>
  <div class="esos">
    <img class="pokeApi" src="https://assets.pokemon.com/assets/cms2-es-xl/img/misc/gus/buttons/logo-pokemon-79x45.png" style="width: 140px; ">
    <div style="padding-top: 2rem;">
      <input v-model="apk" placeholder="Ingrese su Pokémon" />
      <button @click="recoger" :style="{ background: fondoColor }">Buscar</button>
    </div>

    <div class="contenedortodo">
      
        <div style="align-self: center; justify-self: center; width: 89%;">
        <h3 style="font-size: 1.7rem; text-align: center;"># {{ pokemon }}</h3>

        <div v-if="tipos.length">
          <h3>Tipos del Pokémon</h3>
          <div class="tipos">
            <span v-for="t in tipos" :key="t.type.name" class="tipo" :style="{ backgroundColor: colorTipo(t.type.name) }">{{ t.type.name }}</span>
          </div>
        </div>
      
      
        <div v-if="debilidades.length">
          <h3>Debilidades</h3>
          <span v-for="d in debilidades" :key="d" class="debilidad" :style="{ backgroundColor: colorTipo(d) }">{{ d }}</span>
        </div>
      </div>
      
      

      <div class="parte1" :style="{ background: fondoColor }">
        <h4 style="font-size: 1.3rem; text-transform: uppercase;">{{ nombre }}</h4>
        <img :src="image" style="width: 21rem; height: 20rem; ">
        <div class="altura-peso">
  <p>Altura: {{ altura }} m</p>
  <p>Peso: {{ peso }} kg</p>
</div>

      </div>
     


      <div style="justify-self: center; width:70%;" >
        <h3>Estadísticas</h3>
        <div v-for="s in stats" :key="s.stat.name" class="stat">
          <p>{{ s.stat.name }}: {{ s.base_stat }}/255</p>
          <div class="barra">
            <div class="relleno" :style="{ width: (s.base_stat / 255 * 100) + '%', background: fondoColor }"></div>
          </div>
        </div>
      </div>
     

      <img class="pokeBola" src="./pokebola.png">
      <img class="pokeBola2" src="./logos.png" style="width: 100px; justify-self: end; margin-top: 2rem; ">
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from "vue";
import axios from "axios";

let pokemon = ref("");
let nombre = ref("");
let image = ref("");
let apk = ref("");
let stats = ref([]);
let tipos = ref([]);
let altura = ref("");
let peso = ref("");
let debilidades = ref([]);


const colores = {
  fire: "#F08030",
  water: "#6890F0",
  grass: "#78C850",
  electric: "#F8D030",
  ice: "#98D8D8",
  fighting: "#C03028",
  poison: "#A040A0",
  ground: "#E0C068",
  flying: "#A890F0",
  psychic: "#F85888",
  bug: "#A8B820",
  rock: "#B8A038",
  ghost: "#705898",
  dark: "#705848",
  dragon: "#7038F8",
  steel: "#B8B8D0",
  fairy: "#F0B6BC",
  normal: "#A8A878"
};


function colorTipo(nombreTipo) {
  return colores[nombreTipo] || "#ccc";
}



const fondoColor = computed(() => {
  if (tipos.value.length === 1) {
    let color = colores[tipos.value[0].type.name] || "#ccc";
    return color;
  } else if (tipos.value.length === 2) {
    let c1 = colores[tipos.value[0].type.name] || "#ccc";
    let c2 = colores[tipos.value[1].type.name] || "#eee";
    return `linear-gradient(135deg, ${c1}, ${c2})`;
  } else {
    return "#f5f5f5";
  }
});

async function recoger() {
  try {
    const res = await axios.get(`https://pokeapi.co/api/v2/pokemon/${apk.value.toLowerCase()}`);
    

    pokemon.value = res.data.id;
    nombre.value = res.data.name;
    image.value = res.data.sprites.other["official-artwork"].front_default;
    stats.value = res.data.stats;
    tipos.value = res.data.types;
    altura.value = (res.data.height / 10).toFixed(1);
    peso.value = (res.data.weight / 10).toFixed(0);

    let debiles = new Set();
    for (let t of tipos.value) {
      let infoTipo = await axios.get(t.type.url);
      let daño = infoTipo.data.damage_relations.double_damage_from;
      daño.forEach(d => debiles.add(d.name));
    }
    debilidades.value = Array.from(debiles);
    if (!apk.value.trim()) {
  const randomId = Math.floor(Math.random() * 898) + 1;
  apk.value = randomId.toString();
  
}
  setInterval(() => {
  if (!apk.value.trim()) {
    const randomId = Math.floor(Math.random() * 898) + 1;
    apk.value = randomId.toString();
    recoger();
  }
}, 8000);
  } catch (err) {
    console.error("Error al buscar el Pokémon", err);
  }
}


</script>

<style>
body {
  font-family: sans-serif;
  margin: 0;
 

}

input {
  padding: 8px;
  border-radius: 6px;
  border: 1px solid #aaa;
}

button {
  margin-left: 8px;
  padding: 8px 12px;
  border: none;
  border-radius: 6px;
  background-color: #ffcc00;
  cursor: pointer;
  transition: 0.3s;
}

button:hover {
  background-color: #ffdd33;
}

.contenedortodo {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  gap: 2.3rem;
  justify-items: center;
  align-items: center;
  text-align: center;
  padding: 1rem;
}

@media (max-width: 768px) {
  .contenedortodo {
    grid-template-columns: auto auto auto;
  }
}

.tipo, .debilidad {
  display: inline-block;
  background-color: rgba(255,255,255,0.5);
  border-radius: 10px;
  padding: 5px 10px;
  margin: 3px;
  text-transform: capitalize;
}

.barra {
  width: 100%;
  height: 10px;
  background-color: #eee;
  border-radius: 5px;
  overflow: hidden;
}

.relleno {
  height: 100%;
  transition: width 0.5s;
}

.parte1 {
  border-radius: 10px;
  padding: 1rem;
  transition: background 0.5s;
  align-self: center;
  margin-top: 2rem;
  
  
}

.pokeBola {
  width: 80px;
  opacity: 70%;
  justify-self: start;
}

.altura-peso {
  display: flex;
  justify-content: space-around;
  margin-top: 0.8rem;
  font-weight: bold;
}


</style>
