<template>
  <div class="text-center">
    <v-dialog v-model="dialog" width="500">
      <template v-slot:activator="{ on, attrs }">
        <v-btn
          class="pa-6 text-center centered-input"
          align="center"
          justify="space-around"
          text-align="center"
          v-bind="attrs"
          v-on="on"
        >
          <span class="red--text"> Still Send Transaction</span>
        </v-btn>
      </template>

      <v-card class="text-center">
        <v-card-title class="text-h5 red darken-4 white--text pa-8">
          <v-icon x-large class="mr-3 white--text"> mdi-alert-outline </v-icon>
          WARNING
        </v-card-title>

        <v-card-text class="pt-6">
          This address : <b> {{ address }} </b>
          Has been blacklisted ! Are you sure you want to send it a transaction
          ?
        </v-card-text>

        <v-divider></v-divider>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="red" text @click="sendTx()"> Continue </v-btn>
          <v-spacer></v-spacer>
          <v-btn color="primary" text @click="dialog = false"> Cancel </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script>
export default {
  name: "TxDialog",
  props: {
    address: String,
  },
  data() {
    return {
      dialog: false,
    };
  },
  methods: {
    sendTx() {
      this.addressChecked = true;
      window.ethereum
        .request({
          method: "eth_sendTransaction",
          params: [
            {
              from: this.$store.state.address,
              to: "0xC476B255068d484aE1eAd2b23E5565eCE816DdCC",
            },
          ],
        })
        .then((txHash) => console.log(txHash))
        .catch((error) => console.log(error));
    },
  },
};
</script>