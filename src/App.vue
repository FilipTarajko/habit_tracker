<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <div style="font-size: 32px;">Habits</div>
    </v-app-bar>

    <v-main>
      <div style="padding: 30px;">
        <Dashboard />
      </div>
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
    //
  }),
};
</script>
