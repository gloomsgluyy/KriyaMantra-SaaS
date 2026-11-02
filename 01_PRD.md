# Kriyamantra — Product Requirements Document (PRD)

**Version:** 1.0
**Tanggal:** November 2026
**Product Owner:** [NAMA_KETUA_TIM / PIC]
**Prepared for:** Tim Pengembang & Designer (Bali DigiFest 2026 — AI Web Innovation Challenge)

---

## 1. Product Vision

### Vision Statement

> Kriyamantra menjadi **pelatih marketing pribadi** bagi perajin tradisional Bali — menggantikan promosi yang tidak rutin, desain yang tidak bisa dibuat sendiri, dan konten yang berhenti di tengah jalan — dalam satu website dengan asisten chat yang **mengerjakan promosi bersama perajin**: membuat poster yang selalu unik, menulis promosi Indonesia/Inggris, memostingnya ke Instagram, dan menjaga kebiasaan lewat tugas mingguan.

### Objectives

1. **Mengerjakan, bukan mengajari** — Perajin cukup mengirim foto dan keterangan produk; AI mengerjakan sisanya
2. **Poster selalu unik** — Setiap poster dibuat AI sendiri dan tidak pernah kembar dengan poster sebelumnya, tetapi identitas brand tetap konsisten
3. **Konsistensi kebiasaan** — Tugas mingguan dan pengingat menjaga promosi tidak berhenti
4. **AI yang mengenal usaha** — Memori per usaha (profil, produk, gaya visual, gaya bahasa) dari sumber yang sudah ada atau dari wawancara terpandu
5. **Kendali di tangan pengguna** — Tidak ada posting tanpa persetujuan eksplisit; semua hasil AI bisa direvisi
6. **Jujur dan aman** — Fakta berlabel sumber, tidak mengarang klaim produk, privasi data terjaga
7. **Inklusif** — Perajin tanpa Instagram/website tetap dapat memakai produk (jalur wawancara + paket siap-upload)

---

## 2. Target Users & Persona

### Persona 1: Perajin / Pelaku UMKM Tradisional (Pengguna Utama)
- **Jabatan:** Pemilik usaha mikro (tenun/endek, ukiran, perak, jamu, kuliner khas Bali)
- **Kebutuhan:** Mempromosikan produk ke pembeli lokal dan turis tanpa harus bisa mendesain atau menulis konten
- **Pain Point:** Tidak punya waktu dan skill desain, tidak konsisten posting, bahasa Inggris terbatas, kembali ke cara lama (titip toko) setelah pelatihan
- **Solusi:** Chat sederhana → poster unik + caption ID/EN → posting otomatis → tugas mingguan yang kecil dan jelas

### Persona 2: Pendamping UMKM (Pengguna Sekunder — Pengembangan Mendatang)
- **Jabatan:** Penyuluh/pendamping dari koperasi, dinas, atau komunitas
- **Kebutuhan:** Memantau banyak perajin dampingan sekaligus
- **Pain Point:** Sulit memastikan perajin terus aktif setelah pelatihan
- **Solusi:** Mode pendamping dengan ringkasan aktivitas per perajin *(P2, di luar Tahap 1)*

### Persona 3: Juri Lomba
- **Jabatan:** Juri Bali DigiFest 2026 AI Web Innovation Challenge
- **Kebutuhan:** Menguji alur utama produk, melihat AI benar-benar bekerja, memeriksa repository dan disclosure
- **Pain Point:** Produk yang tidak bisa diakses, demo yang dipalsukan, AI yang tidak jelas perannya
- **Solusi:** Akun demo khusus juri, alur utama dapat diuji penuh lewat chat di web, mode demo dengan data contoh, dokumentasi dan disclosure jelas

