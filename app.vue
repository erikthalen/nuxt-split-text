<script setup>
const text = `<p class="big">We work in many different ways depending on the client, project and budget. We work in teams and try to involve everybody all the way.</p><p>We like projects to be short and intense even if it’s not always a possibility. </p><p class="small">We have a sprint based process framework called “The Involver” which we apply on all projects.</p>`

const progress = ref(0)
</script>

<template>
  <div>
    <input type="range" min="0" max="1" step="0.0001" v-model="progress" />

    <p>{{ progress }}</p>

    <rich-text :text="text" :progress="progress" />
  </div>
</template>

<style>
input {
  margin-bottom: 30px;
  position: relative;
  z-index: 1;
}

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
