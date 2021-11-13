<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <div style="font-size: 32px;">Habits</div>
    </v-app-bar>

    <v-main id="main" :class="darkMode ? 'dark' : ''">
      <Dashboard v-model="darkMode" />
    </v-main>
  </v-app>
</template>

<script>
import Dashboard from "./components/Dashboard";

export default {
  name: "App",

  components: {
    Dashboard,
  },

  data: () => ({
    darkMode: false,
  }),
};
</script>

<style scoped>
#main.dark {
  background: #252526;
}
</style>
