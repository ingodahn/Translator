<template>
  <div class="editor">
    <div class="editorSource">
      <i>{{ CurrentRow.group }}:</i>
      <br />
      {{ CurrentRow.sourceTerm }}
    </div>
    <div>
      <input class="editorInput" type="text" size="75" v-model="CurrentRow.targetTerm" />
      <input class="editorButton" type="button" value="Save All" v-on:click="saveAll" />
    </div>
  </div>
</template>

<script>
export default {
  name: "Editor",
  props: ["CurrentRow"],
  data() {
    return {
      langFile: "lang.json"
    };
  },
  methods: {
    saveAll: function() {
      this.$emit("saveAll");
    }
  }
};
</script>

<style scoped>
.editor {
  margin: auto;
  width: 75%;
}
.editorSource {
  text-align: left;
}
.editorButton {
  margin: 1em;
  color: #fff;
  background-color: #17a2b8;
}
</style>