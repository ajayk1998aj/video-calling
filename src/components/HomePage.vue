<template>
  <div class="home">
    <div id="nav">
      <button class="btn btn-success" @click="logoutUser">Logout</button>
    </div>
    <div class="form-group">
      <div class="form-group">
        <p>
          Welcome <b>{{ this.username }}</b
          >, your UID is <b>{{ this.uid }}</b> <br />
          Enter the receiver Id to start a chat
        </p>
        <p v-if="error">
          <b class="text-danger"> Receiver ID is required </b>
        </p>
        <input
          type="text"
          class="form-control"
          placeholder="Enter receiver UID"
          v-model="receiver_id"
        />
      </div>

      <div v-if="incomingCall">
        <button class="btn btn-success" @click="acceptCall">Accept Call</button>
        <button class="btn btn-success" @click="rejectCall">Reject Call</button>
      </div>
      <div v-else-if="ongoingCall">
        <button class="btn btn-secondary">Ongoing Call ...</button>
      </div>
      <div v-if="role === 'admin'">
        <button @click="startVideoChat" class="btn btn-secondary">
          Start Call
          <span v-if="showSpinner" class="fa fa-spin fa-spinner"></span>
        </button>
      </div>
    </div>
    <div id="callScreen"></div>
  </div>
</template>
<script>
import { CometChat } from "@cometchat-pro/chat";

