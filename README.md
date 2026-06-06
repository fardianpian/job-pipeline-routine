# job-pipeline-routine

Repositori untuk **Claude Code Remote Routine** yang mencari lowongan kerja sampingan tiap 2 hari dan mengirim hasilnya ke database Notion "Job Pipeline".

## Isi
- `CLAUDE.md` — konteks profil + aturan global (dibaca tiap run).
- `.claude/skills/job-search/SKILL.md` — prosedur pencarian lowongan.
- `.mcp.json` — (opsional) deklarasi Notion MCP server.
- `output/` — folder hasil report markdown per run.

## Cara pakai
1. Push repo ini ke GitHub.
2. Hubungkan GitHub ke akun Claude (`/web-setup` di CLI).
3. Buat Remote routine di https://claude.ai/code/routines, pilih repo ini.
4. Atur environment: tambahkan domain `indeed.com`, `id.jobstreet.com`, `linkedin.com`, `glints.com` ke Allowed domains.
5. Aktifkan connector Notion.
6. Set schedule trigger tiap 2 hari (cron `0 8 */2 * *`, zona Asia/Makassar).

Lihat `SETUP.md` untuk langkah detail.
