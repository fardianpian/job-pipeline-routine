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
| Music Arranger OR Composer OR Music Director | Indonesia |
| Jingle Composer OR Film Scorer OR Mixing Engineer | Indonesia |
| Music Supervisor OR Music Coordinator OR Music Editor | Indonesia |
| Mixing Engineer OR Mastering Engineer OR Recording Engineer | Indonesia |
| Sound Designer OR SFX Artist OR Foley Artist | Indonesia |
| Audio Editor OR Game Audio OR Audio Post Production | Indonesia |
| Guru Musik OR Instruktur Musik OR Dosen Musik | Indonesia |
| Music Lecturer OR Dosen Seni OR Dosen Komposisi | Indonesia |
| AI Music Trainer OR Music Data Annotator | remote |
| Arts Program Officer OR Cultural Programmer | Indonesia |
| Event Manager OR Event Producer | Indonesia |

Response fields yang diambil: `title`, `company`, `location`, `link`, `updated`. **Jangan ambil `snippet`.**

## 4. Panggil Indeed MCP
Gunakan tool `mcp__claude_ai_Indeed__search_jobs` dengan `country_code: "ID"`.

Jalankan untuk setiap baris:

| search | location |
|---|---|
| Music Arranger OR Composer | remote |
| Jingle Composer OR Film Scorer OR Mixing Engineer | remote |
| Sound Designer OR SFX Artist OR Foley Artist | remote |
| Guru Musik OR Music Teacher OR Instruktur Musik | Bali |
| Music Lecturer OR Dosen Musik OR Dosen Seni | remote |
| AI Music Trainer OR Music Data Annotator | remote |
| Event Manager OR Event Coordinator | remote |

Response fields: `title`, `company`, `location`, `job_type`, `apply_link`.

**Jangan panggil `get_job_details`** kecuali lowongan masuk ke shortlist 20 teratas dan sama sekali tidak punya tanggal.

## 5. Panggil SerpAPI Google Jobs
Endpoint: `GET https://serpapi.com/search.json?engine=google_jobs&hl=id&gl=id&api_key={SERPAPI_KEY}&q=...`

Jalankan untuk setiap query:

| q |
|---|
| Music Arranger Indonesia |
| Composer Indonesia |
| Music Director Indonesia |
| Penata Musik Indonesia |
| Jingle Composer Indonesia |
| Film Scorer Indonesia |
| Mixing Engineer Indonesia |
| Sound Designer Indonesia |
| SFX Indonesia |
| Mixing Engineer Indonesia |
| Music Supervisor Indonesia |
| AI music trainer remote |
| Guru musik Bali |
| Music Lecturer Indonesia |
| Dosen Seni Indonesia |
| Arts program Indonesia |
| Event Production Bali |

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

**Bidang utama** (Music Arranger, Composer, Music Director, Jingle Composer, Film Scorer, Mixing Engineer, Music Supervisor, Sound Designer, SFX Artist, Foley Artist):
- **5** — bidang utama + lokasi Bali, ATAU remote (Indonesia maupun luar negeri)
- **4** — bidang utama + kota lain Indonesia tapi REMOTE
- **2** — bidang utama tapi onsite/hybrid di luar Bali → tidak memenuhi syarat lokasi

**Bidang pendukung** (Education, Arts, Event Production, AI+Music, Writing):
- **4** — Education/AI+Music remote atau di Bali
- **3** — Arts/Event di Bali atau remote
- **2** — event hotel/MICE/sales, atau pendukung onsite luar Bali
- **1** — hampir tidak relevan

**Catatan:** Lowongan onsite/hybrid di luar Bali → Fit Score maksimal 2, tidak masuk 20 teratas kecuali tidak ada hasil lain.

## 8. Kirim lowongan NEW ke Notion
Database: **🧭 Job Pipeline** — ID `05ee16d7-ef50-484e-b7f8-fd1e150d68bc`

Hanya tambahkan entry berstatus **NEW**. Properti:
- **Position** (title)
- **Company** (text)
- **Field** (select) — Music/Sound, SFX/Audio, Arts/Culture, Writing/Content, Event Management, AI/Automation, Education, Other
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
  - Education → Music/Arts
  - Event Management → Event/Arts
  - Writing/Content → Writing/Content
  - AI/Automation → AI Workflow
  - Other → Music/Arts (default)
