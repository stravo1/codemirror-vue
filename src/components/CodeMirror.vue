<script setup lang="ts">
import { ref, onMounted, watch } from "vue";
import { EditorState } from "@codemirror/state";
import { EditorView, basicSetup } from "codemirror";
import { keymap, lineNumbers } from "@codemirror/view";
import { defaultKeymap, indentWithTab } from "@codemirror/commands";
import {
  autocompletion,
  completionKeymap,
  closeBrackets,
} from "@codemirror/autocomplete";
import { oneDark } from "@codemirror/theme-one-dark";
import jsCompletionsFromGlobalScope from "../utils/globalCompletion";
import customPythonCompletions from "../utils/pythonCustomCompletion";

import { python, pythonLanguage } from "@codemirror/lang-python";
import { javascript, javascriptLanguage } from "@codemirror/lang-javascript";
import { html } from "@codemirror/lang-html";
import { css } from "@codemirror/lang-css";
import { json } from "@codemirror/lang-json";

const props = defineProps<{
  type: "python" | "js" | "html" | "css" | "json";
}>();
const editorContainer = ref<HTMLDivElement | null>(null);
const view = ref<EditorView | null>(null);
const pythonCompletions = ref({});

defineExpose({
  view,
});

const getPythonCompletions = async () => {
  try {
    let res = await fetch(
      "http://site.test:8009/api/v2/method/builder.api.get_codemirror_completions",
    );
    let data = (await res.json()).data;
    return data;
  } catch (err) {
    console.log("error: ", err);
  }
};

const createStartingState = async () => {
  pythonCompletions.value =
    props.type == "python" ? await getPythonCompletions() : {};
  const extensions = [
    basicSetup,
    keymap.of(defaultKeymap),
    keymap.of([indentWithTab]),
    keymap.of([
      {
        key: "Ctrl-s",
        mac: "Cmd-s",
        run: () => {
          console.log("Ctrl-s");
          return true;
        },
      },
    ]),
    lineNumbers(),
    autocompletion({
      activateOnTyping: true,
    }),
    closeBrackets(),
    oneDark,
    keymap.of([...defaultKeymap, ...completionKeymap]),
  ];

  switch (props.type) {
    case "js":
      extensions.push(
        javascript({
          jsx: false,
          typescript: false,
        }),
        javascriptLanguage.data.of({
          autocomplete: jsCompletionsFromGlobalScope,
        }),
      );
      break;
    case "python":
      extensions.push(
        python(),
        pythonLanguage.data.of({
          autocomplete: (context: any) =>
            customPythonCompletions(context, pythonCompletions.value),
        }),
      );
      break;
    case "html":
      extensions.push(html());
      break;
    case "css":
      extensions.push(css());
      break;
    case "json":
      extensions.push(json());
      break;
  }

  let startState = EditorState.create({
    doc: "",
    extensions,
  });
  return startState;
};

watch(
  () => props.type,
  async (newType) => {
    console.log(newType);
    view.value?.destroy();

    if (editorContainer.value) {
      view.value = new EditorView({
        state: await createStartingState(),
        parent: editorContainer.value,
      });
    }
  },
);

onMounted(async () => {
  if (editorContainer.value) {
    view.value = new EditorView({
      state: await createStartingState(),
      parent: editorContainer.value,
    });
  }
});
</script>

<template>
  <div
    class="code-mirror-editor"
    ref="editorContainer"
    style="height: 50vh"
  ></div>
</template>

<style>
.cm-editor {
  height: 100%;
  width: 100%;
}
</style>
