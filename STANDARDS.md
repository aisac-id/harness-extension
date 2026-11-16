# STANDARDS — Harness Extension v1.5

Semua extension di bawah akun `aisac-id` wajib mematuhi dokumen ini.
Kata kunci: **WAJIB** (harus), **DISARANKAN** (sebaiknya), **DILARANG** (tidak boleh).

---

## 1. Arsitektur & Kode

1. **Maks 300 LoC per file** (total baris termasuk komentar). Jika lebih, pecah jadi class/module baru.
2. **OOP wajib**: gunakan `class` ES6+, bukan kumpulan fungsi lepas.
   - Satu file = satu class utama (helper kecil boleh menyertai).
   - Single Responsibility: `SidebarUI` tidak boleh sekaligus mengurus storage dan RPA.
   - Komposisi lebih baik daripada inheritance dalam; inheritance maks 2 level.
   - Dependency injection lewat constructor, bukan global variable.
3. **ES Modules** (`import`/`export`), tanpa bundler jika memungkinkan. Jika perlu build, gunakan esbuild (bukan webpack berat).
4. Penamaan: class `PascalCase.js`, util `camelCase.js`, folder `kebab-case`.
5. Komunikasi antar-konteks (background ↔ content ↔ sidebar) WAJIB lewat satu class `MessageBus` dengan tipe pesan terdefinisi (konstanta, bukan string tersebar).
6. Semua operasi async pakai `async/await` + `try/catch` di boundary; error dilempar sebagai class `AppError` dengan kode.
7. **DILARANG**: `eval`, `new Function`, remote code, jQuery, lodash full, moment.js, dan library > 50 KB kecuali disetujui eksplisit.

### Struktur folder standar

```
extension/
├── manifest.json
├── background/
│   └── ServiceWorker.js      # entry MV3, tipis, delegasi ke core
├── content/
│   └── ContentBridge.js      # injeksi sidebar, jembatan DOM
├── sidebar/
│   ├── SidebarApp.js         # root UI, mount/shadow DOM
│   ├── VerticalTabs.js       # navigasi tab vertikal
│   ├── panels/               # 1 class panel per tab, maks 300 LoC
│   └── sidebar.css
├── core/
│   ├── MessageBus.js
│   ├── StorageService.js     # wrapper chrome.storage
│   ├── SecretVault.js        # enkripsi secret + import .env (7.7-7.9)
│   └── Settings.js
├── llm/                      # LLM multi-provider (lihat 5f)
│   ├── LlmHub.js             # registri provider, routing, fallback, biaya
│   ├── providers/            # 1 class per provider, maks 300 LoC
│   └── PromptLibrary.js      # template prompt per tugas bisnis
├── license/
│   └── LicenseService.js     # aktivasi lisensi/bundle offline (bagian 9)
├── onboarding/
│   └── OnboardingFlow.js     # tutorial bernomor (bagian 8, UX_FLOW.md)
├── rpa/
│   ├── RpaEngine.js          # eksekusi langkah
│   ├── SelectorEngine.js     # resolusi selector + fallback
│   ├── ActionLibrary.js      # click/type/wait/scroll/extract, dll.
│   └── ScriptStore.js        # simpan/export/import script JSON
├── scrapers/                 # mesin panen data (lihat bagian 6)
│   ├── ScraperHub.js         # registri scraper + orkestrasi job panen
│   ├── ImageScraper.js
│   ├── TextScraper.js        # teks, tabel, list, metadata
│   ├── NovelScraper.js       # multi-halaman/bab, export TXT/EPUB/MD
│   └── FileDownloader.js     # unduh file via chrome.downloads
├── pipeline/                 # olah, simpan, analisis hasil panen
│   ├── DataPipeline.js       # extract → clean → store → export/kirim
│   ├── LibraryStore.js       # IndexedDB: dataset + blob, metadata
│   ├── AnalysisService.js    # statistik, dedupe, filter
│   └── ExportService.js      # JSON/CSV/Markdown/TXT/EPUB/ZIP
├── connectors/               # lapisan konektivitas (lihat bagian 5)
│   ├── ConnectionHub.js      # registri + status semua koneksi
│   ├── web/                  # 1 class SiteAdapter per situs/API
│   ├── NativeBridge.js       # native messaging ke desktop app
│   ├── LocalBridge.js        # WebSocket ws://127.0.0.1 (terminal/CLI)
│   ├── PlatformBridge.js     # koneksi ke platform via ingest token (5e)
│   └── native-host/          # companion app + script install per OS
└── utils/                    # helper murni, tanpa state
```

