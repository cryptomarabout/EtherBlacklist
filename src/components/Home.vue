<template>
  <v-container>
    <v-row>
      <v-col class="mt-15 text-center centered-input" cols="10" sm="6">
        <!-- <v-card
          elevation="3"
          style="
            min-width: 450px;
            height: 280px;
            border-radius: 20px;
            border: 1px solid;
            font-weight: 500;
            opacity: 0.95;
          "
          class="pa-5 white mr-4"
        > -->
        <h1 class="display-2 font-weight-light mb-3">Blacklist checker</h1>

        <p class="subheading font-weight-regular">
          Please enter an address to check if it has been blacklisted
        </p>
        <v-spacer />
        <v-spacer />
        <v-text-field
          v-model="eth_address"
          label="Enter Ethereum address"
          solo-inverted
          shaped
        />
        <v-spacer />
        <v-btn
          color="secondary"
          class="ma-1 pa-6 white--text"
          align="center"
          justify="space-around"
          @click="checkAddress(eth_address)"
        >
          Check on EtherBlacklist
          <v-icon dark right> mdi-magnify </v-icon>
        </v-btn>
        <v-spacer />
        <v-btn
          v-if="covalentResult !== null"
          color="secondary"
          class="ma-1 pa-6 white--text"
          align="center"
          justify="space-around"
          @click="dialog = true"
        >
          Consult address history
          <v-icon dark right> mdi-magnify </v-icon>
        </v-btn>
        <v-dialog
          v-model="dialog"
          v-if="covalentResult !== null"
          :retain-focus="false"
        >
          <v-card color="primary" dark>
            <v-card-title> <h1>Transaction History</h1></v-card-title>
            <template>
              <div class="text-h4 font-weight-light" align="right">
                Number of Transactions :
                <v-chip
                  class="text-subtitle-1 font-weight-light secondary"
                  margin="500px"
                >
                  {{ covalentResult.data.items.length }}
                </v-chip>
              </div>
            </template>
            <v-card-text class="pa-0">
              <v-data-table
                dense
                :headers="headers"
                :items="covalentResult.data.items"
                :disable-pagination="false"
                align="center"
                class="pa-0"
                :header-props="{ sortIcon: null }"
              >
                <template
                  v-for="header in headers.filter((header) =>
                    header.hasOwnProperty('formatter')
                  )"
                  v-slot:[`item.${header.value}`]="{ value }"
                >
                  {{ header.formatter(value) }}
                </template>
                <template v-slot:[`item.log_events`]="{ item }" align="center">
                  
                  
                    <v-chip @click="dialog2 = true" class="white--text"> Event List </v-chip>
                    <v-dialog v-model="dialog2">
                  <v-chip
                    v-for="event in item.log_events"
                    :key="event.block_signed_at"
                    class="text-subtitle-1 font-weight-light"
                    style="margin: 5px"
                  >
                    <span v-if="event.decoded"> {{ event.decoded.name }} </span>
                    <span v-else> Decode event failed </span>
                  </v-chip>
                  </v-dialog>
                </template>

                <template v-slot:footer>
                  <v-row class="ma-3">
                    <v-spacer />
                    <h2>Data source:</h2>
                    <v-img
                      class="ml-3"
                      max-width="140px"
                      src="@/assets/covalent-logo.png"
                    />
                    <v-spacer />
                  </v-row>
                </template>
                <template v-slot:no-data> No data </template>
              </v-data-table>
            </v-card-text>
          </v-card>
        </v-dialog>
        <!-- </v-card> -->
      </v-col>
      <v-spacer />
      <v-col class="my-4" v-if="this.addressChecked">
        <v-card
          elevation="3"
          style="
            width: 450px;
            min-height: 280px;
            border-radius: 20px;
            border: 1px solid;
            font-weight: 500;
            opacity: 0.95;
          "
          class="pa-5 white mr-4 mt-7"
        >
          <div v-if="badgeResults[1].length > 0">
            <v-row class="ma-2">
              <h2 class="font-weight-light">Blacklist marks :</h2>
              <v-spacer />
              <v-chip
                class="text-subtitle-1 font-weight-light red my-1"
                outlined
                text-color="red"
              >
                {{ badgeResults[1].length }} marks
              </v-chip>
              <v-alert type="error" dense class="mt-1" style="width: 500px">
                WARNING<br />
                Blacklisted address, please be carefull !
              </v-alert>
            </v-row>
            <v-list-item
              three-line
              v-for="badge in badgeResults[1]"
              :key="badge"
              class="my-2 pa-0"
              style="border: 1px solid red"
            >
              <v-list-item-avatar width="80px" height="80px" class="ml-2">
                <v-img
                  contains
                  :src="badges[badge].uri"
                  transition="scale-transition"
                />
              </v-list-item-avatar>

              <v-list-item-content align="left">
                <v-list-item-title
                  v-text="badges[badge].name"
                ></v-list-item-title>
                <v-list-item-subtitle
                  class="font-italic"
                  v-text="badges[badge].tags"
                ></v-list-item-subtitle>
                <v-list-item-subtitle
                  v-text="badges[badge].description"
                ></v-list-item-subtitle>
              </v-list-item-content>

              <v-list-item-action>
                <v-tooltip bottom>
                  <template v-slot:activator="{ on, attrs }">
                    <v-icon
                      color="grey lighten-1"
                      v-bind="attrs"
                      v-on="on"
                      class="mr-2 mt-4"
                    >
                      mdi-information
                    </v-icon>
                  </template>
                  <span>More informations</span>
                </v-tooltip>
              </v-list-item-action>
            </v-list-item>
          </div>
          <div v-else>
            <v-row class="ma-4">
              <h2 class="font-weight-light">Blacklist marks :</h2>
              <v-chip
                class="text-subtitle-1 font-weight-light green ml-4 mt-1"
                outlined
                text-color="green"
              >
                None
              </v-chip>
            </v-row>
            <v-col class="text-center">
              <v-alert type="success" dense class="mt-1 text-left">
                Not blacklisted, you can interact "safely"
              </v-alert>
              <v-spacer />
              <v-btn
                class="pa-6 mt-10 mb-n15 text-center centered-input"
                align="center"
                justify="space-around"
                text-align="center"
                v-on:click="sendTx()"
              >
                <span class="green--text"> Send Transaction</span>
              </v-btn>
            </v-col>
          </div>
          <v-col class="text-center"> </v-col>
          <TxDialog
            v-if="badgeResults[1].length > 0"
            :address="this.eth_address"
          />
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import { mapState } from "vuex";
import TxDialog from "./TxDialog";
import axios from "axios";
import Web3 from "web3";
import json from "./BadgeDefinitionFactory.json";
import json2 from "./BadgeTokenFactory.json";
axios.headers= {"Access-Control-Allow-Origin": "*"}

