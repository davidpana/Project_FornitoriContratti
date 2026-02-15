<template>
  <div id="app">
    <PdfUploader />
  </div>
</template>

<script>
import PdfUploader from './components/PdfUploader.vue';

export default {
  name: 'App',
  components: {
    PdfUploader
  }
};
</script>

<style>
body {
  font-family: Arial, sans-serif;
  background: #f0f2f5;
}
</style>
