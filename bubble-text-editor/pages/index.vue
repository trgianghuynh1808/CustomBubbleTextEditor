<template>
  <Editor :prefill="defaultContent" />
</template>

<script>
import Editor from "~/components/Editor";

export default {
  components: {
    Editor
  },
  data() {
    return {
      defaultContent: `<h2>Medium Editor</h2>`
    };
  }
};
</script>