---

## 2. Performa — Ringan di Browser (WAJIB)

1. Total ukuran extension (unpacked) **< 500 KB**; target < 200 KB.
2. **Vanilla JS + Web Components/Shadow DOM**. Framework hanya boleh Preact (maks), DILARANG React/Vue/Angular penuh.
3. **Event-driven, bukan polling**: gunakan `MutationObserver`, `chrome.webNavigation`, event DOM. Interval periodik hanya via `chrome.alarms` (minimum 30 detik) dan harus bisa dimatikan.
4. Service worker MV3 harus **stateless-friendly**: state di `chrome.storage.session/local`, bukan variabel global (worker bisa mati kapan saja).
5. Content script: injeksi sidebar **lazy** (hanya saat diaktifkan user), jangan parse/scan seluruh DOM saat load.
6. Debounce/throttle semua handler `scroll`, `resize`, `input` (min 100–250 ms).
7. Animasi hanya `transform`/`opacity` (GPU), DILARANG animasi `width/height/top/left` yang memicu reflow.
8. Tanpa CDN eksternal: semua asset lokal, font system stack (`system-ui`), ikon SVG inline.
9. Batas memori: observer/listener wajib di-`disconnect`/`remove` saat sidebar ditutup; DILARANG menyimpan referensi node DOM lepas (memory leak).
10. Idle cost target: **0% CPU** saat tidak ada aktivitas; verifikasi lewat Chrome Task Manager.

---

## 3. UI — Sidebar & Tab Vertikal (WAJIB)

1. Sidebar menempel di **pinggir kanan** viewport, lebar **25% (1/4)** layar.
   - `min-width: 280px`, `max-width: 420px` agar tetap usable di layar kecil/besar.
   - Tinggi 100% viewport, `position: fixed`, `z-index` maksimum yang wajar (2147483646).
2. Render di dalam **Shadow DOM** agar CSS halaman host tidak bocor masuk dan sebaliknya.
3. **Tiga mode tampilan** (state tersimpan di storage, transisi hanya `transform/opacity`):
   - **Rail**: collapsed, ±48px, hanya icon tab.
   - **Sidebar** (default): 25% viewport sesuai aturan di atas.
   - **Fullscreen**: overlay 100% viewport untuk kerja berat (tabel data, reader novel, editor script RPA, galeri gambar). Toggle via tombol di header + shortcut `Alt+Shift+F`; `Esc` keluar dari fullscreen; konten panel harus responsif terhadap ketiga mode (grid melebar, tabel tambah kolom).
   - Toggle rail↔sidebar via tombol + shortcut `Alt+Shift+S`.
4. **Tab vertikal**: icon rail menurun di sisi kiri sidebar (bukan tab horizontal di atas).
   - Setiap tab = satu panel class terpisah di `sidebar/panels/`.
   - Panel di-render lazy saat tab pertama kali dibuka.
   - Tab aktif ditandai visual jelas; label muncul saat hover/expanded.
5. Sidebar TIDAK boleh menggeser layout halaman secara default (overlay). Mode "push content" opsional, off by default.
6. Tema: dukung dark/light mengikuti `prefers-color-scheme`, tanpa flash saat load.
7. Semua teks UI Bahasa Indonesia (kecuali diminta lain); aksesibel: focusable, `aria-label` pada icon button, kontras cukup.
8. **Full Settings (WAJIB)** — tab "Settings" (icon gear, posisi terbawah rail vertikal), satu class `SettingsPanel` + sub-panel per kategori (masing-masing maks 300 LoC):
   - **Semua perilaku harus bisa diatur** — DILARANG magic number hardcoded di luar class `Settings` (delay RPA, retry, timeout, batas paralel unduhan, filter ukuran gambar, kuota storage, pacing scraper, polling interval, dll. semua punya default di `Settings` dan bisa diubah user).
   - Kategori minimal: **Umum** (bahasa, tema dark/light/system, mode tampilan default, shortcut), **RPA** (delay, retry, timeout, batas aksi/jam), **Scraper** (per jenis: filter, pacing, paralel, format export default), **Koneksi** (toggle per connector, Platform URL + ingest token, pairing token, whitelist command, status uji koneksi/test connection), **Data & Penyimpanan** (kuota, pemakaian, bersihkan per dataset, lokasi unduhan), **Lanjutan** (level log, mode developer).
   - Fitur wajib panel Settings: **pencarian setting**, validasi input + pesan error jelas, tombol **reset per kategori & reset semua** (dengan konfirmasi), **export/import settings JSON** (secret/token TIDAK ikut ter-export secara default; opsi sertakan dengan peringatan eksplisit), perubahan berlaku **tanpa reload** bila memungkinkan.
   - Secret ditampilkan tersamarkan (masked) dengan tombol lihat/salin, tidak pernah muncul di log.
   - Settings versi-skema (`settingsVersion`) untuk migrasi saat extension update.

