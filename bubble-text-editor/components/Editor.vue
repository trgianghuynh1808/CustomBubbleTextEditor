<template>
  <div class="editor" v-html="prefill" ref="editor">This is editor</div>
</template>

<script>
import MediumEditor from "medium-editor";

export default {
  data() {
    return {
      editor: null,
      hasContent: false
    };
  },
  props: ["prefill"],
  mounted() {
    this.createElement();
  },
  methods: {
    createElement() {
      this.editor = new MediumEditor(this.$refs.editor);
      //check prefill exists
      if (this.prefill) {
        if (/<[a-z][\s\S]*>/i.test(this.prefill)) {
          this.hasContent = true;
        } else {
          this.hasContent = false;
        }
        this.$refs.editor.focus();
      }
    }
  }
};
</script>

<style>
@import "~/assets/css/medium-editor.css";
@import "~/assets/css/default.css";
</style>
