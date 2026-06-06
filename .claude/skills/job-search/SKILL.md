---
name: job-search
description: Cari lowongan terbaru via Jooble API + SerpAPI Google Jobs, beri skor, tandai NEW/ACTIVE, dan kirim ke database Notion Job Pipeline. Gunakan setiap kali routine berjalan.
---

# Prosedur Job Search

## 1. Ambil kredensial
Baca `JOOBLE_API_KEY` dan `SERPAPI_KEY` dari teks instruksi yang diberikan di awal sesi.

## 2. Panggil Jooble API
Endpoint: `POST https://jooble.org/api/{JOOBLE_API_KEY}`
Header: `Content-Type: application/json`

Jalankan untuk setiap kelompok kata kunci (`ResultOnPage: 20`):

| keywords | location |
|---|---|
| SEO Specialist OR SEO Strategist OR Technical SEO | Indonesia |
| SEO Content Writer OR Content Strategist OR Digital Marketing | Indonesia |
| AI Automation OR Prompt Engineer OR AI Workflow | Indonesia |
| Notion Consultant OR No-Code Builder OR Productivity Consultant | Indonesia |
| Content Manager OR Copywriter OR Social Media Specialist | Indonesia |
| Technical Writer OR Instructional Designer OR E-Learning | Indonesia |
| Grant Writer OR Proposal Writer OR Program Officer | Indonesia |

Response fields: `title`, `company`, `location`, `snippet`, `link`, `updated`.

## 3. Panggil SerpAPI Google Jobs
Endpoint: `GET https://serpapi.com/search.json?engine=google_jobs&hl=id&gl=id&api_key={SERPAPI_KEY}&q=...`

Jalankan untuk query berikut:

| q |
|---|
| SEO Specialist remote Indonesia |
| AI Automation Specialist remote Indonesia |
| Content Strategist Bali remote |
| Notion Consultant freelance Indonesia |
| Technical Writer remote Indonesia |
| Digital Marketing Specialist Bali |

Response fields: `jobs_results[].title`, `company_name`, `location`, `detected_extensions.posted_at`, `detected_extensions.schedule_type`, `apply_link`.

## 4. Gabungkan & Tentukan Status NEW / ACTIVE

Gabungkan semua hasil Jooble + SerpAPI, lalu:

1. Buang duplikat internal (title + company sama).
2. Buang posting lebih dari 14 hari.
3. Untuk setiap lowongan, cek database Notion "Job Pipeline":
   - **Tidak ada** → status = **NEW**, tambahkan ke Notion.
   - **Sudah ada & Status masih "New" atau "Maybe"** → status = **ACTIVE**, jangan tambah duplikat, cukup catat di output tabel.
   - **Sudah ada & Status "Applied" / "Interview" / dst** → abaikan sepenuhnya.
4. Batasi total output maksimal **20 lowongan** — prioritaskan Fit Score tertinggi, lalu tanggal terbaru.

## 5. Skor
Beri Fit Score 1–5 per lowongan berdasarkan profil di `CLAUDE.md`:
- **5** — cocok sempurna (bidang inti + remote/Bali + tipe sesuai)
- **4** — cocok baik (bidang inti, lokasi sedikit kompromi)
- **3** — relevan (bidang pendukung atau lokasi kurang ideal)
- **2** — marginal
- **1** — hampir tidak relevan, tapi masih layak dicatat

## 6. Kirim lowongan NEW ke Notion
Database: **🧭 Job Pipeline** — ID `05ee16d7-ef50-484e-b7f8-fd1e150d68bc`

Hanya tambahkan entry berstatus **NEW**. Properti:
- **Position** (title)
- **Company** (text)
- **Field** (select) — SEO/GEO, AI/Automation, Notion/Systems, Content Marketing, Writing/Education, Grant/Arts, Web/SMB, Teaching
- **Type** (select) — Freelance, Part-time, Contract, Full-time Remote
- **Location** (text)
- **Fit Score** (number 1–5)
- **Status** — "New"
- **Link** (url) — `apply_link` (SerpAPI) atau `link` (Jooble)
- **Source** (select) — "Jooble" atau "Google Jobs"
- **Date Posted** (date YYYY-MM-DD)
- **CV Variant** (select):
  - SEO/GEO → GEO/SEO
  - AI/Automation → AI Workflow
  - Notion/Systems → Notion Consultant
  - Field lain → GEO/SEO (default)
