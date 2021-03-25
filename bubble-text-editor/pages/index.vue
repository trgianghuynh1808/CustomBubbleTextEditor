<template>
  <Editor :options="options" :prefill="defaultContent" v-model="content" />
</template>

<script>
import Editor from "~/components/Editor";

export default {
  components: {
    Editor
  },
  data() {
    return {
      content: ``,
      defaultContent: `<h1><b>Bubble Editor</b> - 9RS Team </h1>`,
      options: {
        forcePlainText: false,
        placeholder: {
          text: "Write something great!!"
        },
        toolbar: {
          buttons: [
            "bold",
            "italic",
            {
              name: "anchor",
              action: "createLink",
              aria: "link",
              tagNames: ["a"],
              contentDefault: "<b>🔗</b>"
            },
            "underline",
            "quote",
            "h1",
            "h2",
            "h3",
            "h4",
            "justifyCenter",
            "justifyLeft",
            "justifyRight",
            // {
            //   name: "pre",
            //   action: "append-pre",
            //   aria: "code highlight",
            //   tagNames: ["pre"],
            //   contentDefault: "<b><\\></b>",
            //   contentFA: '<i class="fa fa-code fa-lg"></i>'
            // },
            "unorderedlist",
            "orderedlist"
          ]
        }
      }
    };
  },
  watch: {
    content(newVal) {
      console.log({ content: newVal });
    }
  }
};
</script>
