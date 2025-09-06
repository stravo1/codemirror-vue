<script setup lang="ts">
import { ref } from "vue";
import CodeMirror from "./components/CodeMirror.vue";
const type = ref<"python" | "js" | "html" | "css" | "json">("python");
const editor = ref(null);

const getState = () => {
  if (editor.value) {
    console.log(editor.value.view.state.doc.toString());
  }
};
</script>

<template>
  <CodeMirror :type ref="editor" />
  <button @click="getState">Get State</button>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
