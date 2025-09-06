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
