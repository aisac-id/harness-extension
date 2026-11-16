# UX_FLOW.md — Onboarding & Tutorial Baku (12 Langkah)

Wajib diimplementasikan class `OnboardingFlow` (bagian 8 STANDARDS.md).
Setiap langkah ditampilkan sebagai `N/12` + judul + penjelasan 1–3 kalimat + **aksi nyata**.
Semua langkah bisa: **Lewati**, **Kembali**, dan **lanjut nanti** (resume otomatis saat dibuka lagi).
Target: user paham dan melihat hasil nyata dalam < 5 menit.

---

## Langkah 1/12 — Selamat datang & nilai produk
- **UI**: layar sambutan di sidebar (bukan fullscreen), logo + satu kalimat nilai produk ("Panen data dari halaman mana pun, olah, dan kirim ke sistem Anda — tanpa API").
- **Aksi**: tombol "Mulai (2 menit)" dan "Lewati tutorial".
- **Penjelasan ke user**: apa yang extension ini lakukan dan apa yang TIDAK (tidak bypass login, tidak simpan password).
- **Selesai bila**: user klik Mulai atau Lewati (state tersimpan).

## Langkah 2/12 — Bahasa & tema
- **UI**: pilih Bahasa (default Indonesia) dan tema (Terang/Gelap/Ikut Sistem).
- **Aksi**: perubahan langsung terlihat tanpa reload.
- **Selesai bila**: user klik Lanjut (nilai default boleh dipakai).

## Langkah 3/12 — Lisensi / trial
- **UI**: tiga pilihan: "Coba Pro 14 hari (tanpa kartu)", "Masukkan kode lisensi" (dari buku/video/bundle — ada contoh gambar letak kode), "Lanjut dengan Free".
- **Aksi**: input kode → validasi offline Ed25519 → badge edition muncul di header.
- **Penjelasan**: apa beda Free vs Pro, dan bahwa data user tetap miliknya apa pun edition-nya.
- **Selesai bila**: edition aktif (trial/free/lisensi).

## Langkah 4/12 — Kenali tampilan (guided tour bernomor)
- **UI**: coach-mark menyorot elemen asli berurutan: (4a) rail tab vertikal, (4b) tombol toggle rail/sidebar, (4c) tombol fullscreen, (4d) tab Data, (4e) tab Koneksi, (4f) tab Settings di bawah.
- **Aksi**: user klik "Berikutnya" pada tiap coach-mark; boleh mencoba toggle sungguhan di tengah tur.
- **Selesai bila**: semua sub-langkah dilihat atau di-skip.

## Langkah 5/12 — Buka sidebar & fullscreen di halaman nyata
- **UI**: instruksi "Buka situs mana saja, tekan Alt+Shift+S lalu Alt+Shift+F".
- **Aksi**: extension mendeteksi halaman aktif dan menampilkan konfirmasi "Sidebar siap di halaman ini".
- **Selesai bila**: user berhasil menampilkan sidebar di satu halaman (atau skip).

## Langkah 6/12 — Demo panen pertama (hasil nyata!)
- **UI**: tombol "Jalankan demo" → extension membuka halaman demo lokal yang disertakan di package (`demo/index.html`, berisi gambar + artikel + tabel contoh).
- **Aksi**: job demo menjalankan ImageScraper + TextScraper sungguhan, progress terlihat langkah demi langkah (log RPA mini).
- **Penjelasan**: "Inilah cara semua scraper bekerja: melihat halaman seperti Anda, bukan lewat API."
- **Selesai bila**: minimal 3 item masuk library.

## Langkah 7/12 — Lihat & olah hasil di tab Data
- **UI**: tab Data terbuka otomatis menampilkan hasil demo: preview galeri/teks, jumlah item, ukuran, deteksi duplikat.
- **Aksi**: user mencoba satu filter dan membuka satu item (fullscreen untuk tabel/galeri).
- **Selesai bila**: user membuka detail satu item.

## Langkah 8/12 — Export hasil
- **UI**: tombol Export pada dataset demo → pilih format (JSON/CSV/ZIP/TXT/EPUB sesuai isi).
- **Aksi**: file benar-benar terunduh; toast sukses + lokasi unduhan.
- **Penjelasan**: hasil bisa dipakai di Excel, buku, atau sistem lain.
- **Selesai bila**: satu file ter-export (atau skip).

## Langkah 9/12 — Sambungkan LLM (opsional, bisa diulang di Settings)
- **UI**: pilih provider (OpenAI-compatible / Anthropic / Gemini / Ollama lokal) → tempel API key ATAU tombol **"Import file .env"**.
- **Aksi**: test connection → indikator hijau; key langsung masuk SecretVault terenkripsi (jelaskan satu kalimat: "Kunci dienkripsi di perangkat Anda, tidak dikirim ke mana pun selain provider pilihan Anda").
- **Penjelasan**: contoh kegunaan — ringkas artikel hasil panen, ekstrak data dari teks berantakan.
- **Selesai bila**: satu provider terhubung atau user memilih "Nanti saja".

## Langkah 10/12 — Sambungkan platform / konektor lain (opsional)
- **UI**: tab Koneksi: Platform URL + Ingest Token (atau import .env), tombol Test; info singkat connector desktop/terminal bila produk mendukung.
- **Aksi**: test connection → status terhubung; semua connector tetap default OFF sampai user mengaktifkan.
- **Selesai bila**: satu koneksi teruji atau "Nanti saja".

## Langkah 11/12 — Job otomatis pertama (RPA / jadwal)
- **UI**: pilih satu template job sesuai produk (mis. "Pantau harga halaman ini", "Panen gambar halaman ini") → jalankan sekali sekarang.
- **Aksi**: job berjalan dengan pacing sopan; user melihat tombol pause/stop bekerja; hasil masuk Data dan (bila tersambung) terkirim ke platform dengan status di log Koneksi.
- **Penjelasan**: batas aman — auto-pause saat situs menolak, tanpa bypass.
- **Selesai bila**: satu job selesai/blocked-terlapor, user pernah melihat kontrol stop.

## Langkah 12/12 — Selesai & peta bantuan
- **UI**: ringkasan checklist apa saja yang sudah tersambung (lisensi, LLM, platform, job pertama) + tiga jalan lanjutan: "Ulangi tutorial" (Settings → Umum), tombol "?" di tiap panel, dan link dokumentasi/support.
- **Aksi**: tombol "Mulai bekerja" menutup onboarding; badge kecil di toolbar menunjukkan status job selanjutnya.
- **Selesai bila**: onboarding ditandai selesai di storage; tidak muncul lagi kecuali diulang manual.

---

## Aturan umum implementasi

1. Progress `N/12` tersimpan di `chrome.storage.local`; tutup browser di tengah → lanjut dari langkah terakhir.
2. Langkah opsional (9, 10, 11) boleh di-skip tanpa menghalangi langkah berikutnya; status "belum disambung" tampil sebagai banner halus di tab Koneksi, bukan popup berulang.
3. Setiap langkah punya acceptance test sendiri (kolom "Selesai bila") — agent wajib menguji alur 1→12 di Chrome sebelum delivery.
4. Teks final per langkah ditulis untuk orang non-teknis; istilah teknis (RPA, ingest, provider) selalu diberi padanan sederhana ("otomatisasi klik", "kirim data", "penyedia AI").
5. Produk boleh menambah langkah 13+ spesifik sektornya (mis. notaris: "Hubungkan portal SIPP"), ditempatkan setelah langkah 12 sebagai "Tur lanjutan".