---

## 4. RPA — Semua Extension Harus RPA-Ready (WAJIB)

Setiap extension, apa pun fungsinya, wajib punya lapisan RPA yang bisa dipakai ulang:

1. **ActionLibrary** minimal mendukung: `click`, `type`, `select`, `scroll`, `wait`, `waitForSelector`, `extract`, `screenshot` (via chrome API), `navigate`, `loop`, `if` (kondisi sederhana).
2. **SelectorEngine** dengan urutan fallback:
   `data-*` attribute → `id` → `aria-label`/role → CSS stabil → XPath (terakhir).
   Hindari selector yang bergantung urutan index/class hasil generate.
3. **Wait strategy**: `MutationObserver` + timeout, DILARANG `setTimeout` buta sebagai satu-satunya strategi tunggu.
4. Setiap action: retry dengan backoff (default 3x), timeout per langkah (default 10 detik), dan hasil berstatus `ok | failed | skipped` beserta alasan.
5. **Script format JSON** standar (steps array: `{action, selector, value, options}`), bisa **export/import** lewat file — sehingga script bisa dibagikan antar extension.
6. **RpaEngine** berjalan step-by-step, bisa **pause/resume/stop** dari sidebar, dan menampilkan log langkah real-time di panel "RPA".
7. Hormati situs: delay antar aksi configurable (default 300–800 ms), tanpa bypass CAPTCHA/login, tanpa aksi massal destruktif.
8. Fitur recorder (rekam klik/ketikan user jadi script) DISARANKAN; jika ada, wajib bisa dimatikan dan tidak merekam nilai field password.

---

## 5. Konektivitas — Web, Desktop App, Terminal (WAJIB bila fitur diminta)

Semua koneksi dikelola satu class **`ConnectionHub`**: registri connector, status (`connected | disconnected | error`), dan indikator status real-time di tab sidebar **"Koneksi"**. Setiap connector adalah class terpisah di `connectors/`, maks 300 LoC, dengan API seragam: `connect()`, `disconnect()`, `send(request)`, event status.

Format pesan seragam antar semua jalur (envelope JSON):

```json
{ "id": "uuid", "type": "request|response|event", "action": "nama.aksi", "payload": {}, "error": null }
```

Semua request punya timeout (default 10 detik) dan retry backoff untuk koneksi yang putus.

### 5a. Koneksi ke web / situs tertentu

1. **SiteAdapter**: satu class per situs/API di `connectors/web/`, mendeklarasikan `matches` (URL pattern) sendiri. `host_permissions` manifest HANYA berisi situs yang didukung — DILARANG `<all_urls>` untuk fitur ini.
2. Adapter memisahkan: autentikasi, pemanggilan API, dan parsing respons — sehingga ganti situs cukup ganti/tambah adapter.
3. Panggilan HTTP/API eksternal dilakukan dari **background service worker** (bebas CORS), bukan content script; content script hanya untuk interaksi DOM halaman.
4. WebSocket untuk realtime: auto-reconnect dengan backoff (maks 5 percobaan), wajib disconnect saat sidebar ditutup / tidak dipakai.
5. API key / token situs dimasukkan user lewat panel **Settings**, disimpan di `chrome.storage.local` (bukan `sync`), DILARANG hardcoded.
6. Rate limit per adapter (default maks 2 request/detik) dan hormati robots/ToS situs; tanpa bypass login/CAPTCHA.

### 5b. Koneksi ke desktop app (Native Messaging)

1. Jalur resmi: **Chrome Native Messaging** (`chrome.runtime.connectNative`), permission `"nativeMessaging"`.
2. Class **`NativeBridge`**: request/response berdasarkan `id` envelope, timeout, auto-reconnect, deteksi host belum terinstall → tampilkan panduan install di sidebar.
3. Repo WAJIB menyertakan **companion host** di `connectors/native-host/`:
   - Satu executable/script ringan (Node.js atau Python, tanpa dependency berat).
   - Protokol stdio native messaging: prefix panjang 4-byte little-endian + JSON, maks 1 MB per pesan (pesan besar di-chunk).
   - Host manifest JSON (`name`, `path`, `type: "stdio"`, `allowed_origins` = ID extension saja).
   - Script install per OS: Windows (registry key), macOS/Linux (folder `NativeMessagingHosts`).
