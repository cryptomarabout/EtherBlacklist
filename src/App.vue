<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <div class="d-flex align-center">
        <v-img
          alt="Vuetify Logo"
          class="shrink mr-2"
          contain
          :src="require('./assets/logo.svg')"
          transition="scale-transition"
          width="40"
        />

        <h2 class="white--text font-weight-light">EtherBlacklist</h2>
      </div>

      <v-spacer></v-spacer>

      <v-btn
        class="rounded-xl pa-6"
        align="center"
        justify="space-around"
        v-on:click="connectionMetamask()"
      >
        <span v-if="this.$store.state.address === ''"> Connect Wallet</span>
        <span v-else> {{ this.$store.state.address }}</span>
        <v-tooltip v-if="this.$store.state.address !== ''" bottom>
          <template v-slot:activator="{ on, attrs }">
            <v-icon
              color="secondary"
              v-bind="attrs"
              v-on="on"
              v-on:click="copyAddress"
              class="ml-2"
            >
              mdi-content-copy
            </v-icon>
          </template>
          <span>{{ msg }}</span>
        </v-tooltip>
      </v-btn>
    </v-app-bar>

    <v-main>
      <Home />
    </v-main>
  </v-app>
</template>

<script>
import Home from "./components/Home";

export default {
  name: "App",

  components: {
    Home,
  },

  data: () => ({
    connectionAsked: false,
    msg: "Copy Address",
  }),
  methods: {
    async copyAddress() {
      await navigator.clipboard.writeText(this.$store.state.address);
      this.msg = "Copied";
    },
    async connectionMetamask() {
      if (this.connectionAsked === false) {
        this.connectionAsked = true;
        var accounts = await window.ethereum.request({
          method: "eth_requestAccounts",
        });
        this.$store.commit("updateAddress", accounts[0]);
      } else {
        this.copyAddress();
      }
    },
  },
};
</script>