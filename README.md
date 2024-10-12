# Komunikasi Data dan Jaringan Komputer - Deploy Aplikasi Web "*SickChill*" 🎥 

## Sekilas Tentang SickChill

SickChill adalah aplikasi web open-source yang digunakan untuk melacak, mengelola, dan mengunduh serial TV secara otomatis. Aplikasi ini dirancang untuk membantu pengguna mengikuti jadwal serial favorit mereka dengan mudah. SickChill mendukung berbagai platform torrent dan hoster untuk mengunduh konten yang diinginkan. Fitur utamanya meliputi pemindaian otomatis dari episode baru, kemampuan untuk memilih resolusi video, serta integrasi dengan sistem notifikasi yang memberikan update secara real-time terkait episode yang tersedia.

## Instalasi

#### Kebutuhan Sistem :
- Linux CLI
- Php 5.3+
- Node.js v18 atau versi yang lebih tinggi
- Web server (GCP)
- Docker dan Docker compose

#### Proses Instalasi :
1. Pastikan sudah memiliki docker dan docker compose dengan menjalankan kode berikut. 
    ```
    $ docker -v
    $ docker-compose -v
    ```
    Jika docker belum terinstal, jalankan kode berikut.
    ```
    $ sudo snap install docker  
    $ sudo apt  install docker-compose
    ```
    
2. Instalasi SickChill. Kita bisa menggunakan docker image pull untuk menginstallnya.
   
    Pull dari docker image:
    ```
    $ docker pull linuxserver/sickchill
    ```