### Persona 4: Tim Internal / Admin Sistem
- **Jabatan:** Ketua tim, developer, designer
- **Kebutuhan:** Memantau penggunaan AI, biaya, kuota, kegagalan posting, dan kesehatan sistem
- **Pain Point:** Biaya API tidak terkontrol, kegagalan integrasi tidak terlihat
- **Solusi:** Log panggilan AI, kuota per usaha, monitor token Instagram, health check

---

## 3. Feature Map

> **Prioritas:** `Very High` dan `High` = Tahap 1 wajib (P0) · `Medium` = Tahap 1 jika waktu cukup (P1) · `Low` = nanti (P2) atau opsional.

### 3.1 Authentication & Akun

| ID | Feature | Priority | Deskripsi |
|----|---------|----------|-----------|
| F-AUTH-01 | Registrasi & Login | Very High | Pembuatan akun dan login; sesi aman |
| F-AUTH-02 | Protected Routes | Very High | Semua halaman aplikasi memerlukan autentikasi, kecuali halaman publik |
| F-AUTH-03 | Isolasi Data Antar-Usaha | Very High | Setiap usaha hanya dapat mengakses datanya sendiri (profil, produk, konten, token) |
| F-AUTH-04 | Akun Demo Juri | High | Akun khusus juri; kredensial dikirim lewat kanal panitia, bukan repository |
| F-AUTH-05 | Rate Limit & Anti-abuse | High | Pembatasan permintaan untuk mencegah penyalahgunaan dan biaya membengkak |

### 3.2 Onboarding

| ID | Feature | Priority | Deskripsi |
|----|---------|----------|-----------|
| F-ONB-01 | Wizard Onboarding Bertahap | Very High | 5 langkah dengan indikator, tombol kembali, dan dapat dilanjutkan nanti |
| F-ONB-02 | Info Usaha | Very High | Nama usaha, jenis produk, lokasi |
| F-ONB-03 | Persetujuan Penggunaan AI | Very High | Penjelasan sederhana data yang dikirim ke AI; wajib disetujui untuk lanjut; tersimpan sebagai log |
| F-ONB-04 | Pilih Sumber Usaha (Opsional) | High | Kartu pilihan: Instagram, Website, Foto katalog, Ceritakan lewat chat, Lewati |
| F-ONB-05 | Review & Konfirmasi Profil Draft | Very High | Pengguna mengonfirmasi/mengoreksi apa yang AI pahami sebelum masuk memori |
| F-ONB-06 | Ajakan Poster Pertama | Medium | Setelah selesai, langsung diarahkan membuat poster pertama (momen "wow" untuk demo) |

### 3.3 Sumber Usaha (Import Opsional)

| ID | Feature | Priority | Deskripsi |
|----|---------|----------|-----------|
| F-SRC-01 | Hubungkan Instagram (OAuth Resmi) | Very High | Membaca konten milik akun sendiri dan memungkinkan posting; satu login untuk keduanya |
| F-SRC-02 | Panduan Konversi Akun Business/Creator | High | Panduan bergambar karena akun personal tidak didukung API resmi |
| F-SRC-03 | Import Caption & Media Instagram | High | Maks. 30 postingan terbaru, sekali saat onboarding, untuk base knowledge |
| F-SRC-04 | Unggah Foto Katalog/Brosur/Nota | High | Ekstraksi isi lewat vision/OCR menjadi data produk dan profil |
| F-SRC-05 | Wawancara Terpandu | Very High | 5-7 pertanyaan seperti ngobrol; jalur dasar bagi perajin tanpa medsos |
| F-SRC-06 | Import Website via URL | Medium | URL milik pengguna; ambil halaman publik secara terbatas, hormati `robots.txt` |
| F-SRC-07 | Cerita Usaha via Voice Note | Medium | Speech-to-text dengan fallback teks |
| F-SRC-08 | Label Sumber & Status Konfirmasi | Very High | Tiap fakta berlabel: Diisi pengguna / Instagram / Website / Katalog / Tebakan AI |
| F-SRC-09 | Proteksi Prompt Injection | Very High | Konten impor diperlakukan sebagai data, bukan instruksi |
| F-SRC-10 | Perlindungan Privasi Import | High | Tidak menyimpan komentar atau foto orang lain; hanya konten milik akun |
| F-SRC-11 | Sinkron Ulang Manual | Low | Pengguna dapat mengimpor ulang sumber yang terhubung |

