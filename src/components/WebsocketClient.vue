<template>
    <div>
      <three-canvas :gltf-model="received_message"></three-canvas>
        <p>test</p>
        <div>
            <p>Websocket connection</p>
            <button @click="connect">Connect</button>
            <button @click="disconnect">Disconnet</button>
        </div>
        <div>
            <div>
              <p>What is your name?</p>
              <input
                v-model="send_message"
                placeholder="What do you want to access"
              >
            </div>
            <button
              @click="receiveFBX"
            >Get Fbx</button>
            <button
              @click="receiveGLTF"
            >Get GLTF</button>
            <button @click="donwloadAsFbx">Download</button>
        </div>
        <div>{{ [received_message].length }}</div>
    </div>
</template>

<script>
import SockJS from "sockjs-client";
import Stomp from "stompjs";
import ThreeCanvas from "./ThreeCanvas.vue"
export default {
  name: "WebsocketClient",
  data() {
    return {
      received_message: undefined,
      send_message: null,
      connected: false
    };
  },
  components: {
    ThreeCanvas
  },
  methods: {
    receiveFBX() {
      console.log("Send message:" + this.send_message);
      if (this.stompClient && this.stompClient.connected) {
        const msg = { fileName: this.send_message };
        this.stompClient.send("/app/fbx-models", {}, JSON.stringify(msg));
      }
    },
    receiveGLTF() {
      if (this.stompClient && this.stompClient.connected) {
        const msg = { fileName: this.send_message };
        this.stompClient.send("/app/gltf-models", {}, JSON.stringify(msg));
      }
    },
    connect() {
      this.socket = new SockJS("http://localhost:8080/resources");
      this.stompClient = Stomp.over(this.socket);
      this.stompClient.connect(
        {},
        () => {
          this.connected = true;
          this.stompClient.subscribe("/topic/fileAccess", tick => {
            this.received_message = JSON.parse(tick.body).content;
          });
        },
        error => {
          console.log(error);
          this.connected = false;
        }
      );
    },
    disconnect() {
      if (this.stompClient) {
        this.stompClient.disconnect();
      }
      this.connected = false;
    },
    tickleConnection() {
      this.connected ? this.disconnect() : this.connect();
    },
    donwloadAsFbx() {
      var element = document.createElement('a');
      element.setAttribute('href', 'data:text/plain;charset=utf-8,' + this.received_messages[0]);
      element.setAttribute('download', "download.fbx");

      element.style.display = 'none';
      document.body.appendChild(element);

      element.click();

      document.body.removeChild(element);
    }   
  },
  mounted() {
    // this.connect();
  }
};
</script>

<style scoped>
</style>