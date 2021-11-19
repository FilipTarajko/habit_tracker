<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <div style="font-size: 32px;">Habits</div>
      <div
        style="display: flex; justify-content: flex-end; margin: auto 0 auto auto"
      >
        <div
          v-for="(lang, index) in langs"
          :key="index"
          style="margin-top: 6px; margin-left: 12px;"
        >
          <img
            style="width: 50px; height: 30px; border-radius: 6px; cursor:pointer;"
            :src="require(`./assets/images/${lang}.png`)"
            @click="$i18n.locale = lang"
          />
        </div>
      </div>
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
    langs: ["en", "pl"],
  }),
  methods: {
    flagPathFromCode: function(code) {
      return "./assets/" + code + ".png";
    },
  },
};
</script>

<style scoped>
#main.dark {
  background: #252526;
}
</style>
