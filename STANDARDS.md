# STANDARDS — Harness Extension v1.0

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

## 5. Keamanan & Privasi (WAJIB)

1. Permission manifest **minimum**: hanya yang benar-benar dipakai; `host_permissions` sesempit mungkin; hindari `<all_urls>` kecuali fitur memang butuh, dan jelaskan alasannya di README.
2. DILARANG: `eval`, remote code, injeksi script dari URL eksternal, exfiltration data.
3. Semua data user disimpan lokal (`chrome.storage`), tidak dikirim ke server kecuali fitur eksplisit dan disetujui user.
4. Sanitasi semua input yang dirender ke DOM (pakai `textContent`, bukan `innerHTML` untuk data user).
5. DILARANG commit secret/token/API key ke repo. Konfigurasi sensitif dimasukkan user lewat panel Settings.
6. Field password tidak pernah di-log/disimpan oleh fitur RPA/recorder.

---

## 6. Kualitas & Delivery

1. `README.md` tiap extension berisi: fungsi, cara install (load unpacked), daftar permission + alasan, dan pernyataan patuh standar Harness v1.0.
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
- [ ] Tidak ada polling liar, eval, CDN eksternal, atau secret di repo
- [ ] Load unpacked di Chrome tanpa error console
- [ ] README + CHANGELOG + manifest SemVer lengkap