<template>
  <div
    class="fullscreen"
    @dragover.prevent="onDragEnter"
    @dragenter="onDragEnter"
    @dragleave.self="onDragLeave"
    @drop.stop.prevent="onDrop"
    style="overflow-y:scroll;"
  >
    <div :style="{'pointer-events': overlayActive ? 'none' : 'auto'}">
      <q-layout view="hHh lpR fFf">
        <menu-bar v-if="currentPage !== 'start'" />

        <q-drawer :model-value="currentPage !== 'start'" bordered :class="$q.dark.isActive ? 'bg-dark' : 'bg-grey-2'">
          <annotation-sidebar />
        </q-drawer>

        <q-page-container>
          <start-page
            v-if="currentPage === 'start'"
            @text-file-loaded="switchToPage('annotate')"
            @json-file-loaded="switchToPage('review')"
          />
          <annotation-page v-if="currentPage === 'annotate'" />
          <review-page v-if="currentPage === 'review'" />
        </q-page-container>
      </q-layout>
      <drag-n-drop-overlay :style="{'visibility': overlayActive && pendingFileDrop == null ? 'visible' : 'hidden'}"/>
      <exit-dialog :show="pendingFileDrop != null && currentPage != 'start'" @hide="pendingFileDrop = null" @confirm="processFileDrop()"/>
    </div>
  </div>
</template>

<script>
import MenuBar from "./components/menubar/MenuBar.vue";
import StartPage from "./components/StartPage.vue";
import AnnotationPage from "./components/AnnotationPage.vue";
import ReviewPage from "./components/ReviewPage.vue";
import AnnotationSidebar from "./components/AnnotationSidebar.vue";
import DragNDropOverlay from "./components/DragNDropOverlay.vue";
import ExitDialog from "./components/ExitDialog.vue";
import { mapState, mapMutations } from "vuex";
import { useQuasar } from "quasar";

export default {
  name: "LayoutDefault",
  setup() {
    const $q = useQuasar();
    return {
      notify(icon, message, level) {
        $q.notify({
          icon,
          message,
          color: level,
          position: "top",
          timeout: 2000,
          actions: [
            {
              label: "Dismiss",
              color: "white",
            },
          ],
        });
      },
    };
  },
  data() {
    return {
      // currentPage: "start", this is now global in index.js store, not local
      overlayActive: false,
      pendingFileDrop: null,
    };
  },
  components: {
    MenuBar,
    StartPage,
    AnnotationPage,
    ReviewPage,
    AnnotationSidebar,
    DragNDropOverlay,
    ExitDialog
  },
  computed: {
    ...mapState(["annotations", "classes", "currentPage"]),
  },
  methods: {
    ...mapMutations(["loadClasses", "loadAnnotations", "setInputSentences", "clearAllAnnotations", "resetIndex", "setCurrentPage"]),
    switchToPage(page) {
      // console.log("page in App.switchToPage (before):", this.currentPage)
      this.setCurrentPage(page);
      // console.log("page in App.switchToPage (after):", this.currentPage)
      // console.log("page in App.switchToPage (after):", $store.state.currentPage)
    },
    onDragEnter() {
      console.log("Here");
      this.overlayActive = true;
    },
    onDragLeave() {
      this.overlayActive = false;
    },
    onDrop(event) {
      console.log(event);
      this.overlayActive = false;
      this.pendingFileDrop = event.dataTransfer.files[0]
      // assumes single file drop. will not cause error if
      //    multiple are dropped, it will just select the first one
      let fileType = this.pendingFileDrop.name.split('.').pop();
      // console.log("FILE TYPE: ", fileType)
      if (fileType === 'txt' || fileType === 'json') {
        // I don't know why, but checking if the current page is start
        // will NOT act like you think. It will execute contents of the 'if'
        // statement no matter what page you are on. However, when I
        // deleted the check, it wouldn't warn the user and ask if they
        // were sure that they wanted to abandon their changes on
        // the current text. This is something to look into and fix.
        if (this.currentPage == "start") {
          this.processFileDrop();
        }
        // An else statement is not included here since other file
        // types are already handled in processFileDrop().
      }
    },
    // If you modify this function, you may also wish to modify
    // onFileSelected() in StartPage.vue.
    processFileDrop() {
      let fileType = this.pendingFileDrop.name.split('.').pop();
      let reader = new FileReader();
      reader.onload = (ev) => {
        let file = ev.target.result;
        // If it is a text file, enter annotation mode
        if (fileType === "txt") {
          try {
              this.setInputSentences(file);
              this.clearAllAnnotations();
              this.resetIndex();
              this.switchToPage('annotate');
            } catch (e) {
              this.notify("fas fa-exclamation-circle", "Invalid file", "red-6");
            }
        }
        // If it is a json file, enter review mode
        else if (fileType === "json") {
          console.log("Will enter review mode: not yet implemented")
          // TODO: implement logic here that would enter review mode.
          //    The code below imports the tags file.
          this.loadAnnotations(JSON.parse(file));
          this.switchToPage('review');
          this.notify(
            "fa fa-check",
            `${this.classes.length} tags imported successfully\n, Annotations imported successfully`,
            "positive"
          );
        }
        // If it is a different file type, raise an alert.
        else {
          alert('Please upload either a .txt or a .json file.');
        }
      };
      reader.readAsText(this.pendingFileDrop); // ? I haven't explored what this does
      this.pendingFileDrop = null;
    },
  },
};
</script>
