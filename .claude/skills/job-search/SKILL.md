---
name: job-search
description: Cari lowongan terbaru via Jooble API + SerpAPI Google Jobs + Indeed MCP, beri skor, tandai NEW/ACTIVE, dan kirim ke database Notion Job Pipeline. Gunakan setiap kali routine berjalan.
---

# Prosedur Job Search

## 1. Ambil kredensial
Baca `JOOBLE_API_KEY` dan `SERPAPI_KEY` dari teks instruksi di awal sesi.

## 2. Fetch entri Notion yang sudah ada (SATU kali)
Query database Notion `05ee16d7-ef50-484e-b7f8-fd1e150d68bc` **sekali** — ambil semua entri, simpan di memori:
- Daftar semua nilai **Link** (URL)
- Daftar semua pasangan **Position + Company**

Gunakan data ini untuk deduplication di Langkah 6. **Jangan cek Notion per-lowongan.**

## 3. Panggil Jooble API
Endpoint: `POST https://jooble.org/api/{JOOBLE_API_KEY}`
Header: `Content-Type: application/json`

Jalankan untuk setiap baris (`ResultOnPage: 5`):

| keywords | location |
|---|---|
| Music Supervisor OR Music Coordinator OR Music Editor | Indonesia |
| Composer OR Arranger OR Music Director | Indonesia |
| Sound Designer OR Audio Engineer OR SFX Artist | Indonesia |
| Arts Program Officer OR Cultural Programmer OR Arts Administrator | Indonesia |
| Music Writer OR Arts Writer OR Cultural Writer | Indonesia |
| Event Manager OR Event Coordinator OR Event Producer | Indonesia |
| AI Specialist OR Prompt Engineer OR AI Workflow Designer | Indonesia |
| Komposer OR Pengaransir OR Direktur Musik OR Koordinator Acara | Indonesia |

Response fields yang diambil: `title`, `company`, `location`, `link`, `updated`. **Jangan ambil `snippet`.**

## 4. Panggil Indeed MCP
Gunakan tool `mcp__claude_ai_Indeed__search_jobs` dengan `country_code: "ID"`.

Jalankan untuk setiap baris:

| search | location |
|---|---|
| Music Supervisor OR Music Coordinator OR Composer | remote |
| Arranger OR Music Director OR Music Editor | remote |
| Sound Designer OR Audio Editor OR SFX Designer | remote |
| Arts Administrator OR Cultural Program Officer | remote |
| Music Writer OR Arts Writer | remote |
| Event Manager OR Event Coordinator | remote |
| AI Specialist OR Prompt Engineer | remote |
| Komposer OR Desainer Suara OR Koordinator Acara | Bali |

Response fields: `title`, `company`, `location`, `job_type`, `apply_link`.

**Jangan panggil `get_job_details`** kecuali lowongan masuk ke shortlist 20 teratas dan sama sekali tidak punya tanggal.

## 5. Panggil SerpAPI Google Jobs
Endpoint: `GET https://serpapi.com/search.json?engine=google_jobs&hl=id&gl=id&api_key={SERPAPI_KEY}&q=...`

Jalankan untuk setiap query:

| q |
|---|
| Music Supervisor OR Composer OR Arranger Indonesia |
| Sound Designer OR SFX Artist OR Music Director Indonesia |
| Arts Program Officer OR Cultural Programmer Indonesia |
| Music Writer OR Arts Writer freelance Indonesia |
| Event Manager OR Event Coordinator Bali |
| AI Specialist OR Prompt Engineer Bali remote |

Response fields: `jobs_results[].title`, `company_name`, `location`, `detected_extensions.posted_at`, `detected_extensions.schedule_type`, `apply_link`.

## 6. Gabungkan & Filter
1. Buang duplikat internal (title + company sama).
2. Buang posting lebih dari 14 hari. Jika tanggal tidak tersedia, tetap masukkan.
3. Gunakan data Notion dari Langkah 2 untuk filter:
   - Link cocok → skip.
   - Position + Company cocok → skip.
   - Tidak cocok → **NEW**.
   - Cocok & Status "New"/"Maybe" → **ACTIVE** (catat di output, jangan tambah entry baru).
   - Cocok & Status "Applied"/"Interview"/"Offer"/"Rejected" → abaikan.
4. Batasi output maksimal **20 lowongan** — prioritaskan Fit Score tertinggi, lalu tanggal terbaru.

## 7. Skor
Beri Fit Score 1–5 per lowongan berdasarkan profil di `CLAUDE.md`:
- **5** — cocok sempurna (bidang inti + remote/Bali + tipe sesuai)
- **4** — cocok baik (bidang inti, lokasi sedikit kompromi)
- **3** — relevan (bidang pendukung atau lokasi kurang ideal)
- **2** — marginal
- **1** — hampir tidak relevan, tapi masih layak dicatat

## 8. Kirim lowongan NEW ke Notion
Database: **🧭 Job Pipeline** — ID `05ee16d7-ef50-484e-b7f8-fd1e150d68bc`

Hanya tambahkan entry berstatus **NEW**. Properti:
- **Position** (title)
- **Company** (text)
- **Field** (select) — Music/Sound, SFX/Audio, Arts/Culture, Writing/Content, Event Management, AI/Automation, Other
- **Type** (select) — Freelance, Part-time, Contract, Full-time Remote
- **Location** (text)
- **Fit Score** (number 1–5)
- **Status** — "New"
- **Link** (url) — `apply_link` (SerpAPI/Indeed) atau `link` (Jooble)
- **Source** (select) — "Jooble", "Google Jobs", atau "Indeed"
- **Date Posted** (date YYYY-MM-DD, kosongkan jika tidak ada)
- **CV Variant** (select):
  - Music/Sound → Music/Arts
  - SFX/Audio → Music/Arts
  - Arts/Culture → Music/Arts
  - Event Management → Event/Arts
  - Writing/Content → Writing/Content
  - AI/Automation → AI Workflow
  - Other → Music/Arts (default)
