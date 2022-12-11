<template>
  <div>
    el path: {{ JSON.stringify(currentPath) }}
    <component :is="currentView" />
  </div>
</template>

<script>
import Admin from "./pages/Admin.vue";
import Scene from "./pages/Scene.vue";

const routes = {
  "/": Admin,
  scene: Scene,
};

export default {
  data() {
    return {
      currentPath: window.location.pathname,
    };
  },
  computed: {
    currentView() {
      return routes[this.currentPath.slice(1) || "/"] || Admin;
    },
  },
  mounted() {
    window.addEventListener("hashchange", () => {
      this.currentPath = window.location.pathname;
    });
  },
};
</script>

<style>
</style>