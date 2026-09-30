# Laporan Praktikum Modul 1: Dockerized ROS 2 Development Environment

## Langkah Pengerjaan
1. **Verifikasi Docker:** Memastikan Docker dan Docker Compose telah terpasang serta dapat berjalan tanpa akses root (`sudo`) dengan menguji `docker run --rm hello-world`.
2. **Pembuatan Workspace dan Package:** Membuat struktur direktori proyek (`docker/`, `src/`, `docs/`) dan menginisialisasi repositori Git di mesin host. Package `my_first_robot_package` dibuat di dalam container menggunakan image `ros:humble-ros-base-jammy` melalui perintah `ros2 pkg create`, sehingga tidak perlu menginstal ROS 2 di OS host.
3. **Konfigurasi Dockerfile & Entrypoint:** Membuat file `Dockerfile` untuk mengunduh dependensi (`demo-nodes-cpp`, `demo-nodes-py`), mengatur user non-root (`dev`), menyalin source code, dan menjalankan `colcon build` secara otomatis. Dibuat juga file `entrypoint.sh` untuk melakukan *source* pada lingkungan ROS sebelum perintah dieksekusi.
4. **Konfigurasi Docker Compose:** Membuat `compose.yaml` yang mendefinisikan service `talker`, `listener`, dan profil `dev`. Seluruh service menggunakan *anchor* YAML untuk memastikan parameter `network_mode: host`, `ipc: host`, dan `ROS_DOMAIN_ID` sama agar komunikasi antar node lancar.
5. **Build dan Eksekusi:** Menjalankan `docker compose build` dan `docker compose up`. Memastikan log pesan `Publishing` dan `I heard` tampil bergantian dengan lancar, serta memverifikasi eksistensi node lewat `ros2 node list` di terminal terpisah.
6. **Dokumentasi Git & GitHub:** Membuat file `.gitignore` dan `README.md` yang memuat instruksi eksekusi repositori, lalu melakukan *commit* dan *push* kode ke repositori GitHub.

## Kendala yang Ditemui dan Solusi
Selama pengerjaan praktikum, terdapat beberapa kendala teknis yang berhasil diselesaikan:
1. **Error UID Not Found saat Membuat Package:** Saat menjalankan perintah `ros2 pkg create` via Docker, muncul pesan `KeyError: 'getpwuid(): uid not found: 1000'`. Hal ini terjadi karena *parser* ROS 2 gagal mengambil nama profil maintainer secara otomatis; OS container tidak mengenali UID `1000` di dalam `/etc/passwd`.
   * **Solusi:** Menambahkan parameter `-v /etc/passwd:/etc/passwd:ro` pada eksekusi perintah Docker agar container dapat membaca daftar *user* langsung dari OS Ubuntu host dengan izin baca (*read-only*).
2. **Node Listener Tidak Merespons:** Saat `docker compose up` dijalankan, node listener bungkam dan log terminal hanya menampilkan pesan dari talker secara berulang (tidak ada *I heard*). Diagnosa awal menunjukkan bahwa file `compose.yaml` terpotong akibat kegagalan *copy-paste* di terminal (terdapat struktur yang tidak valid dan hilangnya blok `volumes`).
   * **Solusi:** Menghentikan proses container, menghapus isi `compose.yaml` yang rusak, dan menulis ulang seluruh blok kode menggunakan editor `nano`. Setelah struktur YAML (terutama *anchor* jaringan dan environment) dibenahi dan container dinyalakan ulang, listener dan talker akhirnya berhasil berkomunikasi.
3. **Masalah Autentikasi Git Push:** Git menolak *commit* pertama karena profil `user.name` dan `user.email` belum dikonfigurasi di sistem operasi yang baru. Setelah identitas diatur, proses eksekusi `git push` ditolak oleh GitHub dengan pesan *authentication failed* akibat penonaktifan login via *password* akun.
   * **Solusi:** Mengonfigurasi `git config --global` untuk mengatur identitas secara lokal. Untuk proses *push*, login dilakukan menggunakan GitHub *Personal Access Token* (PAT) dengan *scope* `repo` sebagai pengganti kata sandi. Pengunggahan data akhirnya berhasil 100%.

## Kesimpulan
Di akhir praktikum, seluruh komponen berhasil diimplementasikan. Repositori dapat dibangun cukup dengan perintah `docker compose build` dan `docker compose up` tanpa harus memasang dependensi di host. Node talker dan listener ROS 2 berjalan otomatis secara bersamaan dan saling bertukar pesan. Lingkungan pengembangan sudah sepenuhnya berjalan di dalam wadah kontainer (*Dockerized*), memenuhi kriteria reproduktibilitas untuk pengerjaan modul-modul robotika selanjutnya.
