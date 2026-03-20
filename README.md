# Project Fornitori Contratti

Frontend Vue 2 per caricamento PDF, invio hash al backend e verifica notarizzazione.

## Funzionalita principali

- Upload di un file PDF tramite interfaccia web
- Chiamata `POST /upload` con visualizzazione risposta (`message`, `filename`, `sha256`)
- Pulsante **Notarization and Check** mostrato solo dopo upload riuscito
- Chiamata `POST /api/notarize` con body `{ hash }`
- Chiamata `GET /api/verify/{id}` per verificare la notarizzazione
- Messaggio finale di successo/fallimento verifica in UI

## Stack tecnologico

- Vue 2 (`vue@2.6.14`)
- Vue CLI Service (`@vue/cli-service@4.5.0`)
- Tailwind CSS (installato, non obbligatorio nel flusso attuale)

## Requisiti

- Node.js 16+ (consigliato LTS)
- npm 8+

## Avvio in locale

1. Installa le dipendenze:

```bash
npm install
```

2. Avvia il server di sviluppo:

```bash
npm run serve
```

3. Apri il browser su:

```text
http://localhost:8080
```

## Build produzione

```bash
npm run build
```

L'output viene generato nella cartella `dist/`.

## Variabili ambiente

Il frontend usa la variabile:

- `VUE_APP_API_BASE_URL` (default: `http://localhost:3000`)

Esempio `.env.local`:

```env
VUE_APP_API_BASE_URL=http://localhost:3000
```

## API attese dal frontend

### 1) Upload PDF

- Endpoint: `POST /upload`
- Content-Type: `multipart/form-data`
- Campo file: `pdf`

Risposta attesa (esempio):

```json
{
  "message": "Upload completed",
  "filename": "contract.pdf",
  "sha256": "..."
}
```

### 2) Notarize hash

- Endpoint: `POST /api/notarize`
- Content-Type: `application/json`
- Body:

```json
{
  "hash": "<sha256>"
}
```

Risposta attesa: oggetto con `notarizationId` oppure `id`.

### 3) Verify notarization

- Endpoint: `GET /api/verify/{id}`
- Risposta: esito verifica (true/false o struttura equivalente)

## Deploy su Vercel

Il progetto include `vercel.json` con:

- `buildCommand`: `npm run build`
- `outputDirectory`: `dist`

Passi rapidi:

1. Collega il repository a Vercel
2. Imposta la variabile `VUE_APP_API_BASE_URL` nelle Environment Variables
3. Esegui il deploy

## Struttura progetto

```text
public/
  index.html
src/
  App.vue
  main.js
  components/
    Header.vue
    PdfUploader.vue
```

## Note

- Le etichette UI sono in inglese.
- Se il backend cambia formato risposta per notarizzazione/verify, aggiornare la logica in `src/components/PdfUploader.vue`.
