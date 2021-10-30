<template>
  <v-app>
    <v-main>
      <Meteo/>
    </v-main>
  </v-app>
</template>

<script>
import Meteo from './components/Meteo';

export default {
  name: 'App',

  components: {
    Meteo,
  },

  data: () => ({
    //
  }),
};
</script>
