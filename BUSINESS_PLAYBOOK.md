# BUSINESS_PLAYBOOK.md — Potensi Extension di Semua Sektor Bisnis

Katalog ide produk yang bisa dibangun di atas Harness Extension.
Semua ide memakai modul standar yang sama: **RPA (4) + Scraper/Pipeline (6) + Platform Bridge (5e) + LLM (5f) + Koneksi desktop/terminal (5b/5c)**.
Pola jual: satu engine, banyak "edition sektoral" (template job + PromptLibrary + branding) — bundling dengan buku/video/template via lisensi (bagian 9).

Legenda modul: `[S]` scraper, `[R]` RPA, `[P]` platform bridge, `[L]` LLM, `[D]` desktop/terminal.

---

## 1. Notaris & PPAT
- **Asisten pemberkasan akta** `[R][L]`: user buka draft akta di Google Docs/Word Online → extension ekstrak pihak, objek, tanggal (LLM) → checklist kelengkapan dokumen (KTP, NPWP, KK, sertifikat) tampil di sidebar; kurang apa, kelihatan.
- **Pengisian berulang ke portal** `[R]`: isi form yang sama berulang di portal pemerintah (AHU, BPN, OSS) dari data klien yang tersimpan — RPA di sesi login user, tanpa simpan password.
- **Pemantau status permohonan** `[R][P]`: cek berkala status di portal (pacing sopan, auto-pause), perubahan status dikirim ke platform kantor → staf tidak perlu refresh manual.
- **Ekstraksi dokumen scan** `[S][L]`: panen teks dari dokumen yang dibuka di browser → LLM jadikan data terstruktur (nama, NIK, alamat) → export CSV ke sistem kantor.
- **Bundling**: buku "Template Akta Lengkap" + video tutorial → kode Pro di halaman akhir buku.

## 2. Hukum / Firma Pengacara
- **Pemantau perkara SIPP pengadilan** `[R][P]`: jadwal cek status perkara per nomor, notifikasi perubahan ke platform firma.
- **Riset putusan** `[S][L]`: panen direktori putusan → ringkas + ekstrak pasal/pertimbangan (LLM) → library riset terindeks.
- **Review kontrak cepat** `[L]`: sorot teks kontrak di halaman → analisis klausul berisiko dengan PromptLibrary khusus legal (opsi LLM lokal untuk kerahasiaan).
- **Bundling**: video course "Riset Hukum Digital" + edition legal.

## 3. HR / Rekrutmen
- **Pemanen kandidat** `[R][S][P]`: cari profil di LinkedIn/job portal memakai sesi login recruiter → data terstruktur ke platform ATS milik perusahaan (pola 5e, pacing anti-banned).
- **Screening CV** `[S][L]`: panen CV yang dibuka di browser → skor kecocokan terhadap job description (LLM), alasan per kandidat, export CSV.
- **Penyebar lowongan** `[R]`: isi form posting lowongan yang sama ke beberapa job portal dari satu data master.
- **Bundling**: buku "Rekrutmen Modern" + template job description.

## 4. Properti / Real Estate
- **Pemantau harga listing** `[S][R][P]`: panen listing marketplace properti per area → analisis harga/m² dan tren → kirim ke platform agency.
- **Auto-posting listing** `[R]`: satu data properti → isi form di beberapa portal iklan.
- **Ringkas dokumen properti** `[S][L]`: panen teks sertifikat/IMB yang dibuka → ekstrak luas, lokasi, status hukum.
- **Bundling**: video "Jualan Properti Online" + edition agency.

## 5. E-commerce / Retail
- **Intelijen harga kompetitor** `[S][P]`: panen harga+stok produk kompetitor terjadwal → dashboard platform → alert saat harga berubah.
- **Analisis review** `[S][L]`: panen review produk sendiri/kompetitor → sentimen + keluhan terbanyak (LLM) → laporan mingguan.
- **Upload produk massal** `[R]`: dari CSV master → isi form produk di marketplace satu per satu dengan pacing aman.
- **Panen aset katalog** `[S]`: gambar produk + deskripsi dari situs supplier → ZIP siap upload.
- **Bundling**: buku "Jago Marketplace" + template CSV produk.

