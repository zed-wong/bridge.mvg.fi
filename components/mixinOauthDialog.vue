<template>
  <v-dialog
    v-model="connectMixinDialog"
    class="dialog-css"
    max-width="500px"
    overlay-opacity="0.5"
  >
    <v-sheet
      class="align-self-start px-9 py-8"
      v-if="!$vuetify.breakpoint.mobile"
    >
      <v-row class="d-flex flex-column mb-0">
        <v-col class="align-center d-flex flex-row pa-0 mb-2">
          <v-spacer />
          <v-btn icon @click="connectMixinDialog = false">
            <v-icon color="black"> mdi-close </v-icon>
          </v-btn>
        </v-col>
        <v-col
          class="py-6 d-flex justify-center oauth-qr-background"
          v-if="qrLoaded"
        >
          <v-card max-width="200px">
            <v-row>
              <vue-qrcode
                v-if="qrUrl"
                :value="qrUrl"
                :options="{ width: 200 }"
              />
              <v-overlay opacity="0" absolute>
                <v-img
                  :src="mixinMessengerLogo"
                  class="mixin-logo-overlay"
                  v-if="!mixinConnected"
                />
                <v-icon v-if="mixinConnected" size="64px" class="oauth-qr-scanned">
                  mdi-checkbox-marked-circle
                </v-icon>
              </v-overlay>
            </v-row>
          </v-card>
        </v-col>
        <v-col class="my-10 d-flex justify-center" v-if="!qrLoaded">
          <v-progress-circular indeterminate color="primary" />
        </v-col>
        <div class="my-5">
          <v-col class="d-flex justify-start">
            <span class="oauth-guide-title">
              {{ $t("mixin_oauth_title") }}
            </span>
          </v-col>
          <v-col class="d-flex justify-start">
            <span class="oauth-guide-text">
              {{ $t("mixin_oauth_text") }}
            </span>
          </v-col>
        </div>
      </v-row>
    </v-sheet>

    <v-sheet class="align-self-start px-9 py-8" v-else>
      <v-row class="d-flex flex-column mb-0">
        <v-col class="align-center d-flex flex-row pa-0 mb-2">
          <v-spacer />
          <v-btn icon @click="connectMixinDialog = false">
            <v-icon color="black"> mdi-close </v-icon>
          </v-btn>
        </v-col>
        <v-col
          class="d-flex justify-center oauth-qr-mobile-background"
          v-if="qrLoaded"
        >
          <v-btn elevation="0" color="#ebf8ff" @click="redirect(qrUrl)">
            <v-img :src="mixinMessengerLogo" class="mr-2 mixin-logo-overlay" />
            <span> {{ $t("mixin_oauth_mobile_title") }} </span>
            <v-icon v-if="mixinConnected" size="16px" color="green" class="ml-1">
              mdi-checkbox-marked-circle
            </v-icon>
          </v-btn>
        </v-col>
      </v-row>
    </v-sheet>
  </v-dialog>
</template>

<script>
import authorize from "../helpers/oauth/authorize";
import { userMe } from "../helpers/registry";
import VueQrcode from "@chenfengyuan/vue-qrcode";

const clientID = process.env.NFT_OAUTH_BOT_ID;
const scope = "PROFILE:READ";
const mixinMessengerLogo = "https://static.fox.one/image/icon_mixin@32x32.png";

export default {
  components: { VueQrcode },
  data() {
    return {
      mixinMessengerLogo,
      qrUrl: "",
      qrLoaded: false,
      accessToken: null,
      outputs: null,
      client: null,
    };
  },
  computed: {
    connectMixinDialog: {
      get() {
        return this.$store.state.connectMixinDialog;
      },
      set(value) {
        this.$store.commit("toggleConnectMixin", value);
      },
    },
    mixinConnected: {
      get() {
        return this.$store.state.mixinConnected;
      },
      set(n) {
        this.$store.commit("connectMixin", n);
      },
    },
  },
  async mounted() {
    this.client = this.auth();
  },
  watch: {
    connectMixinDialog(n) {
      if (n) this.client = this.auth();
    }
  },
  methods: {
    qrAfterSuccess() {
      setTimeout(() => {
        this.qrUrl = "";
        this.connectMixinDialog = false;
      }, 1000);
    },
    redirect(url) {
      location.href = url
    },
    auth() {
      authorize(
      { clientId: clientID, scope: scope, pkce: true },
      {
        onShowUrl: (url) => {
          this.qrUrl = url;
          this.qrLoaded = true;
        },
        onError: (error) => {
          this.connectMixinDialog = false;
          console.error(error);
          return;
        },
        onSuccess: async (data) => {
          this.accessToken = data;
          this.qrAfterSuccess();
          const user = await userMe(data);
          this.mixinConnected = true;
          this.$store.commit("connectMixin", true);
          this.$store.commit("setOauthUser", user);
          localStorage.setItem("access_token", data);
          localStorage.setItem("oauth_user", JSON.stringify(user));
          return;
        },
      }
    );
    }
  },
};
</script>

<style scoped>
.mixin-logo-overlay {
  border-radius: 8px;
  background-color: white;
}
.oauth-qr-background {
  border-radius: 8px;
  background-color: #f2f3f6;
}
.oauth-qr-scanned {
  color: #4caf50;
  background-color: #e8f5e9;
  border-radius: 50%;
}
.oauth-qr-mobile-background {
  border-radius: 8px;
  background-color: #ebf8ff;
}
.oauth-guide-title {
  font-size: 18px;
  font-weight: 600;
  line-height: 21px;
}
.oauth-guide-text {
  color: #808080;
  font-size: 14px;
  font-weight: 500px;
  line-height: 150%;
}
</style>