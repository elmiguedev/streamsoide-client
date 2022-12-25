<template>
  <div class="container-fluid h-100">
    <div class="row h-100 w-100">
      <div class="col h-100 w-100" style="transform: scale(0.5)">
        <scene />
      </div>
      <div class="col">
        <div class="row">
          <div class="col">
            <a :href="getUrl()">link nuevo</a>
            <button @click="twitchTest()">Twitch test</button>
            <button @click="youtubeSignin()">Youtube login</button>
            <button @click="youtubeTest()">Youtube test</button>
          </div>
        </div>
        <div class="row">
          <div class="col">
            <h3>Comments</h3>
            <hr />
          </div>
        </div>
        <div
          v-for="comment in comments"
          :key="comment.message"
          class="row"
          style="cursor: pointer"
        >
          <div
            class="col"
            :class="{ 'bg-primary': currentComment === comment }"
          >
            <strong @click="showComment(comment)">{{ comment.user }}: </strong>
            <div v-html="comment.message"></div>
            <span
              v-if="currentComment === comment"
              style="padding: 8px"
              @click="clearComment()"
              >X</span
            >
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import tmi from "tmi.js";
import io from "socket.io-client";
import Scene from "./Scene.vue";

const serverUrl = "ws://localhost:5001";
const socket = io(serverUrl);

export default {
  components: {
    Scene,
  },
  data() {
    return {
      twitchLink:
        "https://id.twitch.tv/oauth2/authorize?client_id=glthmr9u22z1ugtcvrbwxzaoam4veb&redirect_uri=http://localhost:4000&response_type=code&scope=chat:read+chat:edit",
      comments: [],
      currentComment: null,
    };
  },
  methods: {
    showComment(comment) {
      this.currentComment = comment;
      socket.emit("comment:show", comment);
    },
    clearComment() {
      this.currentComment = null;
      socket.emit("comment:clear");
    },
    getUrl() {
      const response_type = "token";
      const client_id = "glthmr9u22z1ugtcvrbwxzaoam4veb";
      const redirect_url = "http://localhost:4000";
      const scope = "channel%3Amanage%3Apolls+channel%3Aread%3Apolls";
      const url = `https://id.twitch.tv/oauth2/authorize?response_type=${response_type}&client_id=${client_id}&redirect_uri=${redirect_url}&scope=${scope}`;
      return url;
    },
    twitchTest() {
      console.log("Holis");

      const client = new tmi.Client({
        channels: ["midudev"],
      });

      client.connect();

      client.on("message", (channel, tags, message, self) => {
        this.comments.push({
          user: tags["display-name"],
          message: this.parseMessage(tags, message),
        });
      });
    },
    checkYoutube() {
      const YT_SCOPE = "https://www.googleapis.com/auth/youtube.force-ssl";
      const paramsString = window.location.hash.replace("#", "?");
      const params = new URLSearchParams(paramsString);

      console.log("Scope:", params.get("scope"));
      console.log("Token:", params.get("access_token"));

      if (params.has("scope") && params.get("scope") === YT_SCOPE) {
        console.log("PASA");
        localStorage.setItem("yt-token", params.get("access_token"));
        window.location = window.location.pathname;
      }
    },
    async youtubeTest() {
      let nextToken = "";
      const channelId = "iMavLi9kEkI";
      const streamUrl = `https://www.googleapis.com/youtube/v3/videos?part=liveStreamingDetails,snippet&id=${channelId}`;
      const streamResponse = await (
        await fetch(streamUrl, {
          headers: {
            Authorization: "Bearer " + localStorage.getItem("yt-token"),
          },
        })
      ).json();

      const liveId =
        streamResponse.items[0].liveStreamingDetails.activeLiveChatId;
      const maxResults = 1;
      const chatUrl = `https://www.googleapis.com/youtube/v3/liveChat/messages?liveChatId=${liveId}&part=snippet,authorDetails&maxResults=${maxResults}&pageToken=${nextToken}`;

      setInterval(async () => {
        const chatResponse = await (
          await fetch(chatUrl, {
            headers: {
              Authorization: "Bearer " + localStorage.getItem("yt-token"),
            },
          })
        ).json();
        nextToken = chatResponse.nextPageToken;
        // console.log(chatResponse);
        const messages = chatResponse.items.map((item) => ({
          message: item.snippet.displayMessage,
          user: item.authorDetails.displayName,
        }));
        console.log("LOS MENSAJES:", messages);
      }, 3000);
    },
    parseMessage(tags, message) {
      if (tags.emotes) {
        let parsedMessage = message;
        Object.keys(tags.emotes).forEach((emote) => {
          const imgUrl = `https://static-cdn.jtvnw.net/emoticons/v1/${emote}/2.0`;
          const imgTag = `<img width="18" src="${imgUrl}"/>`;
          const emotePosition = tags.emotes[emote][0];
          const from = emotePosition.split("-")[0];
          const to = +emotePosition.split("-")[1];
          const emoteName = message.substring(from, to + 1);
          parsedMessage = parsedMessage.replaceAll(emoteName, imgTag);
        });

        return parsedMessage;
      } else {
        return message;
      }
    },
    replaceRange(s, start, end, substitute) {
      return s.substring(0, start) + substitute + s.substring(end);
    },
    youtubeSignin() {
      /*
       * Create form to request access token from Google's OAuth 2.0 server.
       */
      // Google's OAuth 2.0 endpoint for requesting an access token
      var oauth2Endpoint = "https://accounts.google.com/o/oauth2/v2/auth";

      // Create <form> element to submit parameters to OAuth 2.0 endpoint.
      var form = document.createElement("form");
      form.setAttribute("method", "GET"); // Send as a GET request.
      form.setAttribute("action", oauth2Endpoint);

      // Parameters to pass to OAuth 2.0 endpoint.
      var params = {
        client_id:
          "706677339858-m8b2ebg91slagb3sd93bcof8n3m5lalb.apps.googleusercontent.com",
        redirect_uri: "http://localhost:4000",
        response_type: "token",
        scope: "https://www.googleapis.com/auth/youtube.force-ssl",
        include_granted_scopes: "true",
        state: "pass-through value",
      };

      // Add form parameters as hidden input values.
      for (var p in params) {
        var input = document.createElement("input");
        input.setAttribute("type", "hidden");
        input.setAttribute("name", p);
        input.setAttribute("value", params[p]);
        form.appendChild(input);
      }

      // Add form to page and submit it to open the OAuth 2.0 endpoint.
      document.body.appendChild(form);
      form.submit();
    },
  },
  mounted() {
    this.checkYoutube();
  },
};
</script>

<style>
</style>