4. Desktop app pihak ketiga yang sudah punya protokol sendiri boleh dihubungkan lewat companion host sebagai penerjemah — extension tetap hanya bicara Native Messaging.
5. Companion host memvalidasi setiap pesan; DILARANG meneruskan perintah apa pun tanpa melewati whitelist (lihat 5c).

### 5c. Koneksi ke terminal / CLI

1. Dua jalur resmi, pilih sesuai kebutuhan:
   - **Via NativeBridge** (5b): companion host mengeksekusi command dan mengembalikan stdout/stderr/exit code.
   - **Via LocalBridge**: WebSocket ke `ws://127.0.0.1:<port>` milik CLI companion di mesin user. DILARANG bind/connect ke alamat selain loopback; wajib **pairing token** (ditampilkan CLI, dimasukkan user sekali di Settings).
2. **Whitelist command WAJIB**: hanya command yang terdaftar di config (mis. `git status`, `npm run build`) yang boleh jalan. Mode "perintah bebas" default OFF, harus diaktifkan sadar oleh user, dan setiap perintah baru butuh konfirmasi eksplisit di sidebar.
3. Eksekusi tanpa shell interpolation: gunakan spawn dengan array argumen, BUKAN string ke `sh -c` / `cmd /c` (cegah command injection). Validasi argumen, batasi working directory.
4. DILARANG default: command destruktif (`rm -rf`, format, dsb.), akses di luar working directory, dan eksekusi berantai tanpa konfirmasi.
5. Setiap eksekusi tercatat di panel "Koneksi": command, waktu, exit code, cuplikan output; bisa di-export sebagai log.
6. Output besar dipotong/stream bertahap (maks 1 MB per pesan native messaging), proses panjang bisa di-**stop** dari sidebar.

### 5d. Keamanan konektivitas (berlaku semua jalur)

1. Semua jalur default **OFF**; user mengaktifkan per connector dari tab "Koneksi".
2. Tidak ada server yang di-expose ke jaringan luar — loopback saja; pairing token untuk LocalBridge, `allowed_origins` ketat untuk native host.
3. Secret (API key, pairing token) hanya di `chrome.storage.local`, tidak pernah di-log, tidak pernah di-commit.
4. Payload dari luar diperlakukan tidak terpercaya: validasi skema envelope, sanitasi sebelum dirender ke DOM (`textContent`).
5. README extension wajib mencantumkan: connector yang dipakai, cara install companion host/CLI, dan alasan tiap permission terkait.

### 5e. Platform Bridge — extension sebagai worker RPA untuk platform (Ingest Token)

Pola: platform tidak bisa login ke situs target / tidak ada API resmi → extension menumpang **sesi login user di browser** untuk RPA, lalu mengirim hasilnya ke platform. Extension = jembatan, bukan API/MCP.

```
Platform ──job──▶ PlatformBridge ──▶ RpaEngine (tab user, sesi login user)
Platform ◀──data── PlatformBridge ◀── hasil extract
```

1. Class **`PlatformBridge`** di `connectors/`, terdaftar di `ConnectionHub`, status tampil di tab "Koneksi".
2. **Konfigurasi user** lewat panel Settings: `Platform URL` + `Ingest Token`. Token disimpan di `chrome.storage.local`, DILARANG hardcoded/commit/di-log. Kirim sebagai header `Authorization: Bearer <token>`.
3. Endpoint **HTTPS wajib** (pengecualian `http://127.0.0.1` hanya untuk development).
4. **Ambil job** dari platform, prioritas jalur:
   - WebSocket / long-poll ke platform (realtime, auto-reconnect backoff), atau
   - Polling via `chrome.alarms` (minimum 30 detik, sesuai aturan performa) ke `GET {platform}/jobs`.
5. **Format job** memakai envelope 5 standar, contoh:
   `{ "id": "...", "type": "request", "action": "linkedin.searchPeople", "payload": { "query": "...", "limit": 20 } }`
   Job dipetakan ke script RPA (JSON steps standar bagian 4) — jadi action baru cukup tambah script, bukan kode baru.
