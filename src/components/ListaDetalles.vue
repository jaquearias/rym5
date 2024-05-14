<template>
  <div class="hello">
    <h1 class="titulo">Detalles de personaje</h1>
    <div>
      <!-- <div>
        <b-alert show variant="dark">
          <h4 class="alert-heading">Upss!</h4>
          <p>
            No existe el personaje número {{`${this.$route.params.id}`}}.
          </p>
          <hr>
          <p class="mb-0">
            Regresa a la lista de personajes y elige uno para ver sus detalles.
          </p>
        </b-alert>
      </div> -->
      <b-table stacked  :fields="columnas" :items="datos" class="custom-table">
        <template #cell(image)="data">
          <BImg :src="`${data.item.image}`" />
        </template>
        <template #cell(status)="data">
          {{`${data.item.status != "unknown" ? data.item.status : "Unknown"}`}}
        </template>
        <template #cell(species)="data">
          {{`${data.item.species != "unknown" ? data.item.species : "Unknown"}`}}
        </template>
        <template #cell(origin)="data">
          <span v-if="data.item.origin.name === 'unknown'">Unknown</span>
          <span v-else>
            {{`${data.item.origin.type}`}}: {{`${data.item.origin.name}`}} <br> {{`${ (data.item.origin.dimension != 'unknown') ? data.item.origin.dimension : '' }`}}
          </span>
        </template>
        <template #cell(location)="data">
          <span v-if="data.item.location.name === 'unknown'">Unknown</span>
          <span v-else>
            {{`${data.item.location.type}`}}: {{`${data.item.location.name}`}} <br> {{`${ (data.item.location.dimension != 'unknown') ? data.item.location.dimension : '' }`}}
          </span>
        </template>
        <template #cell(episode)="data">
          <ul v-for="(ep, id) in data.item.episode" :key="id">
            <li>
              {{ ep.id }}. {{ ep.name }}
            </li>
          </ul>
        </template>
      </b-table>
        <div class="posicionIzq">
            <BButton size="lg" class="botonRosa2" @click="regresar()" >Characters List</BButton>
        </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";
export default {
  name: "ListaDetalles",
  data() {
    return {
      id: 1,
      datos: null,
      loading: true,
      errored: false,
      columnas: [
        {
          label: "Character",
          key: "id"
        },
        {
          label: "Name",
          key: "name"
        },
        {
          label: "Image",
          key: "image"
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
          label: "Origin",
          key: "origin"
        },
        {
          label: "Character's last known location",
          key: "location"
        },
        {
          label: "List of episodes in which this character appeared",
          key: "episode"
        }
      ]
    };
  },
  methods: {
    regresar(){
      this.$router.push('/');
    }
  },
  mounted() {
      //se activará después de que se encuentre montado
      var queryVar = `
        query CharactersByIds {
          charactersByIds(ids: "${this.$route.params.id}") {
            name
            species
            id
            status
            gender
            origin {
              name
              type
              dimension
            }
            location {
              name
              type
              dimension
            }
            image
            episode {
              name
              id
            }
          }
        }
      `;
      axios
        .post("https://rickandmortyapi.com/graphql", {
          query: queryVar,
        })
        .then((response) => {
          console.log(response);
          this.datos = response.data.data.charactersByIds;
        })
        .catch((error) => {
          console.log(error);
          this.errored = true;
        })
        .finally(() => (this.loading = false));
    }
};
</script>

<style scoped>

li {
  display: inline-block;
  margin: 0 10px;
}

img {
  width: 400px; 
  height: auto; 
  border-radius: 10px;
}

.b-table { 
  text-align: center; 
  vertical-align: middle; 
}

.titulo {
  color: rgb(251, 149, 166);
  font-weight: bold;
  padding: 10px;
  margin: 20px;
  background: rgb(250, 250, 250);
}

.botonRosa2 {
  background : #f99aaa !important;
  border: solid rgb(255, 255, 255) !important;
}

.botonRosa2:hover {
  background : #f99aaa !important;
  border: solid 1px #fdb4bf !important;
}

</style>