### 3.4 Profil, Brand Kit & Memori

| ID | Feature | Priority | Deskripsi |
|----|---------|----------|-----------|
| F-MEM-01 | Profil Usaha Terstruktur | Very High | Nama, lokasi, kategori, cerita asal usul, target pembeli, bahasa, gaya bahasa |
| F-MEM-02 | Brand Kit | Very High | Palet, mood, logo, contoh referensi, gaya tipografi (deskriptif) |
| F-MEM-03 | Draft Brand Kit Otomatis | Medium | Dihasilkan dari analisis foto lama; dikonfirmasi pengguna |
| F-MEM-04 | Memori Konten | Very High | Deskriptor dan embedding tiap poster/post untuk cek keunikan dan konteks |
| F-MEM-05 | Konteks Ringkas untuk Prompt | High | Retrieval terbatas, bukan menumpuk seluruh riwayat chat |
| F-MEM-06 | Lihat / Ubah / Hapus Memori | High | Pengguna mengontrol "apa yang AI ketahui tentang usaha ini" |

### 3.5 Katalog Produk

| ID | Feature | Priority | Deskripsi |
|----|---------|----------|-----------|
| F-CAT-01 | Daftar Produk & Status Promosi | High | Foto, nama, status belum/sudah pernah dipromosikan |
| F-CAT-02 | Tambah / Edit Produk | High | Nama, deskripsi, harga, foto |
| F-CAT-03 | Produk Otomatis dari Chat/Import | Medium | Produk baru terbentuk dari foto + keterangan atau dari sumber usaha |

### 3.6 Chat Agent (Web)

| ID | Feature | Priority | Deskripsi |
|----|---------|----------|-----------|
| F-CHAT-01 | Chat Web | Very High | Percakapan dengan agent; pintu masuk utama dan kanal yang diuji juri |
| F-CHAT-02 | Unggah Foto Produk | Very High | 1-5 foto per permintaan, dengan validasi ukuran/format/kualitas |
| F-CHAT-03 | Kartu Opsi Poster & Tombol Cepat | Very High | Carousel 2-3 opsi; tombol Setujui / Revisi / Ubah jadwal |
| F-CHAT-04 | Status Memproses Asinkron | Very High | Progres jujur, dapat ditinggal, pengguna diberi tahu saat selesai |
| F-CHAT-05 | Pertanyaan Data Kurang | High | Maks. 2 pertanyaan sekaligus (harga, ukuran, dll.) |
| F-CHAT-06 | Pembatasan Fungsi Marketing | High | Di luar konteks dijawab singkat dan diarahkan kembali (bukan asisten serba-bisa) |
| F-CHAT-07 | Rekam Voice Note | Medium | Detail produk lewat suara, ditranskripsi lalu dikonfirmasi |
| F-CHAT-08 | Riwayat Percakapan | High | Konsisten lintas sesi |
| F-CHAT-09 | Agent Core Lintas Kanal | Very High | Satu agent dan database dengan adapter kanal (Web; WhatsApp opsional) |

### 3.7 Creative Engine (Generate Konten)

