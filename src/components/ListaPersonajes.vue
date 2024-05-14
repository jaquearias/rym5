
<template>
  <div class="hello">
      <h1 class="titulo">Characters List</h1>
      {{this.buscaNombre}}
      <div class="fondo">
        <input id="buscar" type="search" placeholder="Search character by name..." autocomplete="off" v-model="buscaNombre">
        <b-button class="botonRosa" @click="buscarNombre">Search</b-button>
      </div>
    <div class="row">
      <!-- Filtros -->
      <div class="col fondo"> 
        <h1 class="subtitulo">Filter</h1>
        <div class="count">
          Results: {{ this.rows }} 
        </div>
        <div class="filtro">
          <h2 class="subFiltro">Gender</h2>
          <span v-for="option in genderOptions" :key="option.id">
            <label>
              <input type="checkbox" :value="option.value" v-model="filtrosGender" >
              {{ option.label }}
            </label><br>
          </span>
        </div>
        <div class="filtro">
          <h2 class="subFiltro">Status</h2>
          <span v-for="option in statusOptions" :key="option.id">
            <label>
              <input type="checkbox" :value="option.value" v-model="filtrosStatus" >
              {{ option.label }}
            </label><br>
          </span>
        </div>
        <div class="filtro">
          <h2 class="subFiltro">Species</h2>
          <span v-for="option in speciesOptions" :key="option.id">
            <label>
              <input type="checkbox" :value="option.value" v-model="filtrosSpecies" >
              {{ option.label }}
            </label><br>
          </span>
        </div>
        <b-button class="botonRosa" @click="filtrarConsulta">Filter</b-button>
      </div>
      <!-- Tabla -->
      <div class="col-10">
        <div>
          <b-table striped hover bordered
            :fields="columnas" :items="datos">
            <template #cell(imagenPersonaje)="data">
              <BImg :src="`${data.item.image}`" rounded="circle" />
            </template>
            <template #cell(status)="data">
              {{
                `${data.item.status != "unknown" ? data.item.status : "Unknown"}`
              }}
            </template>
            <template #cell(species)="data">
              {{
                `${data.item.species != "unknown" ? data.item.species : "Unknown"}`
              }}
            </template>
            <template #cell(gender)="data">
              {{
                `${data.item.gender != "unknown" ? data.item.gender : "Unknown"}`
              }}
            </template>
            <template #cell(id)="data">
              <BButton class="botonRosa" @click="send(data.item.id)"
                >Info</BButton
              >
            </template>
          </b-table>
          <div>
            <b-pagination v-model="currentPage" pills size="lg" align="center" :total-rows="rows"></b-pagination>
          </div>
        </div>
      </div> 
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "ListaPersonajes",
  data() {
    return {
      datos: null,
      currentPage: 1,
      rows: null,
      buscaNombre: null,
      filtrosGender: [],
      filtrosStatus: [],
      filtrosSpecies: [],
      cadenaFiltros: " ",
      copyCadenaFiltros: " ",
      query: null,
      columnas: [
        {
          label: "Character",
          key: "imagenPersonaje"
        },
        {
          label: "Name",
          key: "name"
        },
        {
          label: "Species",
          key: "species"
        },
        {
          label: "Gender",
          key: "gender"
        },
        {
          label: "Status",
          key: "status"
        },
        {
          label: "See more",
          key: "id"
        },
      ],
      genderOptions: [
        { id: 1, value: 'female', label: 'Female' },
        { id: 2, value: 'male', label: 'Male' },
        { id: 3, value: 'genderless', label: 'Genderless' },
        { id: 4, value: 'unknown', label: 'Unknown' }
      ],
      statusOptions: [
        { id: 1, value: 'alive', label: 'Alive' },
        { id: 2, value: 'dead', label: 'Dead' },
        { id: 3, value: 'unknown', label: 'Unknown' }
      ],
      speciesOptions: [
        { id: 1, value: 'human', label: 'Human' },
        { id: 2, value: 'Alien', label: 'Alien' },
        { id: 3, value: 'Animal', label: 'Animal' },
        { id: 4, value: 'Humanoid', label: 'Humanoid' },
        { id: 5, value: 'Poopybutthole', label: 'Poopybutthole' },
        { id: 6, value: 'Robot', label: 'Robot' },
        { id: 7, value: 'Cronenberg', label: 'Cronenberg' },
        { id: 8, value: 'Disease', label: 'Disease' },
        { id: 9, value: 'Mythological Creature', label: 'Mythological Creature' },
        { id: 10, value: 'unknown', label: 'Unknown' }
      ],
      loading: true,
      errored: false,
    };
  },
  mounted() { //se activará después de que se encuentre montado
    this.armaQuery();
  },
  watch: {
    currentPage: function () {
      this.armaQueryPagina();
      this.scrollToTop();
    }
  },
  methods: {
    send(id) {
      this.$router.push("/listaDetalles/" + id);
    },
    armaQuery() {
      var queryVar = `
          query Characters {
              characters(
                ${this.cadenaFiltros}
                page: ${this.currentPage}
              ) {
                  results {
                      id
                      image
                      name
                      species
                      gender
                      status
                  }
                  info {
                      count
                  }
              }
          }
        `;
      console.log(queryVar);
      this.query = queryVar;
      this.getInfo();
    },
    armaQueryPagina() {
      var queryVar = `
          query Characters {
              characters(
                ${this.copyCadenaFiltros}
                page: ${this.currentPage}
              ) {
                  results {
                      id
                      image
                      name
                      species
                      gender
                      status
                  }
                  info {
                      count
                  }
              }
          }
        `;
      console.log(queryVar);
      this.query = queryVar;
      this.getInfo();
    },
    getInfo() {
      var baseURL = "https://rickandmortyapi.com/graphql";
      axios
        .post(baseURL, {
          query: this.query,
        })
        .then((response) => {
          this.datos = response.data.data.characters.results;
          this.rows = response.data.data.characters.info.count;
          console.log(response.data.data.characters.info.count)
        })
        .catch((error) => {
          console.log(error);
          this.errored = true;
        })
        .finally(() => (this.loading = false));
    },
    scrollToTop() { // Función para desplazar al inicio de la página
      window.scrollTo({
        top: 0,
        behavior: 'smooth'
      });
    },
    filtrarConsulta(){
      this.cadenaFiltros = " ";
      if(this.filtrosGender.length === 0 && this.filtrosSpecies.length === 0 && this.filtrosStatus.length === 0 && this.rows >= 826) {
        console.log("Cadena sin filtros");
      }
      else {
        if(this.filtrosGender.length !== 0){
          this.cadenaFiltros = this.cadenaFiltros + 'gender: "' + this.filtrosGender[0] + '"';
        }
        if(this.filtrosSpecies.length !== 0){
          if(this.cadenaFiltros !== " "){
            this.cadenaFiltros = this.cadenaFiltros + ', ';
          }
          this.cadenaFiltros = this.cadenaFiltros + 'species: "' + this.filtrosSpecies[0] + '"';
        }
        if(this.filtrosStatus.length !== 0){
          if(this.cadenaFiltros !== " "){
            this.cadenaFiltros = this.cadenaFiltros + ', ';
          }
          this.cadenaFiltros = this.cadenaFiltros + 'status: "' + this.filtrosStatus[0] + '"';
        }
        console.log("Cadena con filtros: "+this.cadenaFiltros);
        if(this.cadenaFiltros !== " "){
          this.cadenaFiltros = `filter: { ${this.cadenaFiltros} }`;
        }
        this.copyCadenaFiltros = this.cadenaFiltros;
        this.armaQuery();
      }
      this.scrollToTop();
    },
    buscarNombre(){
      if(this.buscaNombre !== " "){
        this.cadenaFiltros = `filter: { name: "${this.buscaNombre}" }`;
      }
      this.copyCadenaFiltros = this.cadenaFiltros;
      this.armaQuery();
    }
  }
};
</script>