6. **Kirim hasil** ke `POST {platform}/ingest`: envelope + `jobId` + `idempotencyKey` (agar retry tidak dobel data) + status `ok|failed|skipped`. Payload besar dipecah per batch (maks 500 KB per request).
7. **Outbox queue**: hasil yang gagal terkirim disimpan di `chrome.storage.local` dan dikirim ulang dengan backoff; antrian job juga persist agar selamat saat service worker mati.
8. Eksekusi job di tab user dengan pacing manusia (delay antar aksi 300–800 ms, jitter), batas jumlah aksi per jam configurable, dan **pause otomatis** bila situs menunjukkan tanda pemblokiran (CAPTCHA, checkpoint, rate-limit) — laporkan status `blocked` ke platform, JANGAN coba bypass.
9. Job bisa di-**pause/stop** dari sidebar; panel "Koneksi" menampilkan log job (id, action, status, jumlah data terkirim).
10. **Batas etis & risiko (wajib tertulis di README):** hanya data yang terlihat oleh user login tersebut; tanpa menyimpan password situs target; tanpa bypass login/CAPTCHA; data pribadi diminimalkan sesuai kebutuhan job; otomatisasi akun sendiri bisa melanggar ToS situs target (mis. LinkedIn) — risiko pembatasan akun ditanggung dan disadari user.
11. Ingest token di sisi platform sebaiknya **write-only** (hanya boleh kirim data + ambil job, bukan akses penuh akun) dan bisa di-rotate dari Settings tanpa install ulang.

### 5f. LLM Bridge — multi-provider, provider terbaru tinggal tambah class

1. Class **`LlmHub`** (terdaftar di `ConnectionHub`): registri provider, routing per tugas, **fallback chain** (mis. provider A gagal/rate-limit → B → C), dan pencatat pemakaian (token & estimasi biaya) yang tampil di Settings → Koneksi.
2. **Kontrak provider seragam** — provider baru = satu class ≤ 300 LoC di `llm/providers/` dengan API: `listModels()`, `chat(messages, opts)`, `stream(messages, opts)` (SSE), `testConnection()`. Menambah provider terbaru TIDAK boleh mengubah LlmHub/pipeline.
3. Provider bawaan yang wajib didukung arsitekturnya:
   - **OpenAI-compatible generic** (satu class covers OpenAI, Groq, DeepSeek, Mistral, Together, OpenRouter, xAI, dan endpoint kompatibel lain — cukup `baseURL` + `model` configurable),
   - **Anthropic** (format messages API sendiri),
   - **Google Gemini** (format sendiri),
   - **Lokal: Ollama / LM Studio** via `http://127.0.0.1` (privasi penuh, tanpa API key).
4. Semua konfigurasi per provider: `baseURL`, `model`, API key (via SecretVault 7.7), timeout, batas biaya harian. **Multi-provider aktif bersamaan** dan bisa dipilih per tugas (mis. ringkas halaman → provider murah; analisis dokumen → provider pintar; data sensitif → lokal).
5. **PromptLibrary**: template prompt per tugas bisnis (ringkas, ekstrak entitas, klasifikasi, drafting) disimpan sebagai data, bisa diedit user di Settings; hasil LLM masuk **DataPipeline yang sama** (6b) dengan provenance `provider+model`.
6. Streaming ke UI wajib untuk jawaban panjang (render bertahap, bisa di-stop); retry backoff untuk 429/5xx; tanpa blocking UI.
7. **Privasi**: kirim data ke LLM cloud hanya atas aksi/toggle eksplisit user per jenis tugas; tampilkan provider tujuan sebelum kirim pertama kali; opsi "local-only" untuk data sensitif. API key tidak pernah masuk prompt/log.
8. Panggilan LLM dari background service worker; DILARANG fetch API key-bearing dari content script halaman pihak ketiga.

---

## 6. Scraping & Data Pipeline — Panen Apa Pun, Olah, Simpan, Analisis (WAJIB bila fitur diminta)

Prinsip: **RPA-first** — semua scraper bekerja pada DOM halaman yang dibuka user dengan sesinya sendiri, TANPA perlu API resmi dan TANPA perlu MCP. API/MCP hanya jalur tambahan opsional (6d).

### 6a. Jenis scraper (class terpisah di `scrapers/`, terdaftar di `ScraperHub`)

