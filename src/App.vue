<template>
  <div id="app">
    <Header />
    <PdfUploader />
  </div>
</template>
<script>
import PdfUploader from './components/PdfUploader.vue';
import Header from './components/Header.vue';

export default {
  name: 'App',
  components: {
    PdfUploader,
    Header
  }
};
</script>

<style>
body {
  font-family: Arial, sans-serif;
  background: #f0f2f5;
}
</style>
