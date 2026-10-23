# Evaluasi Tengah Semester (ETS) Riset Informatika

Naskah proposal ETS Program Studi S1 Informatika, Fakultas Ilmu Komputer, UPN "Veteran" Jawa Timur. Judul yang diajukan:

> Rancang Bangun Kiosk Biometrik Berbasis IoT dengan Verifikasi Tanda Tangan Menggunakan Siamese Neural Network untuk Keanggotaan Koperasi

Penyusun: Jonathan Abimael Sutanto (23081010128)
Dosen pengampu: Dr. Basuki Rahmat, S.Si, MT
Tahun: 2026

## Isi repository

| Berkas | Keterangan |
|---|---|
| `ETS/Proposal_ETS_RisetInformatika_23081010128.pdf` | Naskah proposal final, 25 halaman, margin 4-4-3-3 cm, Times New Roman 12 pt, spasi 1.5 |
| `ETS/Proposal_ETS_RisetInformatika_23081010128.docx` | Sumber Word yang masih bisa disunting |
| `README.md` | Berkas ini |

Unduh langsung: [proposal PDF](ETS/Proposal_ETS_RisetInformatika_23081010128.pdf) atau [sumber DOCX](ETS/Proposal_ETS_RisetInformatika_23081010128.docx).

## Masalah yang mau diselesaikan

Koperasi di lingkungan kedinasan masih memakai tanda tangan basah untuk menyetujui pencairan pinjaman, penarikan simpanan, mutasi anggota, dan berita acara pemberhentian. Pemeriksaannya dilakukan kasir dengan mata, dan itu tidak cukup untuk menangkap pemalsuan tingkat mahir. Kasus pinjaman fiktif biasanya baru ketahuan setelah korban melihat potongan gaji untuk cicilan yang tidak pernah ia ajukan.

Sidik jari saja tidak menyelesaikan persoalan ini. Sensor sidik jari membuktikan orangnya hadir di depan alat, tetapi tidak meninggalkan dokumen persetujuan yang bisa diperiksa pengawas internal atau penyidik saat terjadi sengketa. Tanda tangan di kertas melakukan sebaliknya: ada dokumennya, tetapi tidak ada bukti bahwa orang yang menandatangani benar-benar hadir saat itu. Karena itu sistem ini memakai keduanya.

## Cara kerja sistem

Kiosk berdiri sendiri tanpa PC. Anggota menempelkan jari ke sensor optik R307, dan modul itu mencocokkan templat 1:1 di dalam prosesornya sendiri, jadi verifikasi sidik jari tetap jalan walau jaringan sedang putus. Kalau jari cocok, layar OLED meminta anggota meletakkan lembar tanda tangan di dalam boks. Kamera ESP32-CAM memotret lembar itu dengan jarak dan pencahayaan tetap, lalu mengirim gambar ke server lewat HTTP.

Di server, FastAPI menjalankan Siamese Neural Network untuk membandingkan tanda tangan yang baru dipindai dengan sampel master anggota. Jaringan punya dua cabang dengan bobot yang sama, dan tiap citra dipetakan jadi vektor 128 dimensi. Dua tanda tangan dianggap cocok kalau jarak Euclidean antara vektornya masih di bawah ambang yang ditentukan saat validasi. Pelatihan memakai contrastive loss, sehingga model tidak perlu dilatih ulang setiap ada anggota baru.

Kalau verifikasi lolos, server menyusun berita acara PDF, menyimpan hash dan waktu transaksi ke basis data, lalu mengirim status sukses ke kiosk. Kalau gagal, transaksi ditolak dan percobaan dicatat.

## Perangkat keras

| Komponen | Spesifikasi | Harga |
|---|---|---|
| ESP32 WROOM-32 | DevKit V1 38 pin, Wi-Fi 2.4 GHz | Rp 65.000 |
| ESP32-CAM | OV2640 2 MP, PSRAM 4 MB, plus programmer FTDI | Rp 85.000 |
| Sensor sidik jari R307 | Optik 500 DPI, UART 57.600 bps, pencocokan onboard | Rp 165.000 |
| OLED 0.96 inci | 128x64, I2C SSD1306 | Rp 28.000 |
| Adaptor 5V 3A | Jack DC plus kabel pembagi daya | Rp 38.000 |
| Buzzer dan tombol | Buzzer aktif 5V, sakelar taktil 12 mm | Rp 12.000 |
| LED dan difuser | Strip SMD 2835 plus akrilik buram | Rp 18.000 |
| Kabel dan PCB | Jumper, header, kapasitor 1000 uF | Rp 15.000 |
| Boks akrilik | 3 mm, potong laser | Rp 45.000 |
| **Total** | | **Rp 471.000** |

