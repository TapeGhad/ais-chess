<template>
  <div>
    <v-btn text class="btn-for-after" @click="openLogin(true)">
      <span class="hidden-sm-and-down">
        login
      </span>
      <v-icon right>mdi-login</v-icon>
    </v-btn>
    <v-dialog persistent v-model="$store.state.registerTab">
      <v-container class="fill-height" fluid>
        <v-row align="center" justify="center">
          <v-col sm="8" md="4">
            <v-card>
              <ModalTabs />
              <v-btn icon large @click="openLogin(false)" class="btn-close">
                <v-icon>mdi-close</v-icon>
              </v-btn>
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-dialog>
  </div>
</template>

<script>
/* eslint-disable */
import ModalTabs from "./ModalTabs";

export default {
  name: 'login_signup_social',
  components: { ModalTabs },
   data() {
    return {
      dialog: false
    };
  },
  methods: {
    openLogin(value) {
      this.$store.dispatch("setRegisterTab", value);
    }
  }
}
</script>

<style lang="scss" scoped>
.btn-for-after {
  position: relative;
  margin-right: 5px;
  &::after {
    content: '';
    position: absolute;
    height: 100%;
    width: 1px;
    background-color: #303030;
    top: 0;
    right: -5px;
  }
}
.btn-close {
  position: absolute;
  right: 0;
  top: 0;
  &::before {
    border-radius: 5px;
  }
}
</style>