<style>
img {
  width: 160px; 
  height: auto; 
}

.b-table tr {
  text-align: center; /* Centrado horizontal */
  vertical-align: middle; /* Centrado vertical */
}

.titulo {
  color: rgb(251, 149, 166);
  font-weight: bold;
  padding: 10px;
  margin: 20px;
  background: rgb(250, 250, 250);
}

.count {
  background-color: rgb(224, 224, 224);
  padding: 10px;
  margin: 20px;
  font-weight: bold;
}

.botonRosa {
  background : #fdb4bf !important;
  border: solid 1px #fdb4bf !important;
  height: 40px;
}

.botonRosa:hover {
  background : #f99aaa !important;
  border: solid 1px #fdb4bf !important;
}

.b-pagination button { /* estilos para todos los botones */
  background-color: #f8f9fa; /* Color de fondo de los botones */
  border-color: #fdb4bf; /* Color del borde del botón activo */
  color: #e57d90; /* Color del texto de los botones */
}

.b-pagination button:hover {
  color: #ffffff; 
  border-color: #fdb4bf; 
  background-color: #fdb4bf; /* Cambiar color de fondo en hover */
  font-weight: bold;
}

.page-item.active .page-link { /* el boton de la página actual */
    color: #fff;
    background-color: #e57d90;
    border-color: #e57d90;
    font-weight: bold;
}

.page-link.active, .active > .page-link {
    border-color: #ed3591;
    background-color: #ed3591; /* Color de fondo del botón activo */
    color: white;
}

.subtitulo {
  color: rgb(251, 149, 166);
  font-weight: bold;
  font-size: 25px;
  background: rgb(250, 250, 250);
}

.fondo {
  background: rgb(250, 250, 250);
  padding: 10px;
  margin-left: 20px;
  margin-bottom: 20px;
  border-radius: 10px;
}

#fondo {
  background: rgb(255, 2, 2);
  padding: 10px;
  margin-left: 20px;
  margin-bottom: 20px;
  border-radius: 10px;
}

#buscar {
  width: 300px;
  height: 40px;
  margin-right: 5px;
  border-radius: 8px;
  vertical-align: middle;
  padding: 10px;
  border: solid 2px gray;
}

.filtro {
  background: #ffe4ec;
  text-align: left;
  padding: 8px;
  border-radius: 8px;
  margin-bottom: 10px;
}

.filtro h2 {
  font-size: 15px;
  font-weight: bold;
  margin: 3px;
}

.filtro input {
  font-size: 3px;
  vertical-align: middle; 
  margin: 5px;
}

.filtro label {
  font-size: 14px;
  vertical-align: middle; 
  display: inline;
}

</style>
