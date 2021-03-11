<template>
  <div>
    <h1>Vue i18n Translator</h1>
    <Reader
      v-if="source.lang == 'no source'"
      ButtonLabel="Source"
      @load="source = $event"
    ></Reader>
    <span v-else>{{ source.lang }}&nbsp;&#8680;</span>
    <Reader
      v-if="target.lang == 'no target'"
      ButtonLabel="Target"
      @load="target = $event"
    ></Reader>
    <span v-else>&nbsp;{{ target.lang }}</span>
    <TranslationTable
      v-if="source.lang != 'no source' && target.lang != 'no target'"
      :Source="source"
      :Target="target"
    />
  </div>
</template>

<script>
import Reader from "./Reader.vue";

import TranslationTable from "./TranslationTable.vue";
export default {
  name: "Translator",
  data() {
    return {
      source: {
        lang: "no source",
        data: {}
      },
      target: {
        lang: "no target",
        data: {}
      }
    };
  },
  components: {
    Reader,
    TranslationTable
  },

  computed: {}
};
</script>
