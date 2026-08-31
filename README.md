# Konfigurasi Apache Web Server HTTPS di Debian 10

Dokumentasi praktek konfigurasi web server Apache2 dengan HTTPS (self-signed certificate menggunakan OpenSSL) di Debian 10, dijalankan di VirtualBox.

## Topologi
- 1 VM Debian 10 sebagai server
- Host OS sebagai client
- Network: Dual adapter
  - `enp0s3` → NAT (akses internet, `apt install`)
  - `enp0s8` → Host-only Adapter, IP static `192.168.28.10/24`

## Langkah Konfigurasi

### 1. Install Apache2 & OpenSSL
```bash
apt update
apt install apache2 openssl ssl-cert
```

### 2. Konfigurasi Virtual Host SSL
```bash
cd /etc/apache2/sites-available
cp default-ssl.conf webssl.conf
```

Edit `webssl.conf`, ubah:
- `DocumentRoot` → `/var/www/sslhtml`
- `SSLEngine` → `on`
- `SSLCertificateFile` → `/etc/apache2/ssl/apache.crt`
- `SSLCertificateKeyFile` → `/etc/apache2/ssl/apache.key`

### 3. Buat Halaman Web
```bash
mkdir /var/www/sslhtml
vi /var/www/sslhtml/index.html
```

### 4. Generate Private Key & Certificate
```bash
mkdir /etc/apache2/ssl
openssl req -x509 -newkey rsa:4096 -keyout /etc/apache2/ssl/apache.key -out /etc/apache2/ssl/apache.crt -nodes -days 365
```

### 5. Enable Site & Module SSL
```bash
a2dissite 000-default.conf
a2ensite webssl.conf
a2enmod ssl
```

### 6. Restart Apache
```bash
systemctl restart apache2
```

### 7. Testing dari Client
Akses melalui browser:
https://192.168.28.10


## Troubleshooting

| Masalah | Solusi |
|---|---|
| `Bad archive mirror` saat instalasi | Debian 10 sudah EOL, ganti mirror ke `archive.debian.org` |
| `a2dissite: command not found` | PATH tidak include `/usr/sbin`, jalankan `export PATH=$PATH:/usr/sbin:/sbin` atau pakai path lengkap |
| `SSLCertificateFile: file does not exist` | Pastikan nama file certificate sesuai dengan yang ditulis di config (`apache.crt`, bukan `apache2.crt`) |
| Browser timeout saat akses IP | Cek subnet Host-only Adapter di VirtualBox (Host Network Manager), samakan dengan IP static di `/etc/network/interfaces` |

## Referensi
- [Konfigurasi Webserver Apache HTTPS Debian - rafimf.gitlab.io](https://rafimf.gitlab.io/notes/network/linux/konfigurasi-webserver-apache-https-debian/)
