<template>
  <div class="field">
    <div class="file is-centered is-primary has-name is-boxed my-4">
      <label class="file-label">
        <input
          class="file-input"
          type="file"
          name="textfile"
          accept=".txt, .json"
          @change="onFileSelected"
        />
        <span class="file-cta">
          <span class="file-icon">
            <font-awesome-icon icon="file-alt" />
          </span>
          <span class="file-label">
            Select file to start annotating
          </span>
        </span>
      </label>
    </div>
  </div>
</template>

<script>
import { mapMutations } from "vuex";

export default {
  name: "LoadTextFile",
  emits: ["text-file-loaded", "json-file-loaded"],
  methods: {
    ...mapMutations(["setInputSentences"]),
    // onFileSelected() is called if the user clicks and manually
    //    selects a file. If they drag and drop, that is handled in
    //    App.vue. If you modify this function, you may also want to 
    //    modify App#onDrop(), App#processFileDrop(), and
    //    StartPage#onFileSelected() to match
    onFileSelected(files) {
      let fs = files.target.files;
      if (!fs.length || fs.length > 1) {
        console.log("Either 0 or >1 file was uploaded, returning.");
        return;
      }
      file = fs[0]

      let fileType = file.name.split('.').pop();
        // If it is a text file, enter annotation mode
      if (fileType === "txt") {
        let reader = new FileReader();
        reader.addEventListener("load", (event) => {
          this.setInputSentences(event.target.result);
          this.$emit("text-file-loaded");
        });
        reader.readAsText(file);
      }    
      // If it is a json file, enter review mode
      else if (fileType === "json") {
        this.loadAnnotations(JSON.parse(file));
        this.notify(
          "fa fa-check",
          `${this.classes.length} tags imported successfully\n, annotations imported successfully`,
          "positive"
        );
        this.$emit("json-file-loaded")
      }
      else {
        alert('Please upload either a .txt or a .json file.');
      }
    },
  },
};
</script>