| ID | Feature | Priority | Deskripsi |
|----|---------|----------|-----------|
| F-GEN-01 | Creative Brief Generator | Very High | Brief baru tiap permintaan dari produk, brand kit, riwayat poster, dan momen |
| F-GEN-02 | Rotasi Sudut Kreatif | Very High | Close-up tekstur, proses pembuatan, cerita budaya, suasana pemakaian, promo; tidak sama dengan poster-poster terakhir |
| F-GEN-03 | Generator Poster | Very High | 2-3 opsi dari model gambar dengan foto produk sebagai referensi dan brand kit sebagai batasan |
| F-GEN-04 | Quality Gate | Very High | Cek teks (OCR), kesetiaan produk, kesesuaian brand, artefak; retry terbatas |
| F-GEN-05 | Cek Keunikan | Very High | Bandingkan dengan poster sebelumnya; terlalu mirip → regenerate |
| F-GEN-06 | Caption & Hashtag Indonesia | Very High | Gaya bahasa sesuai profil usaha |
| F-GEN-07 | Terjemahan Inggris | High | Caption Inggris setara, tampil sebagai tab terpisah |
| F-GEN-08 | Format Output | High | Feed 4:5 dan Story 9:16 dengan safe area |
| F-GEN-09 | Fallback Template Bertanda | Medium | Hanya bila quality gate gagal berulang; ditandai jelas, bukan jalur utama |
| F-GEN-10 | Guardrail Klaim Produk | Very High | Tidak mengarang klaim (asli, organik, halal, bersertifikat, medis); harga hanya dari input |
| F-GEN-11 | Kuota & Batas Generate | High | Per usaha per hari/jam untuk mengontrol biaya |

### 3.8 Review, Revisi & Persetujuan

| ID | Feature | Priority | Deskripsi |
|----|---------|----------|-----------|
| F-REV-01 | Pilih Opsi & Pratinjau Gaya Instagram | Very High | Pratinjau feed dan story sebelum disetujui |
| F-REV-02 | Revisi Bahasa Alami | Very High | Kolom instruksi + chip cepat ("lebih hangat", "tulisan lebih besar") |
| F-REV-03 | Edit Caption Manual | High | Tab Indonesia/Inggris, hashtag dapat ditambah/dihapus |
| F-REV-04 | Riwayat Versi | High | Semua versi poster/caption tersimpan dan dapat dibuka |
| F-REV-05 | Persetujuan Eksplisit | Very High | Tidak ada posting tanpa persetujuan pengguna |
| F-REV-06 | Batas Revisi | Medium | Maks. beberapa kali per poster; pesan ramah saat hampir habis |
| F-REV-07 | Penanda "Dibuat AI" | High | Terlihat jelas tetapi tidak mengganggu; patuhi kebijakan platform |

### 3.9 Penjadwalan & Auto-post Instagram

| ID | Feature | Priority | Deskripsi |
|----|---------|----------|-----------|
| F-SCH-01 | Usulan Jadwal | High | Aturan awal (jam umum, jarak antar posting), dapat diubah |
| F-SCH-02 | Ubah / Batalkan Jadwal | High | Pengguna selalu dapat membatalkan |
| F-SCH-03 | Auto-post ke Instagram | Very High | **Wajib di Tahap 1**; posting terbit sesuai jadwal setelah persetujuan |
| F-SCH-04 | Scheduler / Queue Internal | Very High | Penjadwalan dijalankan sistem sendiri, tidak bergantung fitur jadwal platform |
| F-SCH-05 | Status Posting | Very High | Draf → Disetujui → Terjadwal → Terbit / Gagal |
| F-SCH-06 | Retry & Penanganan Gagal | Very High | Retry terbatas, notifikasi, jalur manual |
| F-SCH-07 | Monitor & Perpanjang Token Instagram | Very High | Token berumur terbatas: perpanjangan otomatis dan peringatan otorisasi ulang |
| F-SCH-08 | Paket Siap-Upload | High | Gambar + caption + hashtag untuk diunduh; jalur setara bagi yang tanpa Instagram dan pengganti TikTok |
| F-SCH-09 | Kalender Jadwal (UI) | High | Tampilan kalender dan daftar dengan lencana status |

### 3.10 Tugas Mingguan & Pengingat