1. **ImageScraper**: kumpulkan `img/srcset`, picture, background-image, dan gambar lazy-load (scroll otomatis via RPA); filter ukuran minimum configurable; nama file dari alt/konteks; unduh massal via FileDownloader atau kemas ZIP.
2. **TextScraper**: teks artikel (reader-mode extraction ringan buatan sendiri, maks 300 LoC — tanpa library Readability penuh), tabel → array of objects, list, dan metadata halaman (judul, penulis, tanggal). Selector pakai SelectorEngine bagian 4.
3. **NovelScraper**: deteksi daftar bab + navigasi multi-halaman otomatis via RpaEngine (next-page, jeda antar halaman), bersihkan boilerplate (iklan/navigasi), dedupe bab, **resume** kalau terputus, export **TXT / Markdown / EPUB** (EPUB builder ringan tanpa dependency berat).
4. **FileDownloader**: unduh file URL langsung via `chrome.downloads` (PDF, arsip, dokumen, media file langsung), antrian + retry + batas paralel (default 3). DILARANG: ripping stream terenkripsi/DRM, memecah HLS/DASH terproteksi.
5. **Scraper umum**: scraper baru = class baru yang mengikuti kontrak seragam `plan() → run() → hasil item[]`, dipetakan ke script RPA JSON bila perlu navigasi. Job panen bisa datang dari sidebar manual ATAU dari platform via PlatformBridge (5e).

### 6b. Pipeline hasil: extract → clean → store → analyze → export/kirim

1. **DataPipeline** adalah satu-satunya jalur hasil scraper; scraper DILARANG menyimpan/mengirim data sendiri.
2. Clean: trim, normalisasi whitespace/encoding, buang duplikat (hash konten), validasi skema item.
3. Setiap item menyimpan provenance: URL sumber, waktu panen, scraper, jobId (bila dari platform).
4. Hasil bisa: (a) disimpan ke library lokal, (b) di-export file, (c) dikirim ke platform via PlatformBridge — bisa kombinasi, dipilih user per job.

### 6c. Penyimpanan & analisis lokal

1. **LibraryStore** pakai **IndexedDB** untuk dataset besar + blob gambar/file (chrome.storage HANYA untuk settings & metadata kecil). Kuota default maks 500 MB, warning di 80%, pembersihan manual dari sidebar.
2. Tab sidebar **"Data"** (masuk rail vertikal): daftar dataset, preview tabel/teks/galeri thumbnail, filter, hapus, dan tombol export/kirim per dataset.
3. **AnalysisService** minimal: jumlah item, ukuran total, deteksi duplikat, distribusi domain/sumber; untuk novel: jumlah bab & kata; untuk tabel: statistik kolom sederhana. Semua komputasi lokal, lazy (hanya saat panel dibuka), tanpa blocking UI (chunk via `requestIdleCallback`).
4. **ExportService**: JSON / CSV / Markdown / TXT / EPUB / ZIP (gambar/file). ZIP builder ringan (store/deflate via `CompressionStream` bawaan browser, tanpa JSZip penuh bila memungkinkan).

### 6d. Konektivitas hybrid — RPA dulu, API/MCP kalau ada (opsional, "gas kalau bisa")

1. Default semua scraper jalan **tanpa API/MCP**. Jangan pernah menjadikan API pihak ketiga sebagai syarat fitur inti.
2. Bila situs target punya **API resmi**: tambahkan mode API di SiteAdapter (5a) — hasil tetap masuk DataPipeline yang sama, sehingga sumber DOM/API bisa ditukar tanpa mengubah penyimpanan/analisis.
3. Bila ingin connect **MCP server**: jalurnya lewat companion (NativeBridge 5b / LocalBridge 5c) yang bertindak sebagai MCP client — extension sendiri TIDAK berbicara MCP langsung. Whitelist tool MCP wajib, default OFF, konfirmasi user per tool baru.
4. Prioritas pemilihan jalur per job: API resmi (paling stabil & ramah situs) → MCP yang diizinkan → RPA DOM (selalu tersedia sebagai fallback).

### 6e. Batas etis & legal (wajib tertulis di README)

1. Hanya konten yang **bisa diakses user itu sendiri** di browser; DILARANG bypass paywall, login, CAPTCHA, atau DRM.
2. Hormati hak cipta: hasil panen (termasuk novel/gambar) untuk penggunaan pribadi/analisis; README wajib memuat disclaimer bahwa redistribusi konten berhak cipta adalah tanggung jawab user.
3. Pacing sopan: jeda antar halaman/unduhan (default 500–1500 ms + jitter), batas paralel, auto-pause saat situs menunjukkan rate-limit/checkpoint (lapor `blocked`, tanpa bypass).
4. Data pribadi pihak lain diminimalkan sesuai kebutuhan job; tanpa exfiltration diam-diam — semua pengiriman ke platform terlihat di log tab "Koneksi".