## 6. Keuangan / Akuntansi / Pajak
- **Ekstraksi invoice & receipt** `[S][L]`: panen invoice yang dibuka di webmail/portal → LLM ekstrak vendor, tanggal, nominal, pajak → CSV untuk software akuntansi.
- **Asisten portal pajak** `[R]`: bantu isi form berulang di e-Faktur/e-Bupot web dari data yang sudah disiapkan (user tetap review & submit).
- **Rekap mutasi** `[S]`: panen tabel mutasi e-banking (sesi user sendiri) → normalisasi → CSV/JSON untuk rekonsiliasi. Data sensitif: default LLM lokal / tanpa LLM.
- **Bundling**: video "Otomasi Pembukuan UMKM" + template pembukuan.

## 7. Asuransi
- **Pembanding premi** `[S][L]`: panen quote dari beberapa portal → tabel perbandingan apples-to-apples (LLM normalisasi istilah).
- **Ringkas polis** `[S][L]`: panen teks polis → ringkasan manfaat/pengecualian bahasa awam untuk nasabah.
- **Input aplikasi massal** `[R]`: isi form aplikasi dari data calon nasabah terstruktur.
- **Bundling**: buku panduan polis + edition agen.

## 8. Kesehatan / Klinik (privasi ketat)
- **Ringkas literatur medis** `[S][L]`: panen artikel jurnal yang diakses sah (langganan institusi) → ringkasan + sitasi; LLM lokal untuk data pasien.
- **Asisten administrasi klaim** `[R]`: bantu isi form klaim BPJS/asuransi web dari data terstruktur, staf tetap verifikasi.
- **Pemantau antrean/portal** `[R][P]`: status antrean/jadwal di portal RS → kirim ke sistem klinik.
- Catatan: data pasien TIDAK boleh ke LLM cloud; edition kesehatan default local-only.
- **Bundling**: video manajemen klinik + edition klinik.

## 9. Pendidikan / Kursus / Guru
- **Pemanen bahan ajar** `[S]`: artikel, gambar, novel domain publik untuk bahan ajar → library + EPUB (hak cipta: pribadi/kelas, disclaimer wajib).
- **Generator kuis** `[S][L]`: panen teks materi → LLM buat soal pilihan ganda + kunci → export Markdown/CSV ke LMS.
- **Asisten LMS** `[R]`: input nilai/rekap kehadiran berulang dari CSV ke LMS web sekolah.
- **Bundling**: buku ajar + kode edition guru di sampul belakang.

## 10. Logistik / Ekspedisi
- **Lacak multi-kurir** `[R][S][P]`: panen status resi dari berbagai portal kurir → satu tabel di platform.
- **Ekstraksi manifest** `[S][L]`: panen daftar pengiriman dari portal marketplace → normalisasi alamat (LLM) → CSV manifest.
- **Input order massal** `[R]`: dari CSV → isi form booking pickup di portal kurir.
- **Bundling**: template manifest + video operasional ekspedisi.

## 11. F&B / Restoran
- **Analisis review restoran** `[S][L]`: panen review Google Maps/portal food delivery → sentimen menu per item → laporan perbaikan.
- **Pantau harga supplier** `[S][P]`: panen harga bahan dari katalog supplier online → alert kenaikan.
- **Menu intel** `[S]`: panen menu+harga kompetitor → tabel perbandingan.
- **Bundling**: buku "Kelola Restoran" + edition F&B.

## 12. Travel / Pariwisata
- **Pemantau tarif** `[S][P]`: panen harga tiket/hotel per tanggal → tren & alert ke platform travel agent.
- **Penyusun itinerary** `[S][L]`: panen artikel wisata → LLM susun itinerary per hari → export PDF-ready Markdown.
- **Bundling**: ebook destinasi + kode Pro.

