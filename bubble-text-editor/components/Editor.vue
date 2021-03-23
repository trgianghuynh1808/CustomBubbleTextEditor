<template>
  <div>
    <div
      class="editor medium-editor-container"
      v-html="prefill"
      ref="editor"
      v-class="editorClass"
    >
      This is editor
    </div>
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
  components: {},
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
        // this.$refs.editor.focus();
      }
      this.editor.subscribe("editableInput", this.triggerChange);
    },
    triggerChange() {
      // this.addClassToPre();
      const content = this.editor.getContent();
      setTimeout(() => {
        if (/<[a-z][\s\S]*>/i.test(content)) {
          this.hasContent = true;
        } else {
          this.hasContent = false;
        }
      }, 0);
      this.$emit("input", content);
      if (this.onChange) {
        this.onChange(content);
      }
    },
    destroyElm() {
      this.editor.destroy();
    }
  },
  destroyed() {
    this.destroyElm();
  }
};
</script>

<style>
@import "~/assets/css/medium-editor.css";
@import "~/assets/css/default.css";

.editor {
  outline: 0px solid transparent !important;
}
</style>
