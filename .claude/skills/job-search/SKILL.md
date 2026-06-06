---
name: job-search
description: Cari lowongan terbaru via Jooble API + SerpAPI Google Jobs, beri skor, dan kirim ke database Notion Job Pipeline. Gunakan setiap kali routine berjalan.
---

# Prosedur Job Search

## 1. Ambil kredensial dari Instruksi routine
Baca `JOOBLE_API_KEY` dan `SERPAPI_KEY` dari teks instruksi yang diberikan di awal sesi.

## 2. Panggil Jooble API
Endpoint: `POST https://jooble.org/api/{JOOBLE_API_KEY}`
Header: `Content-Type: application/json`

Jalankan request untuk setiap kelompok kata kunci berikut (gunakan `ResultOnPage: 20`):

| keywords | location |
|---|---|
| SEO Specialist OR SEO Strategist OR Technical SEO | Indonesia |
| SEO Content Writer OR Content Strategist OR Digital Marketing | Indonesia |
| AI Automation OR Prompt Engineer OR AI Workflow | Indonesia |
| Notion Consultant OR No-Code Builder OR Productivity Consultant | Indonesia |
| Content Manager OR Copywriter OR Social Media Specialist | Indonesia |
| Technical Writer OR Instructional Designer OR E-Learning | Indonesia |
| Grant Writer OR Proposal Writer OR Program Officer | Indonesia |

Contoh request body:
```json
{
  "keywords": "SEO Specialist OR SEO Strategist",
  "location": "Indonesia",
  "ResultOnPage": 20
}
```

Field yang tersedia dari response: `title`, `company`, `location`, `salary`, `snippet`, `link`, `updated` (tanggal posting).

## 3. Panggil SerpAPI Google Jobs
Endpoint: `GET https://serpapi.com/search.json`

Parameter wajib:
- `engine=google_jobs`
- `api_key={SERPAPI_KEY}`
- `hl=id`
- `gl=id`

Jalankan untuk kata kunci prioritas tinggi:

| q |
|---|
| SEO Specialist remote Indonesia |
| AI Automation Specialist remote Indonesia |
| Content Strategist Bali remote |
| Notion Consultant freelance Indonesia |
| Technical Writer remote Indonesia |
| Digital Marketing Specialist Bali |

Contoh URL:
```
https://serpapi.com/search.json?engine=google_jobs&q=SEO+Specialist+remote+Indonesia&hl=id&gl=id&api_key={SERPAPI_KEY}
```

Field yang tersedia dari `jobs_results[]`: `title`, `company_name`, `location`, `detected_extensions.posted_at`, `detected_extensions.schedule_type`, `apply_link`.

## 4. Gabungkan & Saring
- Gabungkan hasil dari Jooble dan SerpAPI.
- Buang duplikat berdasarkan kombinasi title + company.
- Buang lowongan yang `updated` / `posted_at` lebih dari 14 hari dari hari ini.
- Cek database Notion "Job Pipeline" — jika Position + Company sudah ada, lewati.

## 5. Skor
Beri Fit Score 1–5 per lowongan berdasarkan profil di `CLAUDE.md`:
- **5** — cocok sempurna (bidang inti + remote/Bali + tipe sesuai)
- **4** — cocok baik (bidang inti, lokasi sedikit kompromi)
- **3** — relevan (bidang pendukung atau lokasi kurang ideal)
- **2** — marginal
- **1** — hampir tidak relevan, tapi masih layak dicatat

## 6. Kirim ke Notion
Database: **🧭 Job Pipeline** — ID `05ee16d7-ef50-484e-b7f8-fd1e150d68bc`

Tambahkan setiap lowongan BARU via Notion connector dengan properti:
- **Position** (title) — nama posisi
- **Company** (text)
- **Field** (select) — SEO/GEO, AI/Automation, Notion/Systems, Content Marketing, Writing/Education, Grant/Arts, Web/SMB, Teaching
- **Type** (select) — Freelance, Part-time, Contract, atau Full-time Remote
- **Location** (text)
- **Fit Score** (number 1–5)
- **Status** — set ke "New"
- **Link** (url) — gunakan `apply_link` (SerpAPI) atau `link` (Jooble)
- **Source** (select) — tulis "Jooble" atau "Google Jobs" sesuai asal data
- **Date Posted** (date — format YYYY-MM-DD, konversi dari `updated` atau `posted_at`)
- **CV Variant** (select):
  - SEO/GEO → GEO/SEO
  - AI/Automation, Notion/Systems → AI Workflow atau Notion Consultant
  - Field lain → GEO/SEO (default)