3. Konfigurasi GCP :
    Jalankan perintah berikut untuk login ke akun Google Cloud:
    ```
    $ gcloud auth login
    ```
    Selanjutnya akan diarahkan untuk login menggunakan akun Google Cloud. Setelah login, inisialisasi konfigurasi dengan perintah berikut untuk terhubung ke proyek:
    ```
    $ gcloud init
    ```
    Untuk langkah konfigurasi, pilih opsi sesuai dengan yang diinginkan.
   - Pilih konfigurasi yang ada atau buat konfigurasi baru
     ![image](https://github.com/user-attachments/assets/f22e14e9-2f66-4e09-ba0f-25563ff16e57)
   - Pilih akun Google yang akan digunakan dan pilih proyek dari daftar proyek yang tersedia
     ![image](https://github.com/user-attachments/assets/7472f350-5e86-409d-b6dd-6212e29a9efc)
4. Setelah menghubungkan Linux CLI dengan GCP, langkah berikutnya adalah membuat repository baru untuk Artifact Registry di GCP.

   Klik tombol + Create Repository yang terdapat di halaman Artifact Registry pada dashboard Google Cloud.
   
   ![image](https://github.com/user-attachments/assets/f22e14e9-2f66-4e09-ba0f-25563ff16e57)
   
   Buat nama repository sesuai keinginan. Pilih Docker sebagai format, atau gunakan format nama sebagai NAME.gcr.io agar format otomatis menjadi Docker. Pilih lokasi penyimpanan repository dengan opsi Region atau Multi-region untuk memastikan keamanan data dan meminimalkan risiko kehilangan data.
   
   ![image](https://github.com/user-attachments/assets/7472f350-5e86-409d-b6dd-6212e29a9efc)
   
   Sebelum melakukan push ke Artifact Registry, pastikan untuk membuat **Service Account** dengan peran IAM (IAM roles) berikut:
   - Artifact Registry Administrator
   - Artifact Registry Create-on-Push Repository Administrator
   - Storage Admin
     
     ![image](https://github.com/user-attachments/assets/cc608fc7-6312-44f5-a8c2-cc7126d65785)
     
     ![image](https://github.com/user-attachments/assets/a14bc9f0-4cf6-44d7-8fe5-becd16065791)

   Langkah selanjutnya adalah mengekspor kredensial dengan cara berikut:
   - Buka Service Accounts di menu IAM & Admin.
   - Klik pada Service Account yang telah dibuat.
   - Pada bagian Keys, klik Add Key, lalu pilih Create New Key.
   - Pilih Key Type sebagai JSON dan simpan file yang diunduh untuk digunakan nanti.
     
     ![image](https://github.com/user-attachments/assets/d8ddc6f6-2bf2-4fa7-a92c-e373e41ebd54)
5. Pindah ke CLI Linux dan atur kredensial aplikasi Google Cloud dengan perintah berikut:
    ```
    $ export GOOGLE_APPLICATION_CREDENTIALS="/path/to/file.JSON"
    ```
    Setelah itu, buat image Docker dari file yang akan di-push. 

    Langkah pertama, menandai (tag) image tersebut dengan perintah berikut:
    ```
    $ docker tag folder-to-be-pushed ARrepo/IDproj/imagename
    ```
    Disini, saya menggunakan perintah dibawah ini.
    ```
    $ docker tag linuxserver/sickchill asia-southeast2-docker.pkg.dev/komdat-438111/komdat/sickchill
    ```
    Penjelasan:
    - ```linuxserver/sickchill``` adalah nama folder berisi kode yang akan di-push.
    - ```asia-southeast2-docker.pkg.dev``` adalah nama repository di Artifact Registry.
    - ```komdat-438111/komdat/``` adalah ID dari project di Google Cloud.
    - ```sickchill``` adalah nama image yang sedang dibuat.
    
    Langkah selanjutnya, push image tersebut ke Artifact Registry menggunakan perintah:
    ```
    $ docker push ARrepo/IDproj/imagename
    ```
    Perintah yang saya gunakan sebagai berikut :
    ```
    $ docker push asia-southeast2-docker.pkg.dev/komdat-438111/komdat/sickchill
    ```

    Bila anda tidak bisa melakukan push ke artifact registry, maka jalankan perintah berikut sebelum perintah push:
    ```
    $ gcloud auth print-access-token | docker login -u oauth2accesstoken --password-stdin https://asia-southeast2-docker.pkg.dev
    ```     
    Ganti asia-southeast2-docker.pkg.dev sesuai dengan region yang digunakan.
    
    Setelah proses push selesai, image akan muncul di Artifact Registry sesuai dengan detail yang telah ditentukan.
    
    ![image](https://github.com/user-attachments/assets/efd2c65a-7bb7-4806-ac83-ed02d26f5c09)
   
    Langkah terakhir, web dapat dideploy menggunakan Cloud Run dengan cara klik **Deploy Container**, pilih **Service**, lalu pilih **Container Image URL** yang sesuai. Pada "Authentication",  pilih "Allow unauthenticated invocations".
    
    ![Screenshot 2024-10-11 231932](https://github.com/user-attachments/assets/d1d6c457-37ea-4863-87da-f39f0ecf443d)

    Scroll halaman ke paling bawah hingga anda dapat melihat drop-down section "Container(s), Volumes, Networking, and Security". Isi "Container Port" dengan 8081.

    ![Screenshot 2024-10-12 103954](https://github.com/user-attachments/assets/493cc493-c658-41a6-a091-bca89c40892d)

    Nama Service akan terisi secara otomatis. Pilih region sesuai keinginan, disarankan untuk memilih yang paling dekat, yaitu ```asia-southeast2```. Biarkan opsi lainnya pada pengaturan default, lalu klik Create.
    
    ![image](https://github.com/user-attachments/assets/6b2ced4c-4f79-4445-a06f-43ae0f90f897)

    Tunggu hingga proses deployment selesai. Setelah itu, klik nama Service yang sudah dibuat. URL aplikasi akan muncul di bagian atas.
   
    ![image](https://github.com/user-attachments/assets/01a473a9-12c1-432c-95b1-174ebdf774d0)

## Konfigurasi (opsional)

Setting server tambahan yang diperlukan untuk meningkatkan fungsi dan kinerja aplikasi, misalnya:
- batas upload file
- batas memori
- dll

Plugin untuk fungsi tambahan:
- login dengan Google/Facebook
- editor Markdown
- dll

## Maintenance (opsional)

Setting tambahan untuk maintenance secara periodik, misalnya:
- buat backup database tiap pekan
- hapus direktori sampah tiap hari
- dll

## Otomatisasi (opsional)

Skrip shell untuk otomatisasi instalasi, konfigurasi, dan maintenance.

## Cara Pemakaian

- Tampilan aplikasi web
- Fungsi-fungsi utama
- Isi dengan data real/dummy (jangan kosongan) dan sertakan beberapa screenshot

## Pembahasan

- Pendapat anda tentang aplikasi web ini:
    - kelebihan
    - kekurangan
- Bandingkan dengan aplikasi web lain yang sejenis

## Referensi

Cantumkan tiap sumber informasi yang anda pakai.
