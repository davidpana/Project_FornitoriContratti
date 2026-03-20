<template>
  <section id="upload" class="upload-shell">
    <div class="upload-card">
      <h2 class="title">Upload PDF and send to API</h2>
      <p class="subtitle">Send the contract to the backend and immediately view `message`, `filename`, and `sha256`.</p>

      <label class="file-label">
        <span>{{ pdfFile ? pdfFile.name : 'Select a PDF file' }}</span>
        <input type="file" accept="application/pdf" @change="handleFileUpload" />
      </label>

      <button class="send-btn" :disabled="!pdfFile || uploading" @click="sendPdf">
        {{ uploading ? 'Uploading...' : 'Send PDF' }}
      </button>

      <button v-if="uploadSuccess" class="notarize-btn" :disabled="notarizing" @click="notarizeAndCheck">
        {{ notarizing ? 'Processing...' : 'Notarization and Check' }}
      </button>

    <div v-if="error" class="error">{{ error }}</div>
    <div v-if="notarizationResult" class="notarization-result" :class="notarizationResult.verified ? 'verified' : 'not-verified'">
      <strong>{{ notarizationResult.verified ? 'Verified successfully' : 'Verification failed' }}</strong>
      <pre>{{ JSON.stringify(notarizationResult.data, null, 2) }}</pre>
    </div>
    <div v-if="serverResponse" class="response">
      <strong class="response-title">API response:</strong>
      <div v-if="serverResponse && typeof serverResponse === 'object'">
        <div v-if="serverResponse.message"><strong>message:</strong> {{ serverResponse.message }}</div>
        <div v-if="serverResponse.filename"><strong>filename:</strong> {{ serverResponse.filename }}</div>
        <div v-if="serverResponse.sha256"><strong>sha256:</strong> <code class="hash">{{ serverResponse.sha256 }}</code></div>
        <pre v-if="Object.keys(serverResponse).length > 0">{{ formattedResponse }}</pre>
      </div>
      <pre v-else>{{ formattedResponse }}</pre>
    </div>
    </div>
  </section>
</template>

<script>

const API_BASE_URL = (process.env.VUE_APP_API_BASE_URL || 'http://localhost:3000').replace(/\/$/, '');

