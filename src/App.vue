<template>
  <v-app id="inspire">
    <v-app id="inspire">
      <v-navigation-drawer v-model="drawer" app clipped>
        <v-list dense>
          <v-list-item link>
            <v-list-item-action>
              <v-icon>mdi-view-dashboard</v-icon>
            </v-list-item-action>
            <v-list-item-content>
              <v-list-item-title>Dashboard</v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </v-list>
      </v-navigation-drawer>

      <v-app-bar app clipped-left>
        <v-app-bar-nav-icon @click.stop="drawer = !drawer"></v-app-bar-nav-icon>
        <v-toolbar-title>Battle Royalle</v-toolbar-title>
      </v-app-bar>

      <v-main>
        <v-container class="fill-height" fluid>
          <v-row>
            <v-col>
              <VueTerminal console-sign="$" allow-arbitrary @command="onCliCommand"></VueTerminal>
            </v-col>
          </v-row>
        </v-container>
      </v-main>

      <v-footer app>
        <span>&copy; {{ new Date().getFullYear() }}</span>
      </v-footer>
    </v-app>
  </v-app>
</template>

<script>
import VueTerminal from "vue-terminal-ui";
import * as signalR from "@microsoft/signalr";

let connection = "";

export default {
  name: "App",
  components: {
    VueTerminal
  },
  props: {
    source: String
  },
  data: () => ({
    drawer: null
  }),
  created() {
    this.$vuetify.theme.dark = true;

    connection = new signalR.HubConnectionBuilder()
      .withUrl("https://localhost:44325/commandhub")
      .configureLogging(signalR.LogLevel.Information)
      .build();

    async function start() {
      try {
        await connection.start();
        console.log("connected");
      } catch (err) {
        console.log(err);
        setTimeout(() => start(), 5000);
      }
    }

    connection.onclose(async () => {
      await start();
    });

    // Start the connection.
    start();

    //connection.start().then(() => console.log("connected"));

    //connection.start().catch(err => console.error(err));
  },
  methods: {
    onCliCommand(data, resolve) {
      // typed command is available in data.text
      console.log(data);
      connection.invoke("AdicionarComando", "vue", data.text).catch(err => console.error(err));
      setTimeout(() => {
        resolve("");
      }, 300);
    }
  }
};
</script>
