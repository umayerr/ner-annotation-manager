<template>
  <section>
    <br />
    <q-toggle
      v-model="showRejected"
      label="View Rejected Annotations"
      @click="showRejectedChanged"
    />
  </section>
</template>
  
<script>
export default {
  name: "ShowRejected",
  data: function() {
    return {
      showRejected: true,
    };
  },
  methods: {
    showRejectedChanged() {
      this.$store.commit("setShowRejected", this.showRejected); 
    },
  },
};
</script>
  