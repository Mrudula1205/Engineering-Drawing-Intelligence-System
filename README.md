# DocIntelligence

DocIntelligence provides a lightweight backend and frontend for extracting structured data from engineering drawings and PDFs. It combines Document AI outputs, OCR fallbacks, and visual-language model (VLM) based spatial extraction to produce JSON payloads suitable for BOMs, title blocks, dimensions, and notes extraction.

## Table of contents

- Features
- Quickstart (development)
- Architecture
- Running with Docker
- Deployment
- Project layout

## Features

- Extract title blocks, BOMs, notes, and structured text from PDFs.
- Visual-language model (VLM) based spatial dimension extraction.
- Merge and validation pipeline with Firestore persistence and job status tracking.
- Frontend UI built with Next.js for previewing extraction results and downloads.

## Quickstart (development)

Prerequisites:
- Python 3.10+
- Node.js 18+
- (Optional) Docker

Backend (Windows PowerShell):

```powershell
cd backend
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
# create or copy .env and fill credentials
copy .env.example .env
uvicorn main:app --reload
```

Backend (macOS / Linux):

```bash
cd backend
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
# create or copy .env and fill credentials
cp .env.example .env
uvicorn main:app --reload
```

The backend API will be available at `http://127.0.0.1:8000` by default.

Frontend:

```bash
cd frontend
npm install
npm run dev
```

Open the frontend at `http://localhost:3000`.

## Architecture

The repository implements an asynchronous pipeline with parallel extraction branches and a final validation/merge stage.

```mermaid
graph TD
  ingest[Ingest] --> extract_structured[Extract Structured Data]
  ingest --> extract_dims[Extract Dimensions using VLM]
  extract_structured --> validate[Validate & Merge]
  extract_dims --> validate
  validate --> store[Store (Firestore)]
  store --> end_node((END))
```

Node overview:
- `ingest`: load PDFs, run Document AI, and produce raw text blocks / layout info.
- `extract_structured_data`: extract title block, BOM and notes (falls back to OCR when needed).
- `extract_dimensions_vlm`: send page images to a VLM for spatial dimension extraction.
- `validate_merge`: merge parallel results and determine job outcome.
- `store_firestore`: persist results and update job status.

See the pipeline implementation in `pipeline/` and node implementations under `pipeline/nodes/`.

## Running with Docker

Build images from repository root:

```bash
docker build -f backend/Dockerfile -t docint-backend .
docker build -f frontend/Dockerfile -t docint-frontend .
```

Compose or Kubernetes manifests can be used for production deployment. Kubernetes manifests are in the `k8s/` folder.

## Deployment

- Cloud build configuration is provided in `cloudbuild.yaml`.
- Kubernetes manifests: `k8s/deployment.yaml`, `k8s/service.yaml`.

## Project layout (high level)

- `backend/` — FastAPI backend, `main.py` and `routers/` for API endpoints.
- `pipeline/` — orchestration graph and node implementations.
- `models/` — shared Pydantic schemas.
- `frontend/` — Next.js app for interacting with and viewing results.
- `k8s/`, `cloudbuild.yaml` — deployment configs.

