# Template Perintah Pembuatan Extension

Salin template ini, isi bagian `[...]`, lalu kirim ke agent bersama GitHub token-mu.

---

## Template dasar

> Buatkan saya browser extension **[NAMA EXTENSION]**.
>
> Fungsi utama: [DESKRIPSI 1–3 KALIMAT, misal: "mengisi formulir web otomatis dari data CSV"].
> Target situs (jika khusus): [URL / "semua situs"].
> Fitur tambahan: [misal: recorder RPA, export hasil ke CSV, jadwal via alarms].
> Konektivitas (jika perlu): [misal: "connect ke API situs X", "connect ke desktop app Y via native messaging", "bisa jalankan command terminal: git, npm"].
>
> **Wajib ikuti seluruh standar di repo `aisac-id/harness-extension`**
> (https://github.com/aisac-id/harness-extension — baca STANDARDS.md dan AGENTS.md dulu).
>
> Buat repo **public** baru `aisac-id/[nama-repo]`, commit, dan push ke `main`.
> Token GitHub saya: [TOKEN — kirim lewat jalur aman, jangan di file].
>
> Sebelum selesai, jalankan checklist acceptance di STANDARDS.md dan laporkan hasilnya.

## Contoh terisi

> Buatkan saya browser extension **Auto Form Filler**.
> Fungsi utama: mengisi formulir web otomatis dari data yang saya tempel di sidebar, satu baris data per submit.
> Target situs: semua situs.
> Fitur tambahan: import CSV, recorder RPA, log hasil per baris.
> Wajib ikuti seluruh standar di repo `aisac-id/harness-extension` (baca STANDARDS.md dan AGENTS.md dulu).
> Buat repo public baru `aisac-id/auto-form-filler`, commit, dan push ke `main`.
> Sebelum selesai, jalankan checklist acceptance dan laporkan hasilnya.

## Tips

- Satu perintah = satu extension. Jangan gabung banyak extension dalam satu perintah.
- Sebutkan target situs kalau extension hanya untuk situs tertentu → permission jadi lebih sempit dan aman.
- Kalau mau menyimpang dari standar (misal butuh library tertentu), tulis eksplisit: "pengecualian standar: boleh pakai library X karena alasan Y".
- Setelah token pernah dibagikan di chat, rotate di GitHub → Settings → Developer settings.