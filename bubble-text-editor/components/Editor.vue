<template>
  <div
    class="editor medium-editor-container"
    v-html="prefill"
    ref="editor"
    v-class="editorClass"
  >
    This is editor
  </div>
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
  props: ["prefill", "options"],
  mounted() {
    this.createElement();
  },
  computed: {
    editorClass() {
      return {
        "has-content": this.hasContent
      };
    }
  },
  methods: {
    createElement() {
      this.editor = new MediumEditor(this.$refs.editor, this.options);
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
