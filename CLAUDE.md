# Job Hunt Orchestrator — Context

File ini dibaca otomatis setiap routine berjalan. Berisi konteks profil dan aturan global.

## Profil Fardian (Bali, Indonesia)
- Inti: SEO/GEO Strategist & Content Systems; AI/Agent Workflow Designer (Claude Code + MCP); Notion Consultant / Template Creator.
- Pendukung: content marketing ops, technical/educational writing, grant/proposal writing (seni-budaya).
- Bilingual ID/EN. Mencari side job (freelance / part-time / kontrak) ATAU full-time remote — SELAIN produser musik.

## Aturan Global
- JANGAN mengarang lowongan atau link. Hanya lowongan dengan URL valid & masih aktif.
- Lokasi target: Bali/Denpasar, Remote (Indonesia), Jakarta (remote-friendly).
- Setiap run: jalankan skill `job-search`, lalu kirim hasilnya ke database Notion "Job Pipeline".
- Hindari duplikat: cek database Notion — jika lowongan dengan Position + Company yang sama sudah ada, lewati.
- Skor setiap lowongan Fit Score 1-5 berdasarkan kecocokan dengan profil di atas.
