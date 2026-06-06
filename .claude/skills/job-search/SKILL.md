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
- Bandingkan dengan report di `output/` dari run sebelumnya agar tidak mengirim ulang lowongan yang sama.

## 4. Skor
Beri Fit Score 1-5 tiap lowongan berdasarkan kecocokan dengan profil di `CLAUDE.md`.

## 5. Tulis report
Simpan ke `output/report-YYYY-MM-DD.md` berisi:
- Ringkasan eksekutif singkat (jumlah temuan, highlight Fit Score >= 4).
- Tabel: Status | Posisi | Perusahaan | Lokasi | Tipe | Bidang | Tgl Posting | Fit Score | Link

## 6. Kirim ke Notion
Tambahkan setiap lowongan BARU ke database Notion "Job Pipeline" via Notion connector dengan properti:
- Position (title), Company, Field, Type, Location, Fit Score, Status = New, Link, Source, Date Posted
