<template>
  <div>
    <input
      type="button"
      class="btn btn-info"
      value="Clear Filter"
      @click="clearSelector()"
    />
    <input
      type="button"
      class="btn btn-info"
      value="Filter Source"
      @click="filterBy('source')"
    />
    <input type="text" v-model="selector" />
    <input
      type="button"
      class="btn btn-info"
      value="Filter Target"
      @click="filterBy('target')"
    />
    <input
      type="button"
      class="btn btn-info"
      value="Yet to Translate"
      @click="emptyTarget()"
    />
  </div>
</template>

<script>
export default {
  name: "Selector",
  props: ["Select"],
  data() {
    return {
      selector: "",
      selectBy: "target"
    };
  },
  methods: {
    clearSelector: function() {
      this.Select.selectBy = "source";
      this.Select.expr = new RegExp(".*");
      this.selectBy = "source";
      this.selector = "";
      this.$emit("change-page");
    },
    filterBy: function(type) {
      this.selectBy = type;
      this.Select.selectBy = type;
      this.Select.expr = new RegExp(this.selector, "i");
      this.$emit("change-page");
    },
    emptyTarget: function() {
      this.Select.selectBy = "target";
      this.selectBy = "target";
      this.selector = "^$";
      this.Select.expr = new RegExp("^$");
      this.$emit("change-page");
    }
  }
};
</script>

<style scoped>
.btn-info {
  margin: 1em;
}
</style>
