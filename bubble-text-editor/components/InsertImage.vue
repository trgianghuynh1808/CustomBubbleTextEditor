<template>
  <file-upload
    ref="upload"
    extensions="gif,jpg,jpeg,png,webp"
    accept="image/png,image/gif,image/jpeg,image/webp"
    name="image"
    :multiple="true"
    v-model="insert.files"
    @input-filter="inputFilter"
    @input-file="inputFile"
  >
    <div class="upload-btn-wrp">
      <v-icon>
        mdi-image
      </v-icon>
    </div>
  </file-upload>
</template>

<script>
import VueUploadComponent from "vue-upload-component";
import axios from "axios";

export default {
  props: ["editor", "insert", "editorRef", "handler"],
  components: {
    // FontAwesomeIcon,
    "file-upload": VueUploadComponent
  },
  data() {
    return {
      currentLine: null,
      currentImg: null,
      currentSize: "is-normal",
      position: {
        top: "0"
      },
      isShow: false
    };
  },
  methods: {
    initializeExisting() {
      const handlerVm = this;
      setTimeout(() => {
        const focused = this.editor.getFocusedElement();
        if (!focused) return false;

        const editorImages = focused.getElementsByClassName("editor-image");
        for (const elm of editorImages) {
          // Set Onclick to Show Image Size Handler
          elm.onclick = function() {
            setTimeout(() => {
              handlerVm.sizingHandler(this);
            });
          };
        }
      });
    },
    addImage(url) {
      this.$emit("uploaded", url);
      if (this.insert.isToggle) {
        const handlerVm = this;
        this.editorRef.focus();
        this.editor.selectElement(this.insert.focusLine);
        this.editor.pasteHTML(
          `<div class="editor-image">
                        <img src="${url}" />
                    </div>
                    <div class="editor-image-description"><br></div>
                    <br />`,
          { cleanAttrs: [], cleanTags: [], unwrapTags: [] }
        );
        this.currentLine = this.editor.getSelectedParentElement().previousElementSibling.previousElementSibling;
        this.currentImg = this.currentLine.querySelector("img");
        const currentPos = this.currentImg.getBoundingClientRect();
        this.window.scrollTo(0, currentPos.top - currentPos.x);
        this.currentLine.onclick = function() {
          setTimeout(() => {
            handlerVm.sizingHandler(this);
          });
        };
        this.insert.isShow = false;
      }
    },
    sizingHandler(elm) {
      const className = elm.className;
      elm.className = className.replace("is-focus", "") + " is-focus";
      if (className.indexOf("expand") > -1) {
        this.currentSize = "is-expand";
      } else if (className.indexOf("full") > -1) {
        this.currentSize = "is-full";
      } else {
        this.currentSize = "is-normal";
      }
      const img = elm.querySelector("img");
      this.currentLine = elm;
      this.isShow = true;
      const currentPos = img.getBoundingClientRect();
      this.position.top = currentPos.top + "px";
      this.$emit("imageClick", {
        position: this.position,
        currentLine: this.currentLine,
        isShow: this.isShow,
        currentSize: this.currentSize
      });
    },
    inputFilter(newFile, oldFile, prevent) {
      if (newFile && !oldFile) {
        if (/(\/|^)(Thumbs\.db|desktop\.ini|\..+)$/.test(newFile.name)) {
          return prevent();
        }
        if (/\.(php5?|html?|jsx?)$/i.test(newFile.name)) {
          return prevent();
        }
      }
    },
    inputFile(newFile, oldFile) {
      if (newFile && !oldFile) {
        this.$refs.upload.active = true;
        this.uploadFileAPI(newFile.file);
      }
    },
    async uploadFileAPI(file) {
      const query = `
      mutation($file: Upload!){
        upload_file(file: $file)
      }`;

      let formData = new FormData();
      const operations = JSON.stringify({
        query,
        variables: { file: null }
      });
      formData.append("operations", operations);
      const map = {
        0: ["variables.file"]
      };
      formData.append("map", JSON.stringify(map));
      formData.append("0", file);

      const response = await axios({
        url: "http://localhost:4002/graphql",
        method: "post",
        data: formData
      });
      if (!response || !response.data) return;

      const { data } = response.data;
      const url = data.upload_file;
      this.addImage(url);
    }
  },
  mounted() {
    this.initializeExisting();
  },
  beforeMount() {
    this.window = window;
    window.addEventListener("scroll", this.handleScroll);
  },
  beforeDestroy() {
    window.removeEventListener("scroll", this.handleScroll);
  }
};
</script>
<style>
.upload-btn-wrp {
  margin-left: 10px;
}
</style>
