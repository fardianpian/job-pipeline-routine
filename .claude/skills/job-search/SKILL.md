---
name: job-search
description: Cari lowongan terbaru di Indeed & Jobstreet, beri skor, dan kirim ke database Notion Job Pipeline. Gunakan setiap kali routine berjalan.
---

# Prosedur Job Search

## 1. Cari lowongan TERBARU (diposting <= 7 hari)
Sumber utama: Indeed (id.indeed.com) & Jobstreet (id.jobstreet.com).
Sumber pelengkap (opsional): LinkedIn, Glints.

## 2. Kata kunci per bidang
- **SEO/GEO:** SEO Specialist, SEO Strategist, SEO Content Writer, Content Strategist, Technical SEO, Digital Marketing Specialist
- **AI/Automation:** AI Automation Specialist, Prompt Engineer, AI Workflow Designer, AI Implementation Consultant, AI Content Specialist
- **Notion/Systems:** Notion Consultant, Notion Specialist, Productivity Consultant, No-Code Builder
- **Content Marketing:** Content Manager, Social Media Specialist, Copywriter, Content Producer
- **Writing/Education:** Technical Writer, Instructional Designer, Course Creator, E-Learning Content Developer
- **Grant/Arts:** Grant Writer, Proposal Writer, Arts/Project Coordinator, Program Officer
- **Web/SMB:** Web Content Specialist, WordPress/CMS Specialist
- **Teaching:** Music Production Instructor, Trainer/Workshop Facilitator

Lokasi: Bali/Denpasar, Remote (Indonesia), Jakarta (remote-friendly).

## 3. Saring
- Buang lowongan duplikat & yang sudah kedaluwarsa.
- Cek database Notion "Job Pipeline" — jika lowongan dengan Position + Company yang sama sudah ada, lewati. Notion adalah satu-satunya sumber kebenaran untuk deduplication.

## 4. Skor
Beri Fit Score 1-5 tiap lowongan berdasarkan kecocokan dengan profil di `CLAUDE.md`.

## 5. Kirim ke Notion
Database: **🧭 Job Pipeline** — ID `05ee16d7-ef50-484e-b7f8-fd1e150d68bc`

Tambahkan setiap lowongan BARU via Notion connector dengan properti:
- **Position** (title) — nama posisi
- **Company** (text)
- **Field** (select) — pilih salah satu: SEO/GEO, AI/Automation, Notion/Systems, Content Marketing, Writing/Education, Grant/Arts, Web/SMB, Teaching
- **Type** (select) — Freelance, Part-time, Contract, atau Full-time Remote
- **Location** (text)
- **Fit Score** (number 1–5)
- **Status** — set ke "New"
- **Link** (url)
- **Source** (select) — Indeed, Jobstreet, LinkedIn, Glints, atau Perplexity
- **Date Posted** (date — format YYYY-MM-DD)
- **CV Variant** (select) — tentukan berdasarkan Field:
  - SEO/GEO → GEO/SEO
  - AI/Automation, Notion/Systems → AI Workflow atau Notion Consultant (pilih yang lebih cocok)
  - Field lain → GEO/SEO (default)