export default {
  name: 'PdfUploader',
  data() {
    return {
      pdfFile: null,
      // hash: '',
      error: '',
      serverResponse: null,
      uploading: false,
      uploadSuccess: false,
      notarizing: false,
      notarizationResult: null
    };
  },
  computed: {
    formattedResponse() {
      return typeof this.serverResponse === 'object'
        ? JSON.stringify(this.serverResponse, null, 2)
        : String(this.serverResponse);
    }
  },
  methods: {
    handleFileUpload(event) {
      this.error = '';
      this.uploadSuccess = false;
      this.notarizationResult = null;
      const file = event.target.files[0];
      const isPdf = file && (file.type === 'application/pdf' || /\.pdf$/i.test(file.name || ''));
      if (isPdf) {
        this.pdfFile = file;
      } else {
        this.error = 'Please select a valid PDF file.';
        this.pdfFile = null;
      }
    },
    async sendPdf() {
      if (!this.pdfFile) return;
      this.error = '';
      this.serverResponse = null;
      this.uploadSuccess = false;
      this.notarizationResult = null;
      this.uploading = true;
      try {
        const formData = new FormData();
        // il backend si aspetta il campo 'pdf' (vedi curl di esempio)
        formData.append('pdf', this.pdfFile, this.pdfFile.name);

        const res = await fetch(`${API_BASE_URL}/upload`, {
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
        this.uploadSuccess = true;
        this.error = '';

      } catch (e) {
        console.error(e);
        this.error = 'Error while sending the PDF.';
      } finally {
        this.uploading = false;
      }
    },
    async notarizeAndCheck() {
      this.notarizing = true;
      this.notarizationResult = null;
      this.error = '';
      try {
        // Step 1: notarize
        const notarizeRes = await fetch(`${API_BASE_URL}/api/notarize`, {
          method: 'POST',
          headers: { 'Accept': 'application/json', 'Content-Type': 'application/json' },
          body: JSON.stringify({ hash: this.serverResponse && this.serverResponse.sha256 })
        });
        const notarizeContentType = notarizeRes.headers.get('content-type') || '';
        const notarizeData = notarizeContentType.includes('application/json')
          ? await notarizeRes.json()
          : await notarizeRes.text();

        if (!notarizeRes.ok) {
          this.error = `Notarization error (${notarizeRes.status}): ${typeof notarizeData === 'string' ? notarizeData : JSON.stringify(notarizeData)}`;
          return;
        }

        const notarizationId = notarizeData.notarizationId || notarizeData.id || notarizeData;
        if (!notarizationId) {
          this.error = 'The notarization response does not contain an ID.';
          return;
        }

        // Step 2: verify
        const verifyRes = await fetch(`${API_BASE_URL}/api/verify/${encodeURIComponent(notarizationId)}`, {
          method: 'GET',
          headers: { 'Accept': 'application/json' }
        });
        const verifyContentType = verifyRes.headers.get('content-type') || '';
        const verifyData = verifyContentType.includes('application/json')
          ? await verifyRes.json()
          : await verifyRes.text();

        this.notarizationResult = {
          verified: verifyRes.ok,
          data: verifyData
        };

      } catch (e) {
        console.error(e);
        this.error = 'Error during notarization or verification.';
      } finally {
        this.notarizing = false;
      }
    }
  }
};
</script>

<style scoped>
.upload-shell {
  max-width: 1100px;
  margin: 2rem auto;
  padding: 0 1rem;
}

.upload-card {
  background: linear-gradient(180deg, #ffffff 0%, #f8fafc 100%);
  border: 1px solid #e2e8f0;
  border-radius: 16px;
  box-shadow: 0 10px 30px rgba(15, 23, 42, 0.08);
  padding: 1.5rem;
}

.title {
  margin: 0;
  font-size: 1.35rem;
  color: #0f172a;
}

.subtitle {
  margin: 0.4rem 0 1rem;
  color: #475569;
  font-size: 0.95rem;
}

.file-label {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 0.75rem;
  border: 1px dashed #94a3b8;
  border-radius: 12px;
  padding: 0.75rem 0.9rem;
  background: #f8fafc;
  color: #334155;
}

.file-label input[type='file'] {
  max-width: 170px;
}

.send-btn {
  margin-top: 1rem;
  border: none;
  border-radius: 10px;
  background: linear-gradient(135deg, #4f46e5, #2563eb);
  color: #fff;
  font-weight: 700;
  padding: 0.7rem 1rem;
  cursor: pointer;
  transition: transform 0.15s ease, box-shadow 0.2s ease, opacity 0.2s ease;
}

.send-btn:hover:enabled {
  transform: translateY(-1px);
  box-shadow: 0 8px 20px rgba(37, 99, 235, 0.35);
}

.send-btn:disabled {
  opacity: 0.65;
  cursor: not-allowed;
}

.response {
  margin-top: 1rem;
  padding: 0.8rem;
  border-radius: 10px;
  border: 1px solid #cbd5e1;
  background: #f8fafc;
  color: #1e293b;
}

.response-title {
  display: block;
  margin-bottom: 0.4rem;
}

.response pre {
  margin-top: 0.65rem;
  background: #0f172a;
  color: #e2e8f0;
  border-radius: 8px;
  padding: 0.6rem;
  overflow-x: auto;
  font-size: 0.8rem;
}

.hash {
  background: #e2e8f0;
  border-radius: 6px;
  padding: 0.1rem 0.3rem;
  font-size: 0.78rem;
}

.error {
  margin-top: 0.85rem;
  color: #b91c1c;
  font-weight: 600;
}

.notarize-btn {
  margin-top: 0.75rem;
  margin-left: 0.5rem;
  border: none;
  border-radius: 10px;
  background: linear-gradient(135deg, #059669, #0d9488);
  color: #fff;
  font-weight: 700;
  padding: 0.7rem 1rem;
  cursor: pointer;
  transition: transform 0.15s ease, box-shadow 0.2s ease, opacity 0.2s ease;
}

.notarize-btn:hover:enabled {
  transform: translateY(-1px);
  box-shadow: 0 8px 20px rgba(5, 150, 105, 0.35);
}

.notarize-btn:disabled {
  opacity: 0.65;
  cursor: not-allowed;
}

.notarization-result {
  margin-top: 1rem;
  padding: 0.8rem;
  border-radius: 10px;
  border: 1px solid;
}

.notarization-result.verified {
  border-color: #059669;
  background: #ecfdf5;
  color: #065f46;
}

.notarization-result.not-verified {
  border-color: #b91c1c;
  background: #fef2f2;
  color: #7f1d1d;
}

.notarization-result pre {
  margin-top: 0.5rem;
  background: #0f172a;
  color: #e2e8f0;
  border-radius: 8px;
  padding: 0.6rem;
  overflow-x: auto;
  font-size: 0.8rem;
}
</style>
