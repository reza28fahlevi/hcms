# Panduan Instalasi NexaHCMS

Panduan memasang aplikasi dari awal di Windows memakai **Laragon** (paling mudah, URL tanpa `/public`). Alternatif XAMPP ada di bagian akhir.

## Kebutuhan

| Komponen | Keterangan |
|---|---|
| PHP **8.3 atau lebih baru** | Wajib (Laravel 13). |
| Ekstensi PHP | `gd`, `zip`, `intl`, `mbstring`, `pdo_mysql`, `fileinfo` |
| MySQL / MariaDB | Database aplikasi |
| Composer | Untuk install dependency PHP |
| Koneksi internet di browser | Bootstrap, jQuery, dan Select2 dimuat dari CDN jsDelivr |
| Node.js | **Tidak perlu** |

> `gd` dipakai untuk PDF payslip, `zip` untuk import/export Excel.

## 1. Siapkan Laragon

1. Install **Laragon Full** dari https://laragon.org.
2. Pastikan versi PHP ≥ 8.3. Jika belum, unduh PHP 8.4 (*Thread Safe*, x64) dari https://windows.php.net/download, ekstrak ke `C:\laragon\bin\php\`, lalu pilih di **Menu → PHP → Version**.
3. Aktifkan ekstensi lewat **Menu → PHP → Extensions**: `gd`, `zip`, `intl`, `mbstring`, `pdo_mysql`, `fileinfo`.
4. Klik **Start All**.

## 2. Salin project

Salin folder project ke:

```
C:\laragon\www\hcms-ziah
```

Folder `vendor`, `node_modules`, dan file `.env` **tidak perlu** ikut disalin.

## 3. Install dependency & konfigurasi

Buka **Terminal** dari Laragon:

```bash
cd C:\laragon\www\hcms-ziah
composer install
copy .env.example .env
php artisan key:generate
```

Edit `.env` dan sesuaikan:

```env
APP_URL=http://hcms-ziah.test

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=hcms_ziah
DB_USERNAME=root
DB_PASSWORD=
```

## 4. Database

Buat database `hcms_ziah` (lewat tombol **Database** di Laragon / HeidiSQL, atau):

```bash
mysql -u root -e "CREATE DATABASE hcms_ziah CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci"
```

Lalu buat tabel, isi data awal, dan tautkan folder storage:

```bash
php artisan migrate --seed
php artisan storage:link
```

- `migrate --seed` membuat tabel serta mengisi role & permission, master data, dan data demo.
- `storage:link` wajib agar foto karyawan dan logo perusahaan tampil.

## 5. Jalankan

Klik **Menu → Apache → Reload** di Laragon, lalu buka:

```
http://hcms-ziah.test
```

### Akun demo

Semua password: `password`

| Email | Role |
|---|---|
| superadmin@hcms.test | Super Admin |
| hr@hcms.test | HR Admin |
| payroll@hcms.test | Payroll Officer |
| manager@hcms.test | Manager |
| karyawan@hcms.test | Karyawan |

> Ganti password akun-akun ini sebelum aplikasi dipakai sungguhan.

## 6. Email payslip (opsional)

Pengiriman payslip memakai antrean (`QUEUE_CONNECTION=database`). Jalankan worker di terminal terpisah agar email diproses:

```bash
php artisan queue:work
```

Bawaan `MAIL_MAILER=log` hanya menulis email ke `storage/logs/laravel.log`. Untuk mengirim sungguhan, isi SMTP di `.env`:

```env
MAIL_MAILER=smtp
MAIL_HOST=smtp.namadomain.com
MAIL_PORT=587
MAIL_USERNAME=user@namadomain.com
MAIL_PASSWORD=rahasia
MAIL_FROM_ADDRESS="hr@namadomain.com"
```

## 7. Verifikasi (opsional)

Buat database kosong `hcms_ziah_test`, lalu:

```bash
php artisan test
```

Semua test harus lulus.

## Alternatif: XAMPP

Jika memakai XAMPP, jangan buka aplikasi lewat `http://localhost/hcms-ziah/public`. Buat virtual host:

1. Tambahkan di `C:\xampp\apache\conf\extra\httpd-vhosts.conf`:

   ```apache
   <VirtualHost *:80>
       ServerName hcms-ziah.test
       DocumentRoot "C:/xampp/htdocs/hcms-ziah/public"
       <Directory "C:/xampp/htdocs/hcms-ziah/public">
           AllowOverride All
           Require all granted
       </Directory>
   </VirtualHost>
   ```

2. Tambahkan baris berikut di `C:\Windows\System32\drivers\etc\hosts` (buka Notepad sebagai Administrator):

   ```
   127.0.0.1 hcms-ziah.test
   ```

3. Restart Apache, lalu buka `http://hcms-ziah.test`.

Untuk percobaan cepat tanpa web server, bisa juga `php artisan serve` lalu buka `http://localhost:8000` (set `APP_URL` sesuai).

> **Jangan** memindahkan isi folder `public` ke root project atau menambah `.htaccess` redirect di root: file `.env` dan source code jadi bisa diakses dari browser.

## Troubleshooting

| Gejala | Solusi |
|---|---|
| URL harus pakai `/public` | Project tidak di `C:\laragon\www` / virtual host belum dibuat; reload Apache. |
| `Your PHP version does not satisfy...` | Ganti ke PHP ≥ 8.3. |
| Export Excel / PDF error | Aktifkan ekstensi `zip` dan `gd`. |
| Foto/logo tidak tampil | `php artisan storage:link` |
| Tampilan berantakan tanpa style | Browser tidak bisa mengakses internet (CDN). |
| Perubahan `.env` tidak berefek | `php artisan config:clear` |
| `SQLSTATE[HY000] [1049] Unknown database` | Database `hcms_ziah` belum dibuat. |
| Error 500 tanpa detail | Lihat `storage/logs/laravel.log`. |
