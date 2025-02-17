<template>
  <v-tooltip bottom>
    <template v-slot:activator="{ on, attrs }">
      <v-btn
        height="40px"
        elevation="0"
        rounded
        :color="'#f4f7fa'"
        class="ml-3 current-network-btn"
        @click="detect"
        v-bind="attrs"
        v-on="on"
      >
        <v-icon class="mr-2" v-if="alertRed" color="red" size="24px"> mdi-alert-circle-outline </v-icon>
        <v-avatar size="24" class="mr-2" v-if="networkIcon && !alertRed">
          <v-img :src="networkIcon" />
        </v-avatar>
        <span v-if="alertRed" style="font-size:16px"> {{ btnText }} </span>
        <span v-else> {{ btnText }} </span>
      </v-btn>
    </template>
    <span v-if="alertRed"> {{ $t("click_to_switch_to") }} {{ supposeNetworkName }} </span>
    <span v-else> {{ $t("current_connected_network") }} </span>
  </v-tooltip>
</template>

<script>
import { useOnboard } from "@web3-onboard/vue";

export default {
  data() {
    return {
      btnText: "",
      networkIcon: "",
      alertRed: false,
    };
  },
  mounted() {
    this.loopDetectNetwork();
  },
  computed: {
    confirmDepositDialog() {
      return this.$store.state.confirmDepositDialog;
    },
    selectedNetwork: {
      get() {
        return this.$store.state.fromNetwork;
      },
    },
    nftNetwork: {
      get() {
        return this.$store.state.nft.fromNetwork;
      }
    },
    connectedChain() {
      return this.$store.state.chainId;
    },
    connected() {
      return this.$store.state.connected;
    },
    tokenMode() {
      return this.$store.state.mode;
    },
    nftMode() {
      return this.$store.state.nft.mode;
    },
    currentRoute() {
      return this.$route.path;
    },
    supposeNetworkName() {
      if (this.currentRoute === "/") return this.tokenMode === 0 ? this.selectedNetwork.name +" "+ this.$t('mainnet') : this.$t('mvm_mainnet');
      if (this.currentRoute === "/nft") return this.nftMode === 0 ? this.nftNetwork.name +" "+ this.$t('mainnet') : this.$t('mvm_mainnet');
    }
  },
  watch: {
    confirmDepositDialog(n) {
      this.check();
    },
    connectedChain(n) {
      this.check();
    },
    connected(n) {
      this.check();
    },
    tokenMode() {
      this.check();
    },
    nftMode() {
      this.check();
    },
    currentRoute() {
      this.check();
    },
  },
  methods: {
    async check() {
      let network_id;
      if (!this.connectedChain) {
        const { connectedChain } = useOnboard();
        network_id = connectedChain.value?.id;
        this.$store.commit("updateChainId", network_id);
      } else {
        network_id = this.connectedChain;
      }

      this.alertWhenMatters(network_id);
      switch (network_id) {
        case "0x1":
          this.btnText = this.$t("ethereum_mainnet");
          this.networkIcon =
            "https://mixin-images.zeromesh.net/zVDjOxNTQvVsA8h2B4ZVxuHoCF3DJszufYKWpd9duXUSbSapoZadC7_13cnWBqg0EmwmRcKGbJaUpA8wFfpgZA=s128";
          return;
        case "0x120c7":
          this.btnText = this.$t("mvm_mainnet");
          this.networkIcon = "https://mvm.dev/logo.svg";
          return;
      }
      this.btnText = this.$t("unsupported_network");
      this.networkIcon = "";
      return false;
    },
    alertWhenMatters(network_id) {
      this.alertRed = false;
      if (network_id != "0x120c7") {
        if (this.tokenMode === 1 && this.currentRoute === "/") {
          this.alertRed = true;
          return;
        }
        if (this.nftMode === 1 && this.currentRoute === "/nft") {
          this.alertRed = true;
          return;
        }
      }
      if (network_id != "0x1") {
        if (this.tokenMode === 0 && this.currentRoute === "/") {
          if (this.selectedNetwork.symbol === "ETH") {
            this.alertRed = true;
            return;
          }
        }
        if (this.tokenMode === 0 && this.currentRoute === "/nft") {
          if (this.nftNetwork.symbol === "ETH") {
            this.alertRed = true;
            return;
          }
        }
      }
    },
    switchTo(chainId) {
      const { connectedWallet, setChain } = useOnboard();
      if (connectedWallet) {
        setChain({ wallet: connectedWallet.value.label, chainId });
      }
    },
    detect() {
      if (this.tokenMode === 0 && this.currentRoute === "/") {
        if (this.selectedNetwork.symbol === "ETH") {
          this.switchTo("0x1");
          return;
        }
      }
      if (this.nftMode === 0 && this.currentRoute === "/nft") {
        if (this.selectedNetwork.symbol === "ETH") {
          this.switchTo("0x1");
          return;
        }
      }

      if (this.tokenMode === 1 && this.currentRoute === "/") {
        this.switchTo("0x120c7");
        return;
      }
      if (this.nftMode === 1 && this.currentRoute === "/nft") {
        this.switchTo("0x120c7");
        return;
      }
      this.check();
    },
    async loopDetectNetwork() {
      while (true) {
        await this.check();
        await new Promise((r) => setTimeout(r, 2000));
      }
    },
  },
};
</script>

<style>
.current-network-btn {
  max-width: 180px;
}
</style>