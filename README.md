# Komunikasi Data dan Jaringan Komputer - Deploy Aplikasi Web "*SickChill*" 🎥 

## Sekilas Tentang SickChill

SickChill adalah aplikasi web open-source yang digunakan untuk melacak, mengelola, dan mengunduh serial TV secara otomatis. Aplikasi ini dirancang untuk membantu pengguna mengikuti jadwal serial favorit mereka dengan mudah. SickChill mendukung berbagai platform torrent dan hoster untuk mengunduh konten yang diinginkan. Fitur utamanya meliputi pemindaian otomatis dari episode baru, kemampuan untuk memilih resolusi video, serta integrasi dengan sistem notifikasi yang memberikan update secara real-time terkait episode yang tersedia.

Berikut adalah website official dari SickChill :
```
https://sickchill.github.io/
```
```
https://github.com/SickChill/SickChill
```

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
   - Pilih akun Google yang akan digunakan dan pilih proyek dari daftar proyek yang tersedia
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

   Berikut adalah tautan pada website hasil deploy self-host SickChill :
   ```
   https://sickchill-1-1074984779123.us-central1.run.app/home/
   ```

## Maintenance

- Apabila ingin menggunakan sesi SickChill baru, disarankan untuk menghapus semua folder tambahan seperti cache agar kembali bersih.

## Cara Pemakaian

