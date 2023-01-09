<template>
  <v-app>
    <v-navigation-drawer v-model="drawer" app clipped permanent width="250">
      <v-list>
        <v-list-item
          v-for="item in items"
          :key="item.title"
          @click="goTo(item)"
        >
          <v-list-item-icon>
            <v-icon>{{ item.icon }}</v-icon>
          </v-list-item-icon>
          <v-list-item-title>{{ item.title }}</v-list-item-title>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>
    <v-app-bar app color="primary" dark fixed prominent height="55">
      <v-app-bar-nav-icon @click.stop="drawer = !drawer"></v-app-bar-nav-icon>
      <v-toolbar-title>ID Card Management</v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn :disabled="!connected" @click="disconnect" text>
        Disconnect
      </v-btn>
    </v-app-bar>
    <v-main>
      <router-view></router-view>
    </v-main>
  </v-app>
</template>

<script>
// import { mapState } from "vuex";
import axios from "axios";

export default {
  data() {
    return {
      drawer: false,
      items: [
        { icon: "mdi-home", title: "Home", route: "/" },
        { icon: "mdi-cog", title: "Settings", route: "/settings" },
      ],
    };
  },
  computed: {
    // ...mapState({
    //   connected: (state) => state.database.connected,
    // }),
  },
  methods: {
    goTo(item) {
      this.$router.push(item.route);
      this.drawer = false;
    },
    async disconnect() {
      try {
        await axios.post("/api/disconnect");
        this.$store.dispatch("database/disconnect");
      } catch (error) {
        console.error(error);
      }
    },
  },
};
</script>
