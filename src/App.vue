<template>
  <v-app id="inspire">
    <v-app-bar app clipped-right flat height="44">
      <v-spacer></v-spacer>
      <v-responsive max-width="56">
        <v-text-field
          dense
          flat
          hide-details
          rounded
          solo-inverted
        ></v-text-field>
      </v-responsive>
    </v-app-bar>
    <v-navigation-drawer
      v-model="drawer"
      app
      class="pt-2 pl-2"
      width="128"
      style="background-color: rgb(13, 17, 23)"
    >
      <v-list shaped>
        <v-list-item
          v-for="n in users"
          :key="n"
          link
          style="background-color: rgb(13, 17, 23)"
        >
          <v-list-item-content>
            <v-list-item-title style="color: rgb(88, 166, 255)">{{
              n
            }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-navigation-drawer
      app
      clipped
      right
      style="background-color: rgb(13, 17, 23)"
    >
      <v-list>
        <v-list-item
          v-for="n in rooms"
          :key="n"
          link
          style="background-color: rgb(13, 17, 23)"
        >
          <v-list-item-content>
            <v-list-item-title> {{ n }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-navigation-drawer>

    <v-main>
      <v-list v-if="msgs.length > 0">
        <v-list-item
          v-for="(msg,k) in msgs"
          :key="k"
          link
          style="background-color: rgb(13, 17, 23)"
        >
          <v-list-item-content>
            <v-list-item-title>
              {{ msg.Sender + ":" + msg.Content }}</v-list-item-title
            >
          </v-list-item-content>
        </v-list-item>
      </v-list>
    </v-main>

    <v-footer app color="transparent" height="128" inset>
      <v-text-field
        v-model="msg"
        clearable
        :append-outer-icon="msg ? 'mdi-send-circle-outline' : ''"
        @click:append-outer="sendMessage"
        background-color="grey lighten-1"
        title="发送消息"
        dense
        flat
        hide-details
        rounded
        solo
      >
      </v-text-field>
    </v-footer>
  </v-app>
</template>

<script>
import config from "./config/config";
import typ from "./config/message";

export default {
  data: () => ({
    master:"处理",
    drawer: null,
    wx: null,
    msg: "",
    rooms: [],
    users: [],
    msgs: [],
  }),
  methods: {
    sendJoinMsg() {
      this.ws.send(
        JSON.stringify({
          Type: 1,
          Content: this.master,
        })
      );
    },
    sendTextMsg(msg) {
      this.ws.send(msg);
    },
    sendMessage() {
      let msg = {
        Type: 3,
        Sender: this.master,
        Content: this.msg,
      };
      this.ws.send(JSON.stringify(msg));
      this.msgs.push(msg);
      this.msg = "";
    },
  },
  created() {
    const ws = new WebSocket(config.wsUrl + "w1");
    this.ws = ws;
    let that = this;
    ws.onopen = function () {
      console.log("Connection open ...");
      that.sendJoinMsg();
    };

    ws.onmessage = function (evt) {
      console.log("Received Message: " + evt.data);
      let data = JSON.parse(evt.data);
      if (data.Type == typ.RoomMessage) {
        that.rooms = JSON.parse(data.Content);
        return;
      }
      if (data.Type == typ.UserMessage) {
        that.users = JSON.parse(data.Content);
        return;
      }
      console.log(data);
      if (data.Type == typ.TextMessage) {
        that.msgs.push(data);
        return;
      }
      // ws.close();
    };

    ws.onclose = function () {
      console.log("Connection closed.");
    };
  },
};
</script>

<style scoped>
</style>