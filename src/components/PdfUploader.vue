<template>
  <div class="pdf-uploader">
    <h2>Carica PDF e invia all'API</h2>
    <input type="file" accept="application/pdf" @change="handleFileUpload" />
    <button :disabled="!pdfFile" @click="sendPdf">Invia PDF</button>
    <div v-if="error" class="error">{{ error }}</div>
  </div>
</template>

<script>

export default {
  name: 'PdfUploader',
  data() {
    return {
      pdfFile: null,
      // hash: '',
      error: ''
    };
  },
  methods: {
    handleFileUpload(event) {
      this.error = '';
      const file = event.target.files[0];
      if (file && file.type === 'application/pdf') {
        this.pdfFile = file;
      } else {
        this.error = 'Seleziona un file PDF valido.';
        this.pdfFile = null;
      }
    },
    async sendPdf() {
      if (!this.pdfFile) return;
      try {
        const formData = new FormData();
        formData.append('file', this.pdfFile);
        // Sostituisci l'URL con quello della tua API
        await fetch('URL_DELLA_TUA_API', {
          method: 'POST',
          body: formData
        });
      } catch (e) {
        this.error = "Errore durante l'invio del PDF.";
      }
    }
  }
};
</script>

<style scoped>
.pdf-uploader {
  max-width: 400px;
  margin: 2rem auto;
  padding: 2rem;
  border: 1px solid #ccc;
  border-radius: 8px;
  background: #fafafa;
}
.error {
  color: red;
  margin-top: 1rem;
}
</style>
