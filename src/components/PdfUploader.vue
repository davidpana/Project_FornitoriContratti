<template>
  <div class="pdf-uploader">
    <h2>Carica PDF e invia all'API</h2>
    <input type="file" accept="application/pdf" @change="handleFileUpload" />
    <button :disabled="!pdfFile || uploading" @click="sendPdf">{{ uploading ? 'Invio...' : 'Invia PDF' }}</button>
    <div v-if="error" class="error">{{ error }}</div>
    <div v-if="serverResponse" class="response">
      <strong>Risposta API:</strong>
      <div v-if="serverResponse && typeof serverResponse === 'object'">
        <div v-if="serverResponse.message"><strong>message:</strong> {{ serverResponse.message }}</div>
        <div v-if="serverResponse.filename"><strong>filename:</strong> {{ serverResponse.filename }}</div>
        <div v-if="serverResponse.sha256"><strong>sha256:</strong> <code>{{ serverResponse.sha256 }}</code></div>
        <pre v-if="Object.keys(serverResponse).length > 0">{{ formattedResponse }}</pre>
      </div>
      <pre v-else>{{ formattedResponse }}</pre>
    </div>
  </div>
</template>

<script>

export default {
  name: 'PdfUploader',
  data() {
    return {
      pdfFile: null,
      // hash: '',
      error: '',
      serverResponse: null,
      uploading: false
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
      this.error = '';
      this.serverResponse = null;
      this.uploading = true;
      try {
        const formData = new FormData();
        // il backend si aspetta il campo 'pdf' (vedi curl di esempio)
        formData.append('pdf', this.pdfFile, this.pdfFile.name);

        const res = await fetch('http://localhost:3000/upload', {
          method: 'POST',
          headers: {
            'Accept': 'application/json'
            // NON impostare Content-Type quando invii FormData: il browser aggiunge il boundary automaticamente
          },
          body: formData
        });

        const contentType = res.headers.get('content-type') || '';
        let data;
        if (contentType.includes('application/json')) {
          data = await res.json();
        } else {
          data = await res.text();
        }

        if (!res.ok) {
          const errMsg = typeof data === 'string' ? data : JSON.stringify(data);
          this.error = `Server error (${res.status}): ${errMsg}`;
          console.error('Upload error:', res.status, data);
          this.serverResponse = data;
          return;
        }

        // risposta attesa: { message, filename, sha256 }
        console.log('API response:', data);
        this.serverResponse = data;
        this.error = '';

      } catch (e) {
        console.error(e);
        this.error = "Errore durante l'invio del PDF.";
      } finally {
        this.uploading = false;
      }
    }
    },
    computed: {
      formattedResponse() {
        return typeof this.serverResponse === 'object'
          ? JSON.stringify(this.serverResponse, null, 2)
          : String(this.serverResponse);
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