- Tampilan aplikasi web
  - **Homepage**
  
  ![image](https://github.com/user-attachments/assets/94087f72-475d-4f63-b7fe-5fc4a78f64df)
  
  Saat pertama kali mengakses SickChill, user akan diarahkan ke halaman utama yang menampilkan daftar acara TV yang sedang dikelola. Halaman ini memungkinkan user untuk memantau semua acara TV yang telah ditambahkan dan status episode mereka.
  
  - **Details of Show**
  
  ![image](https://github.com/user-attachments/assets/22a317de-3a1b-4c61-bef2-939e68590615)
  
  User dapat mengklik salah satu acara TV yang ditampilkan pada Homepage untuk melihat detail acara tersebut lebih lanjut.     - Halaman detail acara ini memungkinkan user untuk:
   - Melihat daftar semua episode dalam satu atau beberapa musim.
   - Memeriksa status unduhan dari setiap episode (misalnya: sudah diunduh, tersedia untuk diunduh, atau belum dirilis).
   - Melihat informasi terkait seperti kualitas unduhan dan subtitle yang tersedia.
     
  - **Schedule**

  ![image](https://github.com/user-attachments/assets/a61ba09a-d753-4a2c-83b7-02e83ca5705e)

  SickChill juga menyediakan kalender pada halaman Schedule yang menampilkan jadwal episode yang akan datang. User dapat melihat kapan episode berikutnya dari acara TV favorit mereka akan tayang, sehingga tidak perlu lagi ketinggalan jadwal.

  - **History**

  ![image](https://github.com/user-attachments/assets/7d531c9d-63b2-4c69-abe1-335e2bf45524)

  Untuk membantu user melacak unduhan sebelumnya, SickChill menyediakan halaman History yang mencatat daftar riwayat semua episode yang telah diunduh atau diproses. User dapat melihat episode mana saja yang telah berhasil diunduh, serta waktu dan tanggal unduhan tersebut.

  - **Add New Show**

  ![image](https://github.com/user-attachments/assets/943d9028-3b38-428a-a136-dfc4dfe332d3)

  User dapat menambahkan daftar acara TV baru dengan mudah. User hanya perlu mencari acara yang diinginkan, memilih kualitas unduhan, dan menambahkannya ke dalam sistem pemantauan otomatis.

  - **Search Providers**

  ![image](https://github.com/user-attachments/assets/e66f35be-0c13-45b2-aa98-349c17d81368)

  SickChill mendukung berbagai penyedia pencarian (search providers) untuk menemukan sumber unduhan terbaik sesuai preferensi user. User dapat memilih dari daftar panjang penyedia torrent atau NZB yang mendukung pencarian episode acara TV favorit mereka.

- Cara menonton episode di SickChill
  - Pada halaman utama, klik “check your settings” di bagian atas

  ![Screenshot 2024-10-12 171052](https://github.com/user-attachments/assets/9511dda3-f12a-41e6-ab0d-f9b10e2e1282)

  - Checklist kotak “enable NZB search providers”, tetapkan direktorinya di “cache”, kemudian simpan pengaturan dengan mengklik “Save Changes” di bagian kiri bawah.

  ![Screenshot 2024-10-12 171122](https://github.com/user-attachments/assets/9635d89c-c2cd-4888-8978-b16f6af4e978)

  - Checklist kotak “enable torrent search providers”, tetapkan direktorinya di “cache”, kemudian simpan pengaturan lagi dengan mengklik “Save Changes” di kiri bawah.

  ![Screenshot 2024-10-12 171132](https://github.com/user-attachments/assets/581ce0fe-ed28-40d3-970f-d98c8ba33802)

  - Klik lagi “check your settings”. Setelah diarahkan ke halaman Search Providers, checklist providers yang diinginkan (checklist semua providers jika tidak ada yang diketahui), lalu klik “Save Changes” lagi untuk menyimpan.

  ![Screenshot 2024-10-12 171156](https://github.com/user-attachments/assets/4a1abb63-c32f-4840-a9ae-7a9724b8034d)

  - Arahkan kursor ke tab “Show”, pilih “Add Shows”, lalu klik “Add New Show”.

  ![Screenshot 2024-10-12 171242](https://github.com/user-attachments/assets/1f446b81-9e17-4c7b-a85c-d004617cf610)

  - Ketik nama show yang ingin dicari pada kolom “Show name”, klik “Search” kemudian cari pada tabel “#2 Pick a Show”. Pilih resolusi dan format yang diinginkan lalu klik “Add Show”.

  ![Screenshot 2024-10-12 171319](https://github.com/user-attachments/assets/30c6ff12-b13a-42c1-bac5-377111cdd19e)

  - Kembali ke halaman utama. Show yang sudah ditambahkan tadi akan terlihat pada halaman utama, kemudian klik show tersebut.

  ![Screenshot 2024-10-12 171500](https://github.com/user-attachments/assets/de58749a-cba6-4d2c-93b0-363a011c4771)

  - Selanjutnya akan ditampilkan list-list episode yang sudah pernah tayang dan terkonfirmasi akan tayang.

  ![Screenshot 2024-10-12 171541](https://github.com/user-attachments/assets/4e2bd117-af9f-4ad4-85f3-2f9b109c89c8)

  - Untuk menonton episode yang sudah tayang, klik simbol magnifying glass yang berada di sebelah kiri tombol plus di kanan halaman lalu tunggu beberapa detik hingga menit.

  ![Screenshot 2024-10-12 171605](https://github.com/user-attachments/assets/d90117c1-aea3-4420-ad7f-7096b3f28b2a)

  - Apabila sudah selesai menonton, status akan berubah menjadi “snatched”. Klik logo plus di paling kanan baris.

  ![Screenshot 2024-10-12 171926](https://github.com/user-attachments/assets/220c4e2d-6b4f-4ce5-b56c-0439ff327ca5)

  - Pilih salah satu hasil pencarian kemudian pilih sesuai resolusi yang diinginkan

  ![Screenshot 2024-10-12 171944](https://github.com/user-attachments/assets/52b2dae1-7503-4028-bcc5-a83d95db11dc)

  - Apabila sudah memilih, klik kanan tombol “https” dan inspect element.

  ![Screenshot 2024-10-12 172132](https://github.com/user-attachments/assets/aab07c39-3434-4344-b031-f48b8067f22e)

  - Beberapa baris di atas akan menampilkan sebuah link torrent, copy link tersebut.

  ![Screenshot 2024-10-12 172159](https://github.com/user-attachments/assets/83489513-23b9-4f65-b83b-6bfa1460d4f3)

  - Paste link tersebut di tab baru.

  ![Screenshot 2024-10-12 172209](https://github.com/user-attachments/assets/5c873970-e50d-452a-81ed-85b707ae4fe8)

  - File torrent akan otomatis terdownload dan pencarian episode berhasil dilakukan. Selanjutnya bisa menggunakan provider torrent bebas untuk mendownload episodenya seperti uTorrent.

  ![Screenshot 2024-10-12 172218](https://github.com/user-attachments/assets/a4627a0a-0ee6-4d06-9f79-0d794da8c717)


## Pembahasan

- Pendapat anda tentang aplikasi web ini:
    - Kelebihan :
      - Dapat men-track shows yang sudah ditonton maupun ingin ditonton.
      - Database terhubung ke berbagai website shows database (IMDb, TheTVDB, AniDB, dan lainnya).
      - SickChill langsung menyediakan link torrent sehingga mempermudah pengguna untuk menonton episode tanpa harus surfing di website torrent secara manual.
    - Kekurangan :
      - User harus menambahkan show yang mereka ingin tonton terlebih dahulu.
      - Post-processing default-nya tidak menggunakan local, sehingga dapat menyulitkan orang awam dalam penggunaannya.
      - Apabila tidak digunakan untuk beberapa waktu yang lama, sesi berpotensi dapat berakhir dan menghapus shows yang telah ditambahkan.
- Bandingkan dengan aplikasi web lain yang sejenis
  - Sonarr :
    - Pada Sonarr, penggunaan fiturnya jauh lebih mudah untuk digunakan dan lebih intuitif.
    - Sonarr memiliki kemampuan untuk "babysit" query downloader secara aktif. Ini berarti apabila terdapat error pada post-processing, Sonarr akan otomatis mengalihkan proses pada query selanjutnya. SickChill harus dikonfigurasi secara manual.
  - Medusa :
    - Medusa cenderung beroperasi lebih cepat dan lancar.
    - Medusa dapat menangani API problem dan memperbaiki key-nya sendiri, SickChill sering terjadi mismatch API key.
