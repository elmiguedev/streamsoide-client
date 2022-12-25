<template>
  <div style="padding: 0; margin: 0; width: 100%; height: 100%; display: flex">
    <div v-if="comment" style="width: 50%; margin-top: auto">
      <p style="background: white; padding: 16px">
        <strong style="background: blue; color: white; padding: 8px"
          >{{ comment.user }}:
        </strong>
        <span v-html="comment.message" style="padding: 8px"></span>
      </p>
    </div>
  </div>
</template>

<script>
import io from "socket.io-client";

const serverUrl = "ws://localhost:5001";
const socket = io(serverUrl);

export default {
  data() {
    return {
      comment: null,
    };
  },
  mounted() {
    socket.on("comment:show", (comment) => {
      this.comment = comment;
    });
    socket.on("comment:clear", () => {
      this.comment = null;
    });
  },
};
</script>

<style>
</style>