## 13. Marketing / Agency / SEO
- **Audit halaman instan** `[S][L]`: panen meta, heading, konten, gambar alt → skor SEO + rekomendasi (LLM).
- **Panen konten kompetitor** `[S]`: artikel/landing page kompetitor → library analisis Angle & CTA.
- **Generator copy** `[L]`: dari data produk hasil panen → caption/iklan multi-varian dengan PromptLibrary agency.
- **Reporting ke klien** `[P]`: hasil audit terjadwal → kirim ke platform agency → laporan white-label.
- **Bundling**: video course SEO + template laporan klien.

## 14. Media / Penerbitan / Penulis
- **Riset mendalam** `[S][L]`: panen sumber → ringkas + klaster tema → outline artikel/buku.
- **Pemanen novel pribadi** `[S]`: baca-later offline dari situs novel (penggunaan pribadi, export EPUB).
- **Arsip gambar berlisensi** `[S]`: panen aset dari situs stok berlisensi milik user + catat provenance lisensi per file.
- **Bundling**: buku menulis + edition penulis (ini bundling paling natural!).

## 15. Otomotif
- **Pantau harga kendaraan bekas** `[S][P]`: panen listing per model/tahun → valuasi & tren ke platform dealer.
- **Auto-posting iklan** `[R]`: satu data mobil → banyak portal jual-beli.
- **Ekstraksi spesifikasi** `[S][L]`: panen halaman spesifikasi → tabel perbandingan varian.
- **Bundling**: video "Jual Beli Mobil Bekas" + edition dealer.

## 16. Pertanian / Perikanan
- **Pantau harga komoditas** `[S][P]`: panen harga pasar/komoditas harian → tren ke platform koperasi.
- **Katalog supplier** `[S]`: panen katalog bibit/pakan → perbandingan harga.
- **Bundling**: penyuluhan digital + edition koperasi tani.

## 17. Konstruksi / Procurement
- **Pemantau tender LPSE** `[R][S][P]`: panen pengumuman tender per kata kunci → filter nilai/pagu → kirim ke platform kontraktor.
- **Ekstraksi BoQ/RAB** `[S][L]`: panen tabel dokumen tender yang dibuka → CSV RAB siap olah.
- **Harga material** `[S]`: panen katalog material bangunan → perbandingan supplier.
- **Bundling**: template RAB + video tender.

## 18. Manufaktur / Trading B2B
- **Sourcing supplier** `[S][L]`: panen katalog B2B → normalisasi spesifikasi & MOQ (LLM) → shortlist.
- **Pantau harga bahan** `[S][P]`: indeks harga bahan baku terjadwal.
- **Bundling**: direktori supplier + edition trading.

## 19. Administrasi / Layanan Publik / Desa
- **Asisten pengisian formulir layanan** `[R]`: isi form permohonan berulang dari data warga terstruktur (user review & submit sendiri).
- **Pemantau status permohonan** `[R][P]`: cek status berkala → laporan ke sistem kantor.
- **Bundling**: SOP digital + edition instansi (white-label).

## 20. Freelancer / Konsultan
- **Pemanen job board** `[S][L]`: panen lowongan proyek → skor kecocokan skill → shortlist harian.
- **Generator proposal** `[L]`: dari deskripsi job hasil panen → draft proposal personal.
- **Rekap invoice klien** `[S][L]`: panen invoice di webmail → rekap pendapatan bulanan CSV.
- **Bundling**: ebook freelancing + kode Pro (model bundling termurah untuk mulai).

---

## Cara memakai playbook ini (untuk pemilik repo)

1. Pilih satu sektor + satu ide paling menyakitkan (painkiller, bukan vitamin).
2. Perintah ke agent: "Buat extension [nama] untuk [sektor], fitur [ide dari playbook], ikuti aisac-id/harness-extension, edition sektoral: template job + PromptLibrary khusus [sektor]".
3. Engine sama untuk semua → biaya bikin produk kedua dan seterusnya jauh lebih murah; yang diganti hanya template job, prompt, branding, dan konten bundling.
4. Jual sebagai: extension standalone (Free/Pro), bundle buku/video (kode di konten), atau white-label ke asosiasi/instansi sektoral.
5. Semua tetap tunduk pada batas standar: tanpa bypass login/CAPTCHA/paywall/DRM, pacing sopan, data sensitif LLM lokal, disclaimer ToS & hak cipta di README.