export default {
  name: "Home",
  components: {
    TxDialog,
  },

  data() {
    return {
      eth_address: "",
      addressChecked: false,
      addressIsBlacklisted: true,
      fakeResult: ["badge0", "badge1"],
      openDialog: false,
      myJson: json,
      myJson2: json2,
      badges: [],
      badgeResults: [],
      covalentResult: null,
      dialog2: false,
      dialog: false,
    };
  },
  watch: {
    async address() {
      await this.getAllBadges();
    },
  },
  computed: {
    ...mapState(["address"]),
    headers() {
      return [
        {
          value: "block_signed_at",
          text: "Date",
          sortable: true,
          class: "text-h5",
          formatter: (value) => {
            return value.replace("T", " ").replace("Z", " ");
          },
          width: "5px",
        },
        {
          value: "from_address",
          text: "From",
          sortable: true,
          class: "text-h5",
          align: "center",
        },
        {
          value: "to_address",
          text: " To ",
          sortable: true,
          class: "text-h5",
          align: "center",
        },
        {
          value: "value_quote",
          text: "Value",
          sortable: true,
          class: "text-h5",
          formatter: this.parseValueQuote,
          align: "center",
        },
        {
          value: "log_events",
          text: "Events",
          sortable: true,
          class: "text-h5",
          align: "center",
        },
        {
          value: "tx_hash",
          text: "Hash",
          sortable: false,
          class: "text-h5",
          align: "center",
          width: "5px",
        },
      ];
    },
  },
  methods: {
    queryCovalent() {
      const path =
        "https://api.covalenthq.com/v1/1/address/" +
        this.eth_address +
        "/transactions_v2/?&key=ckey_2000734ae6334c75b8b44b1466e";
      axios
        .get(path)
        .then((res) => {
          console.log(res.data);
          this.covalentResult = res.data;
        })
        .catch((error) => {
          // eslint-disable-next-line
          console.error(error);
        });
    },
    checkAddress() {
      this.addressChecked = false;
      this.addressChecked = true;
      this.getBadgesFromAddress();
      this.queryCovalent();
    },
    async getBadgesFromAddress() {
      const web3 = new Web3(window.ethereum, null, {
        transactionConfirmationBlocks: 1,
      });
      var contract = new web3.eth.Contract(
        this.myJson2.abi,
        "0x254dffcd3277C0b1660F6d42EFbB754edaBAbC2B"
      );
      var query = await contract.methods
        .getAllOwnedBadges(this.eth_address)
        .call({ from: this.$store.state.address });
      console.log(query);
      this.badgeResults = query;
    },
    async getAllBadges() {
      var badgesTmp = [];
      var myRegEx = /'.*?'/;
      const web3 = new Web3(window.ethereum, null, {
        transactionConfirmationBlocks: 1,
      });
      var contract = new web3.eth.Contract(
        this.myJson.abi,
        "0xcfeb869f69431e42cdb54a4f4f105c19c080a601"
      );
      for (var badgeId in [0, 1, 2]) {
        var query = await contract.methods
          .getBadgeDefinition(badgeId)
          .call({ from: this.$store.state.address });
        console.log(query["_name"].match(myRegEx)[0]);
        console.log(query);
        badgesTmp[badgeId] = {
          name: query["_name"].match(myRegEx)[0],
          description: query["_description"],
          uri: query["_image_uri"],
          //uri: "https://einvestidor.estadao.com.br/wp-content/uploads/sites/715/2021/03/hacker3342696-19202_150320215733.jpg",
          tags: query["_tags"],
        };
        //var res = await query.send({ from: this.$store.state.address });
        console.log(badgesTmp);
      }
      this.badges = badgesTmp;
    },
    displayEthAddress(address, addressIsCaller) {
      if (addressIsCaller) {
        return "You";
      }
      return address.slice(0, 4) + "..." + address.slice(-2);
    },
    formatPrice(val) {
      return `$${val.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ",")}`;
    },
    parseValueQuote: function (value) {
      if (!value) return "0";
      return "$" + parseInt(value).toLocaleString();
    },
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