# STANDARDS — Harness Extension v1.1

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
│   └── Settings.js
├── rpa/
│   ├── RpaEngine.js          # eksekusi langkah
│   ├── SelectorEngine.js     # resolusi selector + fallback
│   ├── ActionLibrary.js      # click/type/wait/scroll/extract, dll.
│   └── ScriptStore.js        # simpan/export/import script JSON
├── connectors/               # lapisan konektivitas (lihat bagian 5)
│   ├── ConnectionHub.js      # registri + status semua koneksi
│   ├── web/                  # 1 class SiteAdapter per situs/API
│   ├── NativeBridge.js       # native messaging ke desktop app
│   ├── LocalBridge.js        # WebSocket ws://127.0.0.1 (terminal/CLI)
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
3. **Collapsible**: tombol toggle mengubah sidebar jadi rail sempit (±48px) atau tersembunyi; state tersimpan di storage. Toggle juga via shortcut keyboard (default `Alt+Shift+S`).
4. **Tab vertikal**: icon rail menurun di sisi kiri sidebar (bukan tab horizontal di atas).
   - Setiap tab = satu panel class terpisah di `sidebar/panels/`.
   - Panel di-render lazy saat tab pertama kali dibuka.
   - Tab aktif ditandai visual jelas; label muncul saat hover/expanded.
5. Sidebar TIDAK boleh menggeser layout halaman secara default (overlay). Mode "push content" opsional, off by default.
6. Tema: dukung dark/light mengikuti `prefers-color-scheme`, tanpa flash saat load.
7. Semua teks UI Bahasa Indonesia (kecuali diminta lain); aksesibel: focusable, `aria-label` pada icon button, kontras cukup.

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

---

## 6. Keamanan & Privasi (WAJIB)

1. Permission manifest **minimum**: hanya yang benar-benar dipakai; `host_permissions` sesempit mungkin; hindari `<all_urls>` kecuali fitur memang butuh, dan jelaskan alasannya di README.
2. DILARANG: `eval`, remote code, injeksi script dari URL eksternal, exfiltration data.
3. Semua data user disimpan lokal (`chrome.storage`), tidak dikirim ke server kecuali fitur eksplisit dan disetujui user.
4. Sanitasi semua input yang dirender ke DOM (pakai `textContent`, bukan `innerHTML` untuk data user).
5. DILARANG commit secret/token/API key ke repo. Konfigurasi sensitif dimasukkan user lewat panel Settings.
6. Field password tidak pernah di-log/disimpan oleh fitur RPA/recorder.

---

## 7. Kualitas & Delivery

1. `README.md` tiap extension berisi: fungsi, cara install (load unpacked), daftar permission + alasan, cara install companion host/CLI bila ada connector, dan pernyataan patuh standar Harness v1.1.
2. Tanpa error di console saat alur normal; log pakai satu class `Logger` dengan level, default minim.
3. Versi `manifest.json` mengikuti SemVer; perubahan dicatat di `CHANGELOG.md`.
4. Ikon: SVG sumber + PNG 16/32/48/128, ukuran kecil.
5. Struktur git: commit message jelas; branch utama `main`.

### Checklist acceptance (agent wajib verifikasi sebelum selesai)

- [ ] Tidak ada file > 300 LoC (`wc -l` semua file `.js/.css/.html`)
- [ ] Semua modul berbentuk class, entry MV3 service worker
- [ ] Total ukuran unpacked < 500 KB
- [ ] Sidebar 25% viewport, pinggir kanan, Shadow DOM, collapsible
- [ ] Tab vertikal menurun di sisi, panel lazy
- [ ] Lapisan RPA lengkap (Engine, Selector, ActionLibrary, ScriptStore) + export/import JSON
- [ ] Bila ada konektivitas: ConnectionHub + tab "Koneksi", connector sesuai bagian 5, default OFF, whitelist command, loopback-only, pairing token
- [ ] Bila ada native messaging: companion host + host manifest + script install per OS disertakan
- [ ] Tidak ada polling liar, eval, CDN eksternal, atau secret di repo
- [ ] Load unpacked di Chrome tanpa error console
- [ ] README + CHANGELOG + manifest SemVer lengkap