<template>
  <div id="app">
    <router-view/>
  </div>
</template>
<script>
import { CometChat } from "@cometchat-pro/chat";
let apiId = "2011402a2e1bbe91";
const region = "us";
const appSetting = new CometChat.AppSettingsBuilder()
  .subscribePresenceForAllUsers()
  .setRegion(region)
  .build();
import "./App.css";
export default {
  data() {
    return {};
  },
  created() {
    this.initializeApp();
  },
  methods: {
    initializeApp() {
      debugger;
      var appID = apiId;//process.env.VUE_APP_COMMETCHAT_APP_ID;
      CometChat.init(appID,appSetting).then(
        () => {
          console.log("Initialization completed successfully");
        },
        error => {
          console.log("Initialization failed with error:", error);
        }
      );
    }
  }
};
</script>