---

## 7. Keamanan & Privasi (WAJIB)

1. Permission manifest **minimum**: hanya yang benar-benar dipakai; `host_permissions` sesempit mungkin; hindari `<all_urls>` kecuali fitur memang butuh, dan jelaskan alasannya di README.
2. DILARANG: `eval`, remote code, injeksi script dari URL eksternal, exfiltration data.
3. Semua data user disimpan lokal (`chrome.storage`), tidak dikirim ke server kecuali fitur eksplisit dan disetujui user.
4. Sanitasi semua input yang dirender ke DOM (pakai `textContent`, bukan `innerHTML` untuk data user).
5. DILARANG commit secret/token/API key ke repo. Konfigurasi sensitif dimasukkan user lewat panel Settings.
6. Field password tidak pernah di-log/disimpan oleh fitur RPA/recorder.
7. **SecretVault (WAJIB untuk semua secret: API key LLM, ingest token, pairing token)** — class `core/SecretVault.js`:
   - Enkripsi **AES-GCM 256** via WebCrypto; key diturunkan dari passphrase user dengan **PBKDF2 ≥ 600.000 iterasi** (atau device-key non-extractable di IndexedDB untuk mode tanpa passphrase).
   - Ciphertext saja yang masuk `chrome.storage.local`; plaintext secret hanya hidup di memory service worker, **auto-lock** setelah idle (default 30 menit, configurable).
   - UI: field secret masked + tombol lihat/salin; unlock sekali per sesi browser.
