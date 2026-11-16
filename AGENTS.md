# AGENTS.md — Instruksi untuk AI/Agent

Jika kamu agent yang diminta membuat extension dengan mengacu repo ini:

1. **Baca `STANDARDS.md` sampai selesai sebelum menulis kode.** Semua aturan di sana mengikat.
2. Jangan menyalin repo ini mentah-mentah; buat repo baru sesuai nama yang diminta user, dengan struktur folder dari STANDARDS.md bagian 1.
3. Setiap class yang kamu buat: cek `wc -l` <= 300 sebelum commit. Jika lebih, pecah.
4. Wajib ada lapisan `rpa/` lengkap walau extension-nya bukan tool RPA — itu fondasi bersama.
5. Bila user minta koneksi ke web/desktop/terminal: ikuti STANDARDS.md bagian 5 — `ConnectionHub` + tab sidebar "Koneksi", SiteAdapter per situs, `NativeBridge` + companion host untuk desktop, whitelist command + loopback/pairing token untuk terminal. Semua connector default OFF.
6. Bila user menyebut "platform", "ingest token", atau extension pengumpul data untuk sistem lain: itu pola **Platform Bridge (bagian 5e)** — job dari platform → RPA di sesi login user → hasil ke endpoint ingest. Wajib: token via Settings, idempotencyKey, outbox queue, auto-pause saat blocked, peringatan ToS di README.
7. Bila user minta scraper/downloader (gambar, teks, novel, file, apa pun): ikuti STANDARDS.md bagian 6 — ScraperHub + scraper class per jenis, semua hasil lewat DataPipeline → LibraryStore (IndexedDB) → tab "Data" untuk preview/analisis/export. RPA-first: fitur inti TIDAK boleh bergantung API/MCP; API/MCP hanya jalur opsional. Tanpa bypass paywall/DRM/CAPTCHA.
8. UI wajib: sidebar kanan 25% viewport (min 280px, max 420px), Shadow DOM, collapsible, tab vertikal menurun di sisi kiri sidebar, panel lazy.
9. Vanilla JS ES Modules + class. Tanpa React/Vue, tanpa CDN, tanpa eval, tanpa polling liar.
10. Jalankan **Checklist acceptance** di akhir STANDARDS.md dan laporkan hasilnya ke user (centang yang lolos, jelaskan yang tidak).
11. Jangan commit token/secret apa pun. Gunakan token hanya untuk membuat repo & push sesuai perintah user.
12. Bahasa UI dan README: Bahasa Indonesia, kecuali user minta lain.
13. Jika ada konflik antara permintaan user dan standar ini, tanyakan user dulu — jangan diam-diam melanggar standar.