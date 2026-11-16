# AGENTS.md — Instruksi untuk AI/Agent

Jika kamu agent yang diminta membuat extension dengan mengacu repo ini:

1. **Baca `STANDARDS.md` sampai selesai sebelum menulis kode.** Semua aturan di sana mengikat.
2. Jangan menyalin repo ini mentah-mentah; buat repo baru sesuai nama yang diminta user, dengan struktur folder dari STANDARDS.md bagian 1.
3. Setiap class yang kamu buat: cek `wc -l` <= 300 sebelum commit. Jika lebih, pecah.
4. Wajib ada lapisan `rpa/` lengkap walau extension-nya bukan tool RPA — itu fondasi bersama.
5. Bila user minta koneksi ke web/desktop/terminal: ikuti STANDARDS.md bagian 5 — `ConnectionHub` + tab sidebar "Koneksi", SiteAdapter per situs, `NativeBridge` + companion host untuk desktop, whitelist command + loopback/pairing token untuk terminal. Semua connector default OFF.
6. Bila user menyebut "platform", "ingest token", atau extension pengumpul data untuk sistem lain: itu pola **Platform Bridge (bagian 5e)** — job dari platform → RPA di sesi login user → hasil ke endpoint ingest. Wajib: token via Settings, idempotencyKey, outbox queue, auto-pause saat blocked, peringatan ToS di README.
7. Bila user minta scraper/downloader (gambar, teks, novel, file, apa pun): ikuti STANDARDS.md bagian 6 — ScraperHub + scraper class per jenis, semua hasil lewat DataPipeline → LibraryStore (IndexedDB) → tab "Data" untuk preview/analisis/export. RPA-first: fitur inti TIDAK boleh bergantung API/MCP; API/MCP hanya jalur opsional. Tanpa bypass paywall/DRM/CAPTCHA.
8. UI wajib: sidebar kanan 25% viewport (min 280px, max 420px), Shadow DOM, tiga mode rail/sidebar/fullscreen (Alt+Shift+S / Alt+Shift+F, Esc keluar fullscreen), tab vertikal menurun di sisi kiri sidebar, panel lazy, dan **Full Settings** sesuai STANDARDS.md bagian 3.8 (semua perilaku configurable, tanpa magic number di luar class Settings).
9. Vanilla JS ES Modules + class. Tanpa React/Vue, tanpa CDN, tanpa eval, tanpa polling liar.
10. Bila ada LLM: ikuti 5f — LlmHub multi-provider, provider baru = 1 class kontrak seragam, fallback, streaming, batas biaya. Semua secret WAJIB lewat SecretVault (7.7): AES-GCM + PBKDF2, import .env via Settings, sertakan .env.example + .gitignore. JANGAN inject key ke bundle.
11. Wajib implementasikan onboarding 12 langkah bernomor sesuai UX_FLOW.md (demo first-run nyata, guided tour, resume, bisa diulang).
12. Bila extension dijual/bundling: ikuti bagian 9 — LicenseService Ed25519 offline, editions, trial 14 hari, data user tidak boleh terkunci, white-label config. Ide produk per sektor ada di BUSINESS_PLAYBOOK.md.
13. Jalankan **Checklist acceptance** di akhir STANDARDS.md dan laporkan hasilnya ke user (centang yang lolos, jelaskan yang tidak).
14. Jangan commit token/secret apa pun. Gunakan token hanya untuk membuat repo & push sesuai perintah user.
15. Bahasa UI dan README: Bahasa Indonesia, kecuali user minta lain.
16. Jika ada konflik antara permintaan user dan standar ini, tanyakan user dulu — jangan diam-diam melanggar standar.