<template>
  <div class="image-handler-container">
    <div
      class="insert-image-container insert-embed-btn"
      v-show="insert.isShow"
      v-bind:style="insert.position"
    >
      <v-btn class="mx-2 " small fab color="secondary" dark @click="toggle">
        <v-icon dark>
          mdi-plus
        </v-icon>
      </v-btn>
      <div class="insert-image-menu" v-show="insert.isToggle">
        <insert-image
          :editor="editor"
          :insert="insert"
          :editorRef="editorRef"
          :handler="handler"
          v-on:uploaded="uploadCallback"
          v-on:imageClick="imageClickHandler"
          title="Insert Image"
        ></insert-image>
      </div>
    </div>
    <image-position
      :handler="handler"
      v-on:onPositionChange="onChange"
    ></image-position>
  </div>
</template>

<script>
import InsertImage from "./InsertImage";
import ImagePosition from "./ImagePosition";

export default {
  components: {
    InsertImage,
    ImagePosition
  },
  data() {
    return {
      insert: {
        position: {
          top: "0",
          left: "0"
        },
        isShow: false,
        isToggle: false,
        embedElm: null,
        files: [],
        focusLine: null
      },
      handler: {
        currentLine: null,
        currentImg: null,
        currentSize: "is-normal",
        position: {
          top: "0"
        },
        isShow: false
      }
    };
  },
  props: ["editor", "editorRef", "onChange"],
  methods: {
    subscribe() {
      this.editor.subscribe("editableKeyup", this.detectShowToggle);
      this.editor.subscribe("editableClick", this.detectShowToggle);
      this.editor.subscribe("editableKeyup", this.detectImageDescription);
      this.editor.subscribe("editableClick", this.detectImageDescription);
    },
    unsubscribe() {
      this.editor.unsubscribe("editableKeyup", this.detectShowToggle);
      this.editor.unsubscribe("editableClick", this.detectShowToggle);
      this.editor.unsubscribe("editableKeyup", this.detectImageDescription);
      this.editor.unsubscribe("editableClick", this.detectImageDescription);
    },
    detectImageDescription() {
      const focused = this.editor.getFocusedElement();
      if (!focused) return;

      const editorImages = focused.getElementsByClassName(
        "editor-image-description"
      );
      for (const elm of editorImages) {
        const description = elm.innerHTML.trim();
        if (!description || description == "<br>") {
          elm.className = "editor-image-description is-empty";
        } else {
          elm.className = "editor-image-description";
        }
      }
    },
    detectShowToggle(e) {
      if (this.insert.isShow && this.insert.isToggle) {
        this.toggle();
      }
      if (e.keyCode == 13) {
        const focused = this.editor.getSelectedParentElement();
        const nextElm = focused.nextElementSibling;
        const prevElm = focused.previousElementSibling;
        if (
          nextElm &&
          prevElm &&
          nextElm.className.indexOf("editor-image-description") > -1 &&
          prevElm.className.indexOf("editor-image") > -1
        ) {
          nextElm.parentNode.insertBefore(nextElm, focused);
        }
      }
      this.handler.isShow = false;
      if (e.target.className.indexOf("editor-image-description") <= -1) {
        const editorImages = this.editor
          .getFocusedElement()
          .getElementsByClassName("editor-image");
        for (const imgElm of editorImages) {
          imgElm.className = imgElm.className.replace("is-focus", "");
        }
      }
      const currentLine = this.editor.getSelectedParentElement();
      const outerHtml = currentLine.outerHTML;
      const isList = outerHtml.indexOf("<li>") > -1;
      const content = currentLine.innerHTML
        .replace(/^(<br\s*\/?>)+/, "")
        .trim();
      if (content || isList) {
        // Not show toggle if focus line has content & list
        this.insert.isShow = false;
        this.insert.isToggle = false;
        this.insert.focusLine = null;
      } else {
        const currentPos = currentLine.getBoundingClientRect();
        this.insert.position.top = currentPos.top + "px";
        this.insert.position.left = currentPos.left - 20 + "px";
        this.insert.isShow = true;
        this.insert.focusLine = currentLine;
      }
    },
    toggle() {
      this.insert.isToggle = !this.insert.isToggle;
    },
    imageClickHandler(value) {
      this.handler = value;
    },
    uploadCallback(url) {
      this.$emit("uploaded", url);
    }
  },
  mounted() {
    this.subscribe();
  },
  destroyed() {
    this.unsubscribe();
  },
  beforeMount() {
    this.window = window;
  }
};
</script>

<style>
.insert-embed-btn {
  z-index: 999;
}
</style>
