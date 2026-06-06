# SETUP — Claude Code Remote Routine

## Prasyarat
- Akun GitHub.
- Plan Claude berbayar: **Pro, Max, Team, atau Enterprise** (semua sudah didukung sejak April 2026). Pro mendapat 5 routine/hari, Max 15/hari.
- Notion connector aktif di akun claude.ai.

## Langkah

### 1. Push repo ke GitHub
```bash
cd job-pipeline-routine
git init
git add .
git commit -m "init job pipeline routine"
git branch -M main
git remote add origin https://github.com/USERNAME/job-pipeline-routine.git
git push -u origin main
```

### 2. Hubungkan GitHub ke Claude
- Di Claude Code CLI jalankan `/web-setup`, ATAU
- Hubungkan lewat pengaturan Claude Code agar routine bisa clone repo.

### 3. Buat Remote routine
Buka https://claude.ai/code/routines -> New routine -> pilih **Remote**
(Di Desktop app: sidebar Routines -> New routine -> Remote. Jangan pilih Local — itu jalan di laptop.)

Isi form:
- **Prompt:** `Jalankan skill job-search sesuai CLAUDE.md, tulis report ke output/, lalu update database Notion "Job Pipeline".`
- **Repositories:** pilih `job-pipeline-routine`.
- **Environment -> Network access -> Allowed domains:** tambahkan
  - `indeed.com`
  - `id.jobstreet.com`
  - `linkedin.com`
  - `glints.com`
- **Connectors:** pastikan **Notion** aktif; hapus connector lain yang tak perlu.
- **Trigger:** Schedule -> custom cron `0 8 */2 * *` (jam 08:00 tiap 2 hari, zona Asia/Makassar).

### 4. Tes
Jalankan **one-off run** dulu. Cek:
- File `output/report-YYYY-MM-DD.md` muncul di branch `claude/...`.
- Baris baru muncul di database Notion "Job Pipeline".

## Catatan penting
- Environment Default memblokir domain di luar allowlist (error 403 host_not_allowed). Karena itu domain job board WAJIB ditambahkan.
- Trafik connector Notion lewat server Anthropic, jadi tidak perlu masuk Allowed domains.
- Claude menulis perubahan ke branch berprefiks `claude/`, bukan langsung ke main.