export default {
  name: "home-page",
  data() {
    return {
      username: "",
      uid: "",
      session_id: "",
      receiver_id: null,
      error: false,
      showSpinner: false,
      incomingCall: false,
      ongoingCall: false,
      showCamera: false,
      img: null,
      camera: null,
      deviceId: null,
      devices: [],
      role: null,
    };
  },
  created() {
    if (this.$route.params.id !== undefined) {
      this.username = this.$route.params.id;
    }
    this.getLoggedInUser();
    let globalContext = this;
    var listnerID = "UNIQUE_LISTENER_ID";
    CometChat.addCallListener(
      listnerID,
      new CometChat.CallListener({
        onIncomingCallReceived(call) {
          console.log("Incoming call:", call);
          globalContext.incomingCall = true;
          globalContext.session_id = call.sessionId;
        },
        onOutgoingCallAccepted(call) {
          console.log("Outgoing call accepted:", call);
          globalContext.ongoingCall = true;
          CometChat.startCall(
            call.sessionId,
            document.getElementById("callScreen"),
            new CometChat.OngoingCallListener({
              onUserJoined: (user) => {
                /* Notification received here if another user joins the call. */
                console.log("User joined call:", user);
                /* this method can be use to display message or perform any actions if someone joining the call */
              },
              onUserLeft: (user) => {
                /* Notification received here if another user left the call. */
                console.log("User left call:", user);
                /* this method can be use to display message or perform any actions if someone leaving the call */
              },
              onCallEnded: (call) => {
                globalContext.ongoingCall = false;
                globalContext.incomingCall = false;
                /* Notification received here if current ongoing call is ended. */
                console.log("Call ended:", call);
                /* hiding/closing the call screen can be done here. */
              },
            })
          );
          // Outgoing Call Accepted
        },
        onOutgoingCallRejected(call) {
          console.log("Outgoing call rejected:", call);
          this.incomingCall = false;
          this.ongoingCall = false;
          this.receiver_id = "";
          // Outgoing Call Rejected
        },
        onIncomingCallCancelled(call) {
          console.log("Incoming call calcelled:", call);
        },
      })
    );
  },
  methods: {
    getLoggedInUser() {
      var user = CometChat.getLoggedinUser().then(
        (user) => {
          if (user === null) {
            var apiKey = "93c453c3b48a94f2fdafcc3eec18d5226084216b";
            this.showSpinner = true;
            CometChat.login(this.username, apiKey).then(
              () => {
                this.getLoggedInUser();
              },
              (error) => {
                this.showSpinner = false;
                console.log("Login failed with error:", error.code);
              }
            );
          } else {
            this.username = user.name;
            this.uid = user.uid;
            this.role = user.role;
          }
        },
        (error) => {
          console.log(error);
        },
        console.log(user)
      );
    },
    startVideoChat() {
      if (!this.receiver_id) this.error = true;
      this.showSpinner = true;
      var receiverID = this.receiver_id;
      var callType = CometChat.CALL_TYPE.VIDEO;
      var receiverType = CometChat.RECEIVER_TYPE.USER;
      var call = new CometChat.Call(receiverID, callType, receiverType);
      CometChat.initiateCall(call).then(
        (outGoingCall) => {
          this.showSpinner = false;
          console.log("Call initiated successfully:", outGoingCall);
          // perform action on success. Like show your calling screen.
          this.showCamera = true;
        },
        (error) => {
          console.log("Call initialization failed with exception:", error);
        }
      );
    },
    acceptCall() {
      let globalContext = this;
      this.ongoingCall = true;
      this.incomingCall = false;
      var sessionID = this.session_id;

      CometChat.acceptCall(sessionID).then(
        (call) => {
          console.log("Call accepted successfully:", call);
          console.log("call accepted now....");
          globalContext.showCamera = true;
          // start the call using the startCall() method
          console.log(globalContext.ongoingCall);
          var sessionId = call.sessionId;
          var callType = call.type;
          let showRecordingButton = true;
          var callSettings = new CometChat.CallSettingsBuilder()
            .setSessionID(sessionId)
            .enableDefaultLayout(true)
            .showRecordingButton(showRecordingButton)
            .setIsAudioOnlyCall(callType == "audio" ? true : false)
            .build();
          CometChat.startCall(
            callSettings,
            document.getElementById("callScreen"),
            new CometChat.OngoingCallListener({
              onUserJoined: (user) => {
                /* Notification received here if another user joins the call. */
                console.log("User joined call:", user);
                /* this method can be use to display message or perform any actions if someone joining the call */
              },
              onUserLeft: (user) => {
                /* Notification received here if another user left the call. */
                console.log("User left call:", user);
                /* this method can be use to display message or perform any actions if someone leaving the call */
              },
              onCallEnded: (call) => {
                /* Notification received here if current ongoing call is ended. */
                console.log("Call ended:", call);
                globalContext.ongoingCall = false;
                globalContext.incomingCall = false;
                globalContext.showCamera = false;
                /* hiding/closing the call screen can be done here. */
              },
              onRecordingStarted: (recordingStartedBy) => {
                // This event will work in JS SDK v3.0.2-beta1 & later.
                console.log(
                  "Listener => onRecordingStarted:",
                  recordingStartedBy
                );
              },
              onRecordingStopped: (recordingStoppedBy) => {
                // This event will work in JS SDK v3.0.2-beta1 & later.
                console.log(
                  "Listener => onRecordingStopped:",
                  recordingStoppedBy
                );
              },
            })
          );
        },
        (error) => {
          console.log("Call acceptance failed with error", error);
          // handle exception
        }
      );
    },
    rejectCall() {
      var sessionID = this.session_id;
      var globalContext = this;
      var status = CometChat.CALL_STATUS.REJECTED;
      CometChat.rejectCall(sessionID, status).then(
        (call) => {
          console.log("Call rejected successfully", call);
          globalContext.incomingCall = false;
          globalContext.ongoingCall = false;
          globalContext.receiver_id = "";
        },
        (error) => {
          console.log("Call rejection failed with error:", error);
        }
      );
    },
    logoutUser() {
      CometChat.logout().then(
        (success) => {
          console.log("Logout completed successfully");
          this.$router.push({ name: "login" });
          console.log(success);
        },
        (error) => {
          //Logout failed with exception
          console.log("Logout failed with exception:", { error });
        }
      );
    },
  },
  components: {},
};
</script>