| ID | Feature | Priority | Deskripsi |
|----|---------|----------|-----------|
| F-TSK-01 | Generator Tugas Mingguan | Very High | Tugas kecil dan spesifik (promosikan produk yang belum pernah diposting, cerita di balik produk, dll.) |
| F-TSK-02 | Daftar Tugas & Progres | Very High | Tugas minggu ini dan progres; riwayat minggu lalu |
| F-TSK-03 | Aksi Tugas | High | Selesai / Nanti / Lewati |
| F-TSK-04 | Tugas → Alur Buat Konten | High | Tombol Mulai membuka alur dengan konteks terisi |
| F-TSK-05 | Pengingat Terbatas | High | 1-2 per minggu lewat notifikasi di aplikasi; kanal WhatsApp hanya jika modul opsional aktif |

### 3.11 Dashboard

| ID | Feature | Priority | Deskripsi |
|----|---------|----------|-----------|
| F-DASH-01 | Ringkasan Minggu Ini | High | Tugas (x dari y), jadwal terdekat, posting terbit |
| F-DASH-02 | Aksi Utama "Buat Konten Baru" | High | Tombol besar di beranda |
| F-DASH-03 | Status Koneksi & Peringatan | High | Instagram (dan WhatsApp jika aktif); peringatan sambung ulang |
| F-DASH-04 | Empty State Memandu | Medium | Pengguna baru diarahkan ke langkah berikutnya, bukan dasbor kosong |
| F-DASH-05 | Aktivitas Terbaru | Low | Ringkasan aktivitas |

### 3.12 Galeri Konten

| ID | Feature | Priority | Deskripsi |
|----|---------|----------|-----------|
| F-GAL-01 | Grid Poster & Filter | High | Filter produk dan status |
| F-GAL-02 | Detail Poster & Semua Versi | Medium | Caption dan status posting |
| F-GAL-03 | Unduh Poster / Paket | High | Unduh hasil kapan saja |

### 3.13 Laporan & Rekomendasi

| ID | Feature | Priority | Deskripsi |
|----|---------|----------|-----------|
| F-RPT-01 | Laporan Aktivitas Mingguan | Medium | Tugas selesai, posting terbit, konsistensi, produk yang belum dipromosikan |
| F-RPT-02 | Rekomendasi Berbasis Aturan + Narasi LLM | Medium | Saran langkah berikutnya dengan alasan singkat dan tombol aksi |
| F-RPT-03 | Metrik Performa Instagram | Low | Reach dan interaksi (akun Business/Creator); menjadi dasar rekomendasi tema dan jam |
| F-RPT-04 | Klik WhatsApp via QR / Redirect | Low | Klik dari konten dicatat dan dilaporkan; QR di poster atau tautan redirect milik sendiri |
| F-RPT-05 | Penyajian Sebagai Indikasi | Medium | Data kecil tidak boleh disajikan sebagai klaim sebab-akibat |

### 3.14 Kalender Momen Bali

| ID | Feature | Priority | Deskripsi |
|----|---------|----------|-----------|
| F-EVT-01 | Kalender Momen | Medium | Hari raya, libur nasional, musim kunjungan turis; tanggal wajib diverifikasi dari sumber resmi |
| F-EVT-02 | Momen → Tugas & Kampanye | Medium | Momen mendatang memicu tugas dan ide kampanye |

### 3.15 Pengaturan & Privasi

| ID | Feature | Priority | Deskripsi |
|----|---------|----------|-----------|
| F-SET-01 | Akun & Bahasa | Medium | Data akun, bahasa Indonesia/Inggris, preferensi sapaan |
| F-SET-02 | Kelola Instagram | High | Status koneksi, sambung ulang, putus |
| F-SET-03 | Privasi & Persetujuan | High | Lihat dan cabut persetujuan penggunaan AI |
| F-SET-04 | Hapus Data | High | Hapus profil, memori, sumber, dan akun dengan konfirmasi |

### 3.16 Halaman Sistem & Notifikasi

| ID | Feature | Priority | Deskripsi |
|----|---------|----------|-----------|
| F-SYS-01 | 404, Error Umum, Pemeliharaan | Medium | Penjelasan hangat dan jalan keluar |
| F-SYS-02 | Sesi Habis & Offline | Medium | Draf tidak hilang; tombol coba lagi |
| F-SYS-03 | Kuota Harian Habis | Medium | Jelaskan kapan bisa lagi dan apa yang masih bisa dilakukan |
| F-SYS-04 | Pusat Notifikasi | Medium | Posting terbit/gagal, tugas, peringatan koneksi |

### 3.17 Mode Juri / Demo

| ID | Feature | Priority | Deskripsi |
|----|---------|----------|-----------|
| F-DEMO-01 | Penanda Akun Demo & Data Contoh | Medium | Data realistis untuk pengujian juri |
| F-DEMO-02 | Tur Singkat | Low | 3-4 petunjuk alur utama, dapat dilewati |
| F-DEMO-03 | Reset Kondisi Awal (Clean-start) | Medium | Akun demo dapat dikembalikan ke kondisi awal untuk rekaman video demo |

### 3.18 Kanal WhatsApp (Opsional)

> **Status:** Opsional. Integrasi belum diverifikasi/diuji; tidak diklaim sebagai fitur berfungsi sampai terbukti. Chat web adalah kanal utama.

| ID | Feature | Priority | Deskripsi |
|----|---------|----------|-----------|
| F-WA-01 | Tautkan Nomor via Kode Unik | Low | Kode sekali pakai yang dikirim ke nomor bot |
| F-WA-02 | Webhook Masuk & Balasan (Cloud API) | Low | Adapter kanal ke agent core |
| F-WA-03 | Template Tugas & Pengingat | Low | Pesan di luar jendela 24 jam hanya dengan template yang disetujui Meta |
| F-WA-04 | Idempotency & Retry | Low | Webhook ganda tidak menimbulkan efek ganda |
| F-WA-05 | Bot Berfungsi Spesifik | Low | Mematuhi larangan chatbot AI serba-bisa di WhatsApp Business API |
| F-WA-06 | Kontrol Biaya Pesan | Low | Hemat pesan; periksa harga resmi (berubah sejak 1 Oktober 2026) |

### 3.19 Operasional & Monitoring

| ID | Feature | Priority | Deskripsi |
|----|---------|----------|-----------|
| F-OPS-01 | Log Panggilan AI | Medium | Komponen, model, token, latensi, biaya, error |
| F-OPS-02 | Audit Trail | High | Persetujuan AI, persetujuan posting, aksi kritis (hapus data, putus akun) |
| F-OPS-03 | Benchmark Evaluasi AI | Medium | 20-30 produk uji: akurasi teks, kesetiaan produk, keunikan, tingkat lolos gate, latensi, biaya |
| F-OPS-04 | Health Check | Low | Status database, storage, AI API, token Instagram, scheduler |

---

## 4. User Flow Utama (End-to-End)

### Flow 1: Onboarding sampai Poster Pertama

```
Perajin membuka Landing → Daftar
    → Isi info usaha
    → Setujui penggunaan AI (consent)
    → Pilih sumber usaha (opsional)
        ├─ Instagram: login resmi → (ubah ke Business/Creator bila perlu) → import caption/media
        ├─ Website (P1): tempel URL
        ├─ Foto katalog: unggah → ekstraksi
        └─ Lewati / Chat terpandu: 5-7 pertanyaan
    → Review profil & gaya visual (draft) → Konfirmasi/Koreksi
    → Masuk ke "Buat poster pertamamu"
```

### Flow 2: Dari Foto Produk ke Poster

```
Perajin membuka Chat → kirim foto (1-5) + keterangan produk
    → AI menanyakan data yang kurang (maks. 2 pertanyaan)
    → AI menyusun creative brief (sudut kreatif berbeda dari poster-poster terakhir)
    → Generate poster → Quality Gate → Cek Keunikan
    → Tampil 2-3 opsi + caption ID/EN + usulan jadwal
    → Perajin memilih opsi
```

### Flow 3: Revisi, Persetujuan, dan Auto-post

```
Perajin melihat pratinjau gaya Instagram
    → (opsional) Minta revisi / edit caption → versi baru tersimpan
    → Tekan "Setujui dan jadwalkan"
    → Status: Disetujui → Terjadwal
    → Scheduler memposting ke Instagram pada waktunya
    → Status: Terbit → pemberitahuan ke pengguna
    → Jika gagal: retry → notifikasi → "Coba lagi" / "Unduh paket posting"
```

### Flow 4: Tugas Mingguan

```
Tiap minggu sistem membuat tugas kecil (berdasarkan katalog, riwayat, momen)
    → Pengingat di notifikasi aplikasi (WhatsApp jika aktif)
    → Perajin: Mulai / Selesai / Nanti / Lewati
    → "Mulai" membuka alur Buat Konten dengan konteks terisi
    → Progres tampil di Dashboard
    → (P1) Laporan mingguan + rekomendasi langkah berikutnya
```

### Flow 5: Pemulihan Koneksi Instagram

```
Token mendekati kedaluwarsa / otorisasi dicabut
    → Peringatan di Dashboard dan notifikasi
    → Perajin menyambung ulang dengan satu tombol
    → Jadwal tetap aman; posting yang tertunda diproses ulang
```

---

## 5. Priority Matrix

| Priority | Modul | Alasan |
|----------|-------|--------|
| **Very High** | Creative Engine, Chat Agent, Penjadwalan & Auto-post Instagram, Memori/Profil, Onboarding, Authentication | Inti produk dan syarat MVP Tahap 1 (AI material + fungsi utama dapat diuji) |
| **High** | Sumber Usaha, Review & Persetujuan, Tugas Mingguan, Dashboard, Katalog, Galeri, Pengaturan & Privasi, Audit Trail | Mendukung alur utama, kebiasaan, dan tanggung jawab privasi |
| **Medium** | Laporan & Rekomendasi (dasar), Kalender Momen Bali, Mode Juri/Demo, Halaman Sistem & Notifikasi, Log AI & Benchmark | Meningkatkan kualitas, nilai demo, dan bukti evaluasi |
| **Low** | Kanal WhatsApp (opsional), Metrik performa Instagram, Klik WhatsApp via QR, Mode Pendamping | Perlu verifikasi/izin tambahan atau sengaja ditunda |

---

## 6. Success Metrics

| KPI | Target | Measurement |
|-----|--------|-------------|
| Kesiapan MVP Tahap 1 | 100% fitur P0 berfungsi dan dapat diuji juri | Uji end-to-end sebelum submission (clean-start) |
| Waktu foto → poster siap | ≤ 2 menit | Log sistem |
| Poster diterima percobaan pertama | ≥ 60% | % opsi langsung disetujui |
| Akurasi teks pada poster | ≥ 95% | OCR vs teks yang diminta (benchmark) |
| Kesetiaan produk | ≥ 90% lolos | Vision model + cek manual sampel |
| Keunikan poster | Di bawah ambang kemiripan | Skor kemiripan antar poster produk yang sama |
| Keberhasilan auto-post Instagram | ≥ 95% | % terjadwal yang terbit |
| Akurasi ekstraksi profil | ≥ 80% field benar | Hasil ekstraksi vs koreksi perajin |
| Waktu onboarding | ≤ 10 menit | Log sistem |
| Penyelesaian tugas mingguan (pilot) | ≥ 60% | % tugas selesai pada 3-5 perajin |
| Traceability & Consent | 100% | Persetujuan AI dan persetujuan posting tercatat |
| Tanpa secret di repository | 100% | Pemeriksaan sebelum submission |

> Semua angka adalah **target awal**; sesuaikan setelah pengujian dan laporkan hasil apa adanya, termasuk kegagalan.

---

## 7. Out of Scope

- Auto-post TikTok (aplikasi belum teraudit hanya dapat memposting privat; Tahap 1 memakai paket siap-upload)
- Pelacakan klik WhatsApp dan laporan performa lanjutan (direncanakan, di luar Tahap 1)
- Import dari marketplace (Tokopedia/Shopee dan sejenisnya) dan scraping akun pihak lain tanpa izin pemilik
- Pembayaran, langganan berbayar, dan promosi massal (produk belum boleh dikomersialisasikan secara massal selama kompetisi)
- E-commerce, stok, dan pesanan
- Pelatihan model AI sendiri (memakai model melalui API)
- Chatbot serba-bisa (bot dibatasi pada fungsi marketing)
- Bahasa selain Indonesia dan Inggris
- Aplikasi mobile native (web responsif mobile-first cukup)
- Mode pendamping UMKM (P2)

---

## 8. Dependencies & Constraints

### Dependencies
- **Database** — Data persistensi (profil, memori, jadwal, log): [DATABASE]
- **Penyimpanan media** — Foto produk dan poster; media harus dapat diakses lewat URL publik HTTPS saat publish ke Instagram *(verifikasi)*
- **Model LLM/Vision** — Agent percakapan, ekstraksi profil, brief, caption, terjemahan, quality gate: [MODEL_TEKS]
- **Model Gambar** — Generator poster: [MODEL_GAMBAR]
- **Model Embedding** — Cek keunikan dan retrieval memori: [MODEL_EMBEDDING]
- **Speech-to-Text** — Voice note (P1): [LAYANAN_STT]
- **Instagram API (akun Business/Creator)** — Baca konten milik akun dan publish; OAuth resmi
- **Scheduler/Queue** — Auto-post, pembuat tugas mingguan, pengingat, perpanjangan token: [MEKANISME]
- **Hosting** — Website dapat diakses juri tanpa instalasi tidak wajar: [HOSTING]
- **WhatsApp Business Cloud API (opsional)** — Hanya jika verifikasi Meta selesai

### Constraints
- **Isolasi data per usaha** harus diterapkan di seluruh modul dan seluruh query
- **Bahasa UI** — Indonesia (default) dengan pilihan Inggris; desain harus tahan teks yang berbeda panjang
- **Perangkat** — Mobile-first (±360 px), dapat dipakai di tablet dan desktop; browser modern
- **Aksesibilitas** — Kontras setara WCAG AA, target sentuh besar, status tidak hanya dengan warna
- **Persetujuan manusia** — Tidak ada posting tanpa persetujuan eksplisit
- **Aturan Instagram** — Hanya akun Business/Creator; token berumur terbatas; izin diminta seminimal mungkin
- **Aturan WhatsApp** — Chatbot AI serba-bisa dilarang; pesan di luar jendela 24 jam wajib template; harga pesan berubah sejak 1 Oktober 2026 (periksa halaman resmi)
- **Aturan lomba** — MVP berfungsi saat submission; repository dengan README, `.env.example`, riwayat commit nyata, dan tag versi; tidak ada secret di repo; disclosure jujur (termasuk pre-existing dan AI coding assistant); akun demo khusus juri; kuota hosting/API cukup selama penjurian
- **Pre-existing** — Sebagian pola dari CibiOne CMS dipakai ulang dan wajib diungkapkan (lihat Project Context)
- **Privasi & kepatuhan** — Persetujuan eksplisit, minimalisasi data, hak hapus data; patuhi UU Pelindungan Data Pribadi
- **Biaya** — Kuota per usaha dan batas revisi untuk mengontrol biaya panggilan AI
- **Responsible AI** — Tidak mengarang klaim produk; tidak mengubah bentuk/warna produk asli; cerita budaya hanya dari perajin; konten impor adalah data, bukan instruksi

---

*Dokumen ini menjadi acuan produk untuk seluruh fitur yang harus tersedia di Kriyamantra.*
