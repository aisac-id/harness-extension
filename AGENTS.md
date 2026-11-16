# AGENTS.md — Instruksi untuk AI/Agent

Jika kamu agent yang diminta membuat extension dengan mengacu repo ini:

1. **Baca `STANDARDS.md` sampai selesai sebelum menulis kode.** Semua aturan di sana mengikat.
2. Jangan menyalin repo ini mentah-mentah; buat repo baru sesuai nama yang diminta user, dengan struktur folder dari STANDARDS.md bagian 1.
3. Setiap class yang kamu buat: cek `wc -l` <= 300 sebelum commit. Jika lebih, pecah.
4. Wajib ada lapisan `rpa/` lengkap walau extension-nya bukan tool RPA — itu fondasi bersama.
5. UI wajib: sidebar kanan 25% viewport (min 280px, max 420px), Shadow DOM, collapsible, tab vertikal menurun di sisi kiri sidebar, panel lazy.
6. Vanilla JS ES Modules + class. Tanpa React/Vue, tanpa CDN, tanpa eval, tanpa polling liar.
7. Jalankan **Checklist acceptance** di akhir STANDARDS.md dan laporkan hasilnya ke user (centang yang lolos, jelaskan yang tidak).
8. Jangan commit token/secret apa pun. Gunakan token hanya untuk membuat repo & push sesuai perintah user.
9. Bahasa UI dan README: Bahasa Indonesia, kecuali user minta lain.
10. Jika ada konflik antara permintaan user dan standar ini, tanyakan user dulu — jangan diam-diam melanggar standar.