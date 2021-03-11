<template>
  <div>
    <label class="text-reader">
      Read {{ ButtonLabel }}
      <input type="file" @change="loadTextFromFile" />
    </label>
  </div>
</template>

<script>
export default {
  props: ["ButtonLabel"],
  methods: {
    loadTextFromFile(ev) {
      const file = ev.target.files[0];
      const reader = new FileReader();

      reader.onload = e => {
        const langData = {
          lang: file.name.split(".")[0],
          data: JSON.parse(e.target.result)
        };
        this.$emit("load", langData);
      };
      reader.readAsText(file);
    }
  }
};
</script>

<style>
.text-reader {
  position: relative;
  overflow: hidden;
  display: inline-block;

  /* Fancy button style 😎 */
  border: 2px solid black;
  border-radius: 5px;
  padding: 8px 12px;
  color: #fff;
  background-color: #17a2b8;
  cursor: pointer;
}
.text-reader input {
  position: absolute;
  top: 0;
  left: 0;
  z-index: -1;
  opacity: 0;
}
</style>
