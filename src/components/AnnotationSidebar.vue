<template>
  <nav class="px-3">
    <split-type-selector class="mt-4 mb-5" />
    <annotation-precision-selector class="mt-4 mb-5" />
    <progress-bar
      class="mb-5"
      :completed="currentIndex"
      :total="inputSentences.length"
    />
    <keyboard-shortcuts />
    <show-rejected />
  </nav>
</template>

<script>
import { mapState } from "vuex";
import ProgressBar from "./sidebar/ProgressBar.vue";
import SplitTypeSelector from "./sidebar/SplitTypeSelector.vue";
import AnnotationPrecisionSelector from "./sidebar/AnnotationPrecisionSelector.vue";
import KeyboardShortcuts from "./sidebar/KeyboardShortcuts.vue";
import ShowRejected from "./sidebar/ShowRejected.vue";

export default {
  name: "AnnotationSidebar",
  components: {
    ProgressBar,
    SplitTypeSelector,
    AnnotationPrecisionSelector,
    KeyboardShortcuts,
    ShowRejected,
  },
  computed: {
    ...mapState(["currentIndex", "inputSentences"]),
  },
};
</script>

<style lang="scss">
.is-single-line {
  width: 90%;
  text-overflow: ellipsis;
  white-space: nowrap;
  overflow: hidden;
}
</style>