8. **Dukungan `.env` (import, bukan baca filesystem)**:
   - Extension TIDAK bisa membaca file `.env` langsung dari disk — jalurnya: user **import file `.env`** lewat Settings → Koneksi (file picker), diparse (`KEY=VALUE`, dukung komentar/kutip), dipetakan ke config (mis. `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `GEMINI_API_KEY`, `PLATFORM_URL`, `INGEST_TOKEN`), lalu disimpan **terenkripsi di SecretVault**. File asli tidak disalin/disimpan.
   - **Export `.env`** opsional dengan peringatan eksplisit (hasil plaintext di file unduhan = tanggung jawab user); default export TIDAK menyertakan secret.
   - Repo wajib menyertakan **`.env.example`** (nama key tanpa nilai) dan `.gitignore` yang memblokir `.env*` (kecuali `.env.example`).
   - Mode companion (NativeBridge/LocalBridge): companion boleh membaca `.env` asli di mesin user (dotenv standar); secret TIDAK dikirim balik ke extension kecuali diminta fitur, dan tidak pernah ke halaman web.
9. **DILARANG**: inject API key saat build ke bundle yang didistribusikan, menaruh secret di `manifest.json`, `chrome.storage.sync`, URL query string, atau log.

---

## 8. UX Flow & Tutorial Bernomor (WAJIB)

1. Setiap extension wajib mengimplementasikan **onboarding flow bernomor** sesuai **[UX_FLOW.md](UX_FLOW.md)** (12 langkah baku, boleh ditambah langkah spesifik produk, tidak boleh dikurangi kecuali fitur terkait tidak ada).
2. Class `OnboardingFlow` menampilkan langkah bernomor `1/12, 2/12, ...` dengan judul + penjelasan singkat + aksi nyata per langkah (bukan cuma teks), bisa **skip**, **back**, dan **resume** (state di storage).
3. **Guided tour** coach-mark bernomor menyorot elemen UI asli (rail tab, tombol fullscreen, panel Data, Settings) — bukan modal terpisah yang tidak nyambung ke UI.
4. Setiap panel utama punya tombol **"?"** yang membuka penjelasan langkah terkait; semua teks penjelasan Bahasa Indonesia, kalimat pendek, tanpa jargon.
5. **First-run demo**: langkah demo menjalankan satu job kecil nyata (mis. panen 3 gambar dari halaman demo lokal yang disertakan) supaya user langsung melihat hasil di tab "Data" tanpa perlu situs target.
6. Tutorial bisa diulang kapan saja dari Settings → Umum → "Mulai ulang tutorial".

---

## 9. Lisensi & Bundling — Bisa Dijual dengan Apa Pun (WAJIB bila dijual)

1. Class **`LicenseService`**: aktivasi via **license key** yang ditandatangani **Ed25519** — verifikasi **offline** memakai public key yang tertanam di extension (tanpa server wajib). Payload key: `edition, bundleId, seats, expiry, issuedTo`.
2. **Editions**: `free` (fitur inti terbatas), `pro` (semua fitur), `bundle-<nama>` (pro + konten bonus), `enterprise` (white-label). Feature gating lewat satu titik (`LicenseService.can(feature)`), bukan cek tersebar.
3. **Bundling dengan apa pun**: buku, video course, template, jasa konsultasi, hardware — mekanismenya sama: kode lisensi dicetak di buku / disisipkan di deskripsi video / QR code → user masukkan sekali di onboarding langkah aktivasi → edition terbuka. `bundleId` mencatat jalur bundling untuk analitik penjual.
4. Aktivasi online opsional (ke platform via 5e) untuk seat management; **offline grace minimal 14 hari** tanpa koneksi; lisensi tidak boleh mengunci data milik user — data lokal tetap bisa di-export walau lisensi habis (fitur panen baru saja yang terkunci).
5. **White-label**: nama, logo, warna, dan link bantuan dari satu file config branding — agar satu extension bisa dijual ulang oleh partner dengan brand mereka tanpa fork kode.
6. Trial default 14 hari fitur pro, tanpa kartu; setelah trial turun ke free dengan pemberitahuan jelas, bukan error mendadak.
7. DILARANG: telemetry diam-diam untuk penegakan lisensi, mengunci export data user, lisensi yang butuh koneksi terus-menerus.
8. README bagian penjualan wajib menjelaskan: editions, cara aktivasi, cara refund/transfer seat, dan kontak support.

---

## 10. Kualitas & Delivery

1. `README.md` tiap extension berisi: fungsi, cara install (load unpacked), daftar permission + alasan, cara install companion host/CLI bila ada connector, dan pernyataan patuh standar Harness v1.5.
2. Tanpa error di console saat alur normal; log pakai satu class `Logger` dengan level, default minim.
3. Versi `manifest.json` mengikuti SemVer; perubahan dicatat di `CHANGELOG.md`.
4. Ikon: SVG sumber + PNG 16/32/48/128, ukuran kecil.
5. Struktur git: commit message jelas; branch utama `main`.

### Checklist acceptance (agent wajib verifikasi sebelum selesai)

- [ ] Tidak ada file > 300 LoC (`wc -l` semua file `.js/.css/.html`)
- [ ] Semua modul berbentuk class, entry MV3 service worker
- [ ] Total ukuran unpacked < 500 KB
- [ ] Sidebar 25% viewport, pinggir kanan, Shadow DOM, tiga mode (rail/sidebar/fullscreen + shortcut), tab vertikal menurun di sisi, panel lazy
- [ ] Full Settings: semua perilaku configurable (tanpa magic number di luar Settings), pencarian, reset, export/import JSON (secret excluded default), test connection
- [ ] Lapisan RPA lengkap (Engine, Selector, ActionLibrary, ScriptStore) + export/import JSON
- [ ] Bila ada konektivitas: ConnectionHub + tab "Koneksi", connector sesuai bagian 5, default OFF, whitelist command, loopback-only, pairing token
- [ ] Bila ada native messaging: companion host + host manifest + script install per OS disertakan
- [ ] Bila ada Platform Bridge: ingest token via Settings, HTTPS, idempotencyKey + outbox queue, auto-pause saat blocked, peringatan ToS di README
- [ ] Bila ada scraper: ScraperHub + DataPipeline satu jalur, LibraryStore IndexedDB, tab "Data" (preview/analisis/export), provenance per item, pacing sopan, disclaimer hak cipta di README
- [ ] Scraper tetap jalan penuh tanpa API/MCP; jalur API/MCP (bila ada) opsional dan masuk pipeline yang sama
- [ ] Bila ada LLM: LlmHub multi-provider + fallback, provider baru = 1 class tanpa ubah hub, streaming, batas biaya, toggle privasi per tugas
- [ ] Semua secret di SecretVault (AES-GCM + PBKDF2), import .env via Settings, .env.example + .gitignore, tanpa secret di bundle/sync/log
- [ ] Onboarding 12 langkah bernomor (UX_FLOW.md) + guided tour + demo first-run + bisa diulang
- [ ] Bila dijual: LicenseService Ed25519 offline, editions + bundleId, trial 14 hari, data user tidak terkunci, white-label config
- [ ] Tidak ada polling liar, eval, CDN eksternal, atau secret di repo
- [ ] Load unpacked di Chrome tanpa error console
- [ ] README + CHANGELOG + manifest SemVer lengkap