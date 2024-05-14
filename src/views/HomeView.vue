
<template>
  <div class="home">
    <ListaPersonajes msg="Welcome to Your Vue.js App"/>
  </div>
</template>

<script>
// @ is an alias to /src
import ListaPersonajes from '@/components/ListaPersonajes.vue'

export default {
  name: 'HomeView',
  components: {
    ListaPersonajes
  }
}
</script>
