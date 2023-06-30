<script setup>
const props = defineProps({
  text: String,
})
</script>

<template>
  <div class="rich-text" v-html="text" />
</template>

<style>
p {
  max-width: 20em;
  font-size: 26px;
  line-height: 1.4;
}

.big {
  font-size: 38px;
  line-height: 1.2;
}

.small {
  font-size: 16px;
  line-height: 1.6;
}
</style>
