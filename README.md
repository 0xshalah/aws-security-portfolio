# Portofolio Keamanan AWS - Shalahuddin Al-Ayyubi

Selamat datang di portofolio proyek keamanan cloud saya. Repositori ini berisi dokumentasi dan artefak dari proyek-proyek yang saya kerjakan untuk mendemonstrasikan keahlian saya di bidang AWS, Pentesting, dan Otomatisasi Keamanan.

---

## Daftar Proyek

### Proyek 1: Analisis Kerentanan & Hardening EC2
* **Tujuan:** Mensimulasikan server yang sengaja dibuat rentan di AWS EC2, melakukan pemindaian keamanan, dan kemudian melakukan *hardening* (perbaikan) menggunakan AWS Security Groups dan Network ACLs.
* **Teknologi:** `AWS EC2`, `Security Groups`, `Apache2`, `PHP`, `MySQL`, `DVWA`, `Nmap`.
* **Status:** `Fase Setup Selesai - Siap untuk Pentesting`

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

Proyek ini dibagi menjadi beberapa fase. Fase pertama adalah *setup* infrastruktur dan aplikasi yang sengaja dibuat rentan.

### Fase 1: Setup Infrastruktur (AWS EC2)

Saya memulai dengan meluncurkan sebuah *instance* EC2 baru dengan konfigurasi berikut:
* **Name:** `Project1-Vulnerable-Server`
* **OS (AMI):** `Ubuntu Server 24.04 LTS` (Free tier eligible)
* **Instance Type:** `t2.micro` (Free tier eligible)
* **Key Pair:** `aws-project1-key` (dibuat baru dan disimpan dengan aman)

![Konfigurasi Nama Server](./Screenshot%202025-10-26%20012900.png)
![Konfigurasi OS Ubuntu](./Screenshot%202025-10-26%20012933.png)
![Konfigurasi Instance Type t2.micro](./Screenshot%202025-10-26%20013020.png)
![Konfigurasi Key Pair](./Screenshot%202025-10-26%20013120.png)

* #### Konfigurasi Firewall (Security Group)
Bagian terpenting adalah konfigurasi *Security Group*. Untuk tujuan proyek ini, saya **sengaja membuat aturan yang sangat tidak aman** agar bisa dianalisis nanti.

* **Security Group Name:** `Project1-Vulnerable-SG`
* **Inbound Rules:**
    1.  `Type: SSH` (Port 22), `Source: Anywhere (0.0.0.0/0)`
    2.  `Type: HTTP` (Port 80), `Source: Anywhere (0.0.0.0/0)`
    3.  `Type: All traffic` (All ports), `Source: Anywhere (0.0.0.0/0)`

![Konfigurasi Security Group Name](./Screenshot%202025-10-26%20013337.png)
![Konfigurasi Security Group Rules](./Screenshot%202025-10-26%20013349.png)

### Fase 2: Setup Server & Aplikasi Rentan (DVWA)

Setelah *instance* berjalan, saya terhubung ke server menggunakan SSH dan file `.pem` saya.

1.  **Update Server:** Pertama, saya memperbarui semua *package manager*.
    ```bash
    apt update && apt upgrade -y
    ```
    ![Terminal apt update upgrade](./Screenshot%202025-10-26%20013904.png)

2.  **Instal LAMP Stack:** Saya menginstal Apache (Web Server), MySQL (Database), dan PHP.
    ```bash
    apt install apache2 -y
    apt install php libapache2-mod-php php-mysql mysql-server -y
    ```
    ![Terminal install apache2](./Screenshot%202025-10-26%20014011.png)
    ![Terminal install PHP & MySQL](./Screenshot%202025-10-26%20014254.png)

3.  **Verifikasi Web Server:** Saya mengakses IP Publik server (`http://98.93.196.5`) dari browser dan memverifikasi bahwa halaman default Apache telah aktif.
    ![Verifikasi Halaman Apache](./Screenshot%202025-10-26%20014118.jpg)

4.  **Instal DVWA:** Saya mengunduh (clone) aplikasi *Damn Vulnerable Web Application* (DVWA) dari GitHub.
    ```bash
    git clone [https://github.com/digininja/DVWA.git](https://github.com/digininja/DVWA.git) /var/www/html/dvwa
    ```
    ![Terminal git clone DVWA](./Screenshot%202025-10-26%20014346.png)

5.  **Konfigurasi Izin (Sengaja Dibuat Rentan):** Saya memberikan izin `777` (baca, tulis, eksekusi untuk semua) ke folder DVWA.
    ```bash
    chmod -R 777 /var/www/html/dvwa
    ```
    ![Terminal chmod 777](./Screenshot%202025-10-26%20014518.png)

6.  **Setup Database & DVWA:** Saya mengkonfigurasi file `config.inc.php` DVWA dan membuat database MySQL yang diperlukan. Setelah me-restart Apache, saya berhasil mengakses halaman login DVWA dan menyelesaikan setup.

### Fase 3: Analisis Kerentanan Awal

Server ini sekarang memiliki beberapa kerentanan kritis yang disengaja:

1.  **Kerentanan Infrastruktur (Security Group):**
    * **SSH (Port 22) terbuka untuk `0.0.0.0/0`:** Ini sangat berisiko. Ini mengizinkan *siapa saja* di internet untuk mencoba serangan *brute force* (menebak password/key) terhadap server saya.
    * **All traffic terbuka untuk `0.0.0.0/0`:** Ini lebih parah. Ini membuka *semua port*, termasuk port sensitif seperti database (`MySQL - 3306`) ke seluruh internet. Penyerang bisa langsung terhubung ke database saya dari mana saja.

2.  **Kerentanan Server (Izin File):**
    * **`chmod 777`:** Ini adalah mimpi buruk keamanan. Ini berarti *user* web server (`www-data`) memiliki izin untuk *menulis* dan *mengeksekusi* file di dalam folder DVWA.
    * **Implikasi:** Jika seorang penyerang berhasil menemukan celah *file upload* di DVWA (yang memang ada), mereka dapat mengunggah *web shell* (misalnya `shell.php`). Karena izin `777`, server akan mengizinkan file itu dieksekusi, memberikan penyerang kendali penuh (Remote Code Execution) atas server saya.

3.  **Kerentanan Aplikasi (DVWA):**
    * Aplikasi DVWA sendiri sengaja dibuat penuh dengan celah, seperti **SQL Injection**, **Cross-Site Scripting (XSS)**, dan **File Inclusion**, yang akan saya gunakan sebagai target pengujian.

### Fase 4: Rencana Pengujian (Pentesting)

Dengan setup ini, server saya siap untuk diuji.

#### 1. Reconnaissance (Nmap)
Saya memulai fase pengujian dengan pemindaian Nmap dari *attack machine* (Kali Linux) untuk mengidentifikasi *port* dan layanan yang berjalan.

![Hasil Nmap Scan](./Screenshot%202025-10-26%20182709.png)

**Analisis Hasil Nmap:**

  * **`PORT 22/tcp open ssh OpenSSH 9.6p1...`**: Port SSH terbuka untuk umum. Ini adalah risiko untuk serangan *brute-force*.
  * **`PORT 53/tcp open domain Unbound`**: Port DNS terekspos. Ini adalah *information disclosure* (kebocoran informasi) yang mengungkapkan bahwa server menggunakan *resolver* "Unbound".
  * **`PORT 80/tcp open http Apache httpd 2.4.58...`**: Port web server utama kita, menjalankan Apache. Ini akan menjadi target utama untuk eksploitasi aplikasi web.
  * **Temuan Penting (Port 3306 Hilang):** Menariknya, Port 3306 (MySQL) tidak muncul sebagai `open` meskipun *Security Group* mengizinkan `All traffic`. Ini adalah contoh bagus dari **Defense in Depth**. Walaupun *firewall* jaringan gagal (sengaja dibuka), *konfigurasi aplikasi* MySQL aman (secara *default* hanya *listening* di `127.0.0.1`), sehingga *port* tersebut tidak merespons koneksi dari internet.

#### 2\. Vulnerability Assessment (Nikto)

Selanjutnya, saya melakukan pemindaian kerentanan pada aplikasi web dengan Nikto untuk mencari kesalahan konfigurasi umum dan file berbahaya.

![Hasil Nikto Scan](./Screenshot%202025-10-26%20191000.png)

**Analisis Hasil Nikto:**

  * **Miskonfigurasi Header:** Nikto menemukan bahwa *header* `X-Frame-Options` dan `X-Content-Type-Options` tidak ada. Ini membuat aplikasi rentan terhadap serangan *Clickjacking* dan *MIME sniffing*.
  * **Directory Indexing:** Nikto menemukan beberapa direktori sensitif yang terekspos: `/config/`, `/database/`, `/docs/`. Ini adalah kebocoran informasi yang parah, memungkinkan penyerang melihat file konfigurasi dan struktur database.
  * **Eksposur .git:** Temuan paling kritis adalah `/git/HEAD` dan `/git/config`. Ini berarti seluruh *source code* aplikasi bisa diunduh oleh penyerang, memberi mereka "cetak biru" lengkap untuk menemukan lebih banyak celah.
  * **File Login:** Menemukan `/login.php`, mengonfirmasi titik masuk aplikasi.

#### 3\. Exploitation (Eksploitasi)

Bagian ini mendemonstrasikan eksploitasi kerentanan yang ditemukan.

**a. SQL Injection (SQLi)**
Saya menargetkan modul "SQL Injection" di DVWA. Pertama, saya login ke DVWA dan mengatur *Security Level* ke **Low**.

  * **Payload:** `' OR 1=1 #`
  * **Hasil:** Payload ini berhasil mem-bypass logika `WHERE` pada *query* SQL, menyebabkan *query* tersebut mengembalikan **semua data** dari tabel `users`, seperti yang terlihat di bawah ini.

![Login ke DVWA](./dvwa-login.png)
![Set Security Level Low](./dvwa-security-low.png)
...
![Hasil SQL Injection Sukses](./sqli-success.png)