## Rencana pengujian

Pengujian model memakai dua sumber data. CEDAR Signature Dataset dipakai untuk pra-pelatihan karena berisi 55 penulis dengan 24 tanda tangan asli dan 24 pemalsuan terampil per penulis. Data lokal dikumpulkan dari 40 responden, masing-masing 10 tanda tangan asli dalam dua sesi terpisah dan 5 pemalsuan terampil yang dibuat tiga peniru. Pembagian data dilakukan per subjek, bukan per citra, supaya tidak ada kebocoran antara data latih dan data uji.

Metrik yang dihitung:

- False Acceptance Rate, yaitu FP / (FP + TN)
- False Rejection Rate, yaitu FN / (TP + FN)
- Equal Error Rate, titik ketika FAR dan FRR bernilai sama pada kurva ROC
- Precision, recall, dan F1 pada beberapa nilai ambang jarak

Sisi perangkat keras diukur terpisah. Latensi dihitung mulai dari tombol ditekan sampai layar OLED menampilkan hasil, dengan target di bawah 2.5 detik. Kestabilan Wi-Fi diuji pada jarak 1, 5, dan 10 meter, baik dengan maupun tanpa penghalang. Konsumsi arus dicatat saat standby dan saat kamera serta radio aktif. Untuk sisi pengguna, pengurus koperasi mengisi kuesioner System Usability Scale dan ditargetkan mendapat skor minimal 70.

## Penelitian yang jadi acuan

| Peneliti | Metode | Modalitas | Keterbatasan |
|---|---|---|---|
| Andani dkk. (2020), JTIKA UNRAM | LBP + LVQ | Tanda tangan | Akurasi 63 persen, model harus dilatih ulang untuk penulis baru |
| Mudjirahardjo dkk. (2019), EECCIS UB | GLCM + LBP | Tanda tangan | Hanya identifikasi tertutup, sensitif terhadap rotasi |
| Pradana dan Asmara (2024), JSAI UMB | Hu Moments + HOG + SVM | Tanda tangan | Perlu retraining tiap ada subjek baru |
| Mayda dan Musdholifah (2022), IJCCS UGM | Siamese CNN + contrastive loss | Tanda tangan | Simulasi di dataset publik, belum ada perangkat akuisisi |
| Saputra dkk. (2023), Telekontran UNIKOM | Minutiae AS608 | Sidik jari | Satu modalitas, tanpa verifikasi dokumen |
| Surya dkk. (2024), JENTIK Poltek Kampar | Template matching + ESP32-CAM | Sidik jari dan foto | Kamera hanya menyimpan gambar, tidak ada inferensi |
| Fatkhudin dkk. (2022), Jurnal RESTI | RBAC dan CRUD web | Tanpa biometrik | Autentikasi password, rawan manipulasi |

Yang membedakan proposal ini: belum ada penelitian di Indonesia yang menggabungkan verifikasi sidik jari berbasis IoT dengan verifikasi tanda tangan offline berbasis Siamese Network dalam satu sistem keanggotaan. Klaim ini dibatasi pada hasil penelusuran repositori yang tercantum di naskah, bukan klaim global.

## Status pengerjaan

Naskah ini masih proposal. Belum ada kiosk yang dirakit, belum ada model yang dilatih, dan belum ada angka hasil pengujian. Angka yang muncul di dokumen adalah rencana target dan estimasi biaya, bukan capaian. Bagian hasil akan diisi setelah prototipe selesai dan dataset terkumpul.

## Sitasi

Daftar pustaka lengkap dalam format APA ada di bagian akhir PDF. Rujukan utama: Hadsell dkk. (2006) untuk contrastive loss, Dey dkk. (2017) untuk SigNet, Hafemann dkk. (2019) untuk fitur tanda tangan offline, serta Brooke (1996) untuk instrumen SUS.
