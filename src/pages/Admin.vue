<template>
  <div class="container-fluid h-100">
    <div class="row h-100 w-100">
      <div class="col h-100 w-100" style="transform: scale(0.5)">
        <scene />
      </div>
      <div class="col">
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
            <strong @click="showComment(comment)">{{ comment.user }}: </strong
            ><span @click="showComment(comment)">{{ comment.message }}</span>
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
import io from "socket.io-client";
import Scene from "./Scene.vue";

const serverUrl = "ws://localhost:3000";
const socket = io(serverUrl);

export default {
  components: {
    Scene,
  },
  data() {
    return {
      comments: [
        { user: "elmiguedev", message: "Hola soy un comentario" },
        { user: "ebuto", message: "Eh que te pasa wacho" },
        { user: "choromeko", message: "Saludo pa los pibes" },
        { user: "messi", message: "Que mirás bobo" },
      ],
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
  },
};
</script>

<style>
</style>