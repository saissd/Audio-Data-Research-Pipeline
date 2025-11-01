# Audio Data Research Pipeline (ADR)

A minimal, educational project demonstrating an **audio data pipeline**: record or upload audio in the browser, preprocess with **FFmpeg**, store metadata in **PostgreSQL (Prisma)**, and browse clips with a simple UI. Designed with **Next.js + TypeScript + TailwindCSS** and **Node.js API routes**. An optional **FastAPI** service shows where **ASR (e.g., Whisper)** could plug in.

> This repo is intended as a clear reference implementation and portfolio artifact—small, readable, and easy to extend.

---

## Features

- **Web capture & upload**: Browser-based recording (**WebRTC**) or file upload.
- **Preprocessing**: Server-side **FFmpeg** resample to mono 16 kHz WAV; basic duration/metrics.
- **Object storage**: Uploads to **AWS S3** (replaceable with any S3-compatible store).
- **Metadata**: Clip records stored in **PostgreSQL** via **Prisma**.
- **Dataset Browser**: View recent clips and simple quality fields.
- **API-first**: Next.js **API Routes** for upload and processing.
- **Optional ASR**: **FastAPI** endpoint to illustrate where transcription would run.

---

## Tech Stack

- **Frontend**: Next.js (App Router), TypeScript, TailwindCSS  
- **Backend**: Node.js (Next.js API Routes), ffmpeg-static + fluent-ffmpeg  
- **Data**: PostgreSQL + Prisma ORM  
- **Cloud**: AWS S3 (v3 SDK) for object storage  
- **Realtime / Media**: WebRTC MediaRecorder API  
- **Optional**: FastAPI microservice for ASR integration

---

## Repository Structure

```
.
├─ apps/web/                 # Next.js app (UI + API routes)
│  ├─ src/app/api/upload/    # Upload endpoint → S3 + DB record
│  ├─ src/app/api/process/   # Post-upload processing with FFmpeg
│  ├─ src/app/dataset/       # Dataset browser page
│  └─ src/server/            # Prisma client, lightweight tRPC router (optional)
├─ packages/db/              # Prisma schema & client
├─ services/asr/             # Optional FastAPI ASR service (stub)
├─ jobs/                     # Example background job shape (stub)
└─ README.md
```

---

## Getting Started (optional to run)

> You can upload this repo as-is to GitHub. If you want to run it locally, follow these steps.

### 1) Prerequisites
- Node.js 18+
- PostgreSQL 14+ (local or remote)
- ffmpeg (bundled via **ffmpeg-static** for server usage)
- (Optional) Python 3.10+ for the FastAPI service

### 2) Configure Environment

Copy the sample env and edit values:

```bash
cp .env.example .env
```

**.env variables**

```
# Database
DATABASE_URL=postgresql://postgres:postgres@localhost:5432/adr

# AWS (or S3-compatible) storage
AWS_REGION=us-east-1
AWS_S3_BUCKET=your-s3-bucket
AWS_ACCESS_KEY_ID=your-access-key
AWS_SECRET_ACCESS_KEY=your-secret-key

# (Optional) ASR microservice
ASR_BASE_URL=http://localhost:8009
```

### 3) Install & Initialize

```bash
cd apps/web
npm install
npx prisma generate
npx prisma migrate dev --name init
npm run dev
```

App will start at **http://localhost:3000**.

### 4) (Optional) Start ASR Microservice

```bash
cd services/asr
pip install -r requirements.txt
uvicorn app:app --reload --port 8009
```

---

## How It Works

1. **Record/Upload** → Client uses WebRTC (or file picker) to send audio to `/api/upload`.  
2. **Store** → Server uploads the raw file to S3 and creates a `Clip` record in PostgreSQL.  
3. **Process** → Server invokes `/api/process` to normalize format (mono 16 kHz WAV) and compute simple metrics (e.g., duration).  
4. **Browse** → The `/dataset` page lists recent clips and their metrics.  
5. **ASR (optional)** → If enabled, the app can call the FastAPI endpoint to attach transcripts later.

---

## Data Model (Prisma)

```prisma
model Clip {
  id           String   @id @default(cuid())
  filename     String
  s3Key        String
  durationSec  Float?
  silencePct   Float?
  snrDb        Float?
  sampleRate   Int?
  channels     Int?
  hash         String?
  createdAt    DateTime @default(now())
  transcript   String?
  status       String   @default("UPLOADED") // e.g., PROCESSED, TRANSCRIBED
}
```

---

## Notes & Extensibility

- **Fraud/quality heuristics**: Add duplicate detection (hash/near-dup), min/max duration, silence thresholds, or MFCC/embedding checks in `process` API or a background job.  
- **Privacy**: Do not commit real keys or sensitive audio. Use `.env` and scrub data before sharing.  
- **Cloud swap**: S3 usage can be replaced with GCS/MinIO by swapping the client.  
- **ASR**: Replace the FastAPI stub with a real model (Whisper, NeMo, etc.) and store transcripts in `Clip.transcript`.

---

## Roadmap (nice-to-have)

- [ ] Basic charts (duration distribution, silence%) on `/dataset`  
- [ ] Near-duplicate detection using perceptual hashes or embeddings  
- [ ] Upload retries & resumable uploads  
- [ ] Role-based access & signed URLs  
- [ ] Batch transcription queue

---

## License

MIT — see `LICENSE`.

---

## Credits

Built for demonstration and hiring discussions focused on **Next.js / TypeScript / Node.js / PostgreSQL / Prisma / AWS S3 / WebRTC / FFmpeg / FastAPI** skills relevant to audio data pipelines.
