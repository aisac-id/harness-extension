# Operation — Harness Extension

Knowledge base komersial dan operasional untuk **Harness Extension**, standar pembuatan browser
extension untuk akun GitHub [`aisac-id`](https://github.com/aisac-id/harness-extension), diterbitkan
oleh [PT. Sainskerta Solusi Nusantara](https://sainskerta.net) — software house & AI partner berbasis
di Gedung Graha Mampang Lt. 3, Jakarta Selatan, beroperasi sejak 2013.

Harness Extension **bukan aplikasi end-user**. Repo ini adalah kontrak engineering: `STANDARDS.md`
v1.5 (10 bagian + checklist acceptance 18 butir), `AGENTS.md` (16 instruksi untuk agen/AI),
`PROMPT_TEMPLATE.md`, `UX_FLOW.md` (onboarding 12 langkah), `BUSINESS_PLAYBOOK.md` (katalog ide 20
sektor bisnis), kit merek di `brand/` dan `promo/`, serta halaman jual statis di `landing/index.html`.
Karena itu posisi komersial yang dipakai di seluruh suite ini adalah **software house enablement**:
lisensi standar + kit merek + playbook, jasa pembangunan extension di atas standar tersebut, dan
lisensi mandiri untuk developer indie.

Seluruh dokumen disusun dari isi repo yang terbaca (`README.md`, `STANDARDS.md`, `AGENTS.md`,
`UX_FLOW.md`, `PROMPT_TEMPLATE.md`, `BUSINESS_PLAYBOOK.md`, `brand/README.md`, `landing/index.html`,
`.env.example`, `.gitignore`) dan dari riset kompetitor nyata yang URL-nya dicantumkan di tiap
dokumen.

> **Catatan penting.** Harga, target pasar, porsi pipeline, proyeksi pendapatan, nilai RTO/RPO,
> ambang SLA, dan perkiraan ROI dalam dokumen ini berstatus **usulan — perlu validasi manajemen**.
> Beberapa prasyarat komersial **belum ada di repo** dan ditandai jujur di dokumen terkait:
> implementasi rujukan (kode extension), CI pemeriksa 300 baris/500 KB, perkakas penerbit kunci
> lisensi Ed25519 beserta prosedur kuncinya, test suite, listing resmi Chrome Web Store, serta
> buku/video untuk jalur bundling.

## 01-Business-Strategy

| Dokumen | Isi Singkat |
|---|---|
| [01-Business-Model-Canvas](01-Business-Strategy/01-Business-Model-Canvas.docx) | Sembilan blok model bisnis: jasa, lisensi enablement, ritel indie |
| [02-ICP-dan-Segmentasi-Pasar](01-Business-Strategy/02-ICP-dan-Segmentasi-Pasar.docx) | Tiga segmen prioritas, kriteria kualifikasi, dan anti-ICP |
| [03-Analisis-Kompetitor](01-Business-Strategy/03-Analisis-Kompetitor.docx) | Empat arena: framework, alat otomasi, RPA enterprise, jasa |
| [04-Analisis-SWOT](01-Business-Strategy/04-Analisis-SWOT.docx) | Kekuatan kontrak mutu, kelemahan ketiadaan implementasi |

## 02-Sales

| Dokumen | Isi Singkat |
|---|---|
| [05-Proposal-Penjualan-B2B-AgensiDigital](02-Sales/05-Proposal-Penjualan-B2B-AgensiDigital.docx) | Proposal agensi/software house: lisensi enablement + jasa |
| [05-Proposal-Penjualan-B2C-DeveloperIndie](02-Sales/05-Proposal-Penjualan-B2C-DeveloperIndie.docx) | Proposal developer indie: lisensi mandiri standar + kit |
| [06-Statement-of-Work](02-Sales/06-Statement-of-Work.docx) | Tujuh fase, kriteria terima terikat, termin, dan tarif |
| [07-Pricing-dan-Packaging](02-Sales/07-Pricing-dan-Packaging.docx) | Struktur paket, harga usulan, diskon, dan penjaga margin |
| [08-Sales-Playbook-dan-Battlecard](02-Sales/08-Sales-Playbook-dan-Battlecard.docx) | Siklus jual, kualifikasi, battlecard, penanganan keberatan |
| [09-Product-Datasheet](02-Sales/09-Product-Datasheet.docx) | Spesifikasi standar, modul, batas etis, dan status implementasi |

## 03-Pre-Sales

| Dokumen | Isi Singkat |
|---|---|
| [10-Discovery-Questionnaire](03-Pre-Sales/10-Discovery-Questionnaire.docx) | Delapan bagian pertanyaan dan daftar penghenti peluang |
| [11-Demo-Script-dan-PoC-Plan](03-Pre-Sales/11-Demo-Script-dan-PoC-Plan.docx) | Naskah demo 30 menit dan PoC 10 hari kerja |
| [12-Technical-Proposal](03-Pre-Sales/12-Technical-Proposal.docx) | Peta modul, kontrak integrasi, nonfungsional, uji |
| [13-ROI-dan-Business-Case](03-Pre-Sales/13-ROI-dan-Business-Case.docx) | Model manfaat pelanggan dan kelayakan investasi internal |

## 04-IT-Communication

| Dokumen | Isi Singkat |
|---|---|
| [14-Arsitektur-Solusi](04-IT-Communication/14-Arsitektur-Solusi.docx) | Konteks MV3, komponen, aliran data, pandangan penyerahan |
| [15-Arsitektur-Keamanan-dan-Threat-Model](04-IT-Communication/15-Arsitektur-Keamanan-dan-Threat-Model.docx) | Aset, lima belas ancaman nyata, dan kesenjangan yang diakui |
| [16-Privasi-Data-dan-Kepatuhan](04-IT-Communication/16-Privasi-Data-dan-Kepatuhan.docx) | Pemetaan pemrosesan ke UU PDP 27/2022 dan kebijakan toko |

## 05-Resilience-Operations

| Dokumen | Isi Singkat |
|---|---|
| [17-Business-Continuity-Plan](05-Resilience-Operations/17-Business-Continuity-Plan.docx) | Kontinuitas repo, akun toko, kunci, dan penyampaian jasa |
| [18-Disaster-Recovery-Plan](05-Resilience-Operations/18-Disaster-Recovery-Plan.docx) | RPO/RTO aset rilis dan prosedur pemulihan bertahap |
| [19-SOP-Insiden-dan-Breach-Response](05-Resilience-Operations/19-SOP-Insiden-dan-Breach-Response.docx) | Enam tahap siklus insiden, buku resep, dan RACI |
| [20-SLA-dan-Model-Dukungan](05-Resilience-Operations/20-SLA-dan-Model-Dukungan.docx) | Lima tingkatan dukungan, waktu respons, dan pengecualian |

## Cara Membaca Suite Ini

1. Mulai dari `01-Business-Strategy` untuk memahami bentuk produk (standar + kit + jasa) dan batas
   yang tidak bisa dinegosiasi: tanpa melewati CAPTCHA, login, paywall, atau DRM.
2. Lanjut ke `02-Sales` untuk materi yang dipakai langsung di depan calon pembeli, termasuk `09`
   sebagai lampiran teknis.
3. Gunakan `03-Pre-Sales` saat menyiapkan discovery, demo, PoC, dan usulan teknis.
4. Baca `04-IT-Communication` bersama tim keamanan dan peninjau kepatuhan pelanggan.
5. Tutup dengan `05-Resilience-Operations` untuk kontrak dukungan dan kesiapan insiden.

Tiga keputusan yang berulang di seluruh dokumen dan perlu diambil manajemen sebelum komersialisasi:
(1) mendanai implementasi rujukan beserta CI dan test suite, (2) menetapkan daftar harga resmi
menggantikan angka contoh di `landing/index.html`, dan (3) menyahkan prosedur kunci lisensi Ed25519
sebelum penjualan pertama.

## Penerbit

PT. Sainskerta Solusi Nusantara — Gedung Graha Mampang Lt. 3, Jakarta Selatan
hello@sainskerta.id · +62 813-5988-3131 · sainskerta.net

Dokumen bersifat rahasia dan hanya untuk pemakaian internal serta prospek bisnis yang
telah menandatangani perjanjian kerahasiaan.

© 2026 PT. Sainskerta Solusi Nusantara · hello@sainskerta.id · sainskerta.net
