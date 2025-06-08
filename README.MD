
# 🔐 Hardening Ubuntu Server 22.04 (CIS Benchmark)

Dokumentasi ini berisi langkah-langkah hardening terhadap server Ubuntu 22.04 LTS berdasarkan panduan **CIS Benchmark v2.0.0**. Tujuannya adalah memperkuat keamanan sistem dari ancaman umum dengan menyesuaikan konfigurasi sistem dan layanan penting secara manual.

---

## ✅ 1. Update dan Patch Sistem (CIS 1.1.1)

Melakukan pembaruan sistem untuk memastikan sistem menggunakan versi paket terbaru yang telah diperbaiki dari sisi keamanan.

```bash
apt update && apt upgrade -y
```

📄 **Dokumentasi:**  
Setelah perintah dijalankan, seluruh paket diperbarui. Dapat diverifikasi dengan `apt list --upgradable` (kosong berarti sudah up-to-date).
<img width="100%" align="centre" src="https://github.com/inrexna/ubuntu-hardening-22.04/src/image/1.png" />

---
---

## ✅ 2. Nonaktifkan Layanan yang Tidak Digunakan (CIS 2.1.)

Mengurangi permukaan serangan dengan menonaktifkan layanan yang tidak diperlukan.

```bash
systemctl list-unit-files --type=service --state=enabled
systemctl disable apache2
```

📄 **Dokumentasi:**  
Layanan seperti `apache2` yang tidak digunakan berhasil dinonaktifkan. Dicek kembali dengan `systemctl is-enabled apache2` → `disabled`.
<img width="100%" align="centre" src="https://github.com/inrexna/ubuntu-hardening-22.04/src/image/2.png" />
<img width="100%" align="centre" src="https://github.com/inrexna/ubuntu-hardening-22.04/src/image/2.1.png" />
---

## ✅ 3. Konfigurasi SSH (CIS 5.2.)

Mengamankan akses SSH agar hanya dapat digunakan oleh administrator terpercaya.

```bash
nano /etc/ssh/sshd_config
```

Ubah atau tambahkan konfigurasi berikut:

```conf
PermitRootLogin no
PasswordAuthentication no
Protocol 2
AllowUsers adminuser
```

Kemudian restart layanan SSH:

```bash
systemctl restart ssh
```

📄 **Dokumentasi:**  
SSH berhasil dikonfigurasi dengan membatasi akses login root, menonaktifkan password auth, dan mengizinkan user tertentu saja.
<img width="100%" align="centre" src="https://github.com/inrexna/ubuntu-hardening-22.04/src/image/3.png" />
---

## ✅ 4. Konfigurasi Firewall dengan UFW (CIS 3.5.)

Mengaktifkan dan mengatur firewall dasar untuk membatasi akses jaringan masuk.

```bash
apt install ufw -y
ufw default deny incoming
ufw default allow outgoing
ufw allow ssh
ufw enable
```

📄 **Dokumentasi:**  
Hanya port SSH (`22/tcp`) yang terbuka dari luar. Dapat diverifikasi dengan `ufw status verbose`.
<img width="100%" align="centre" src="https://github.com/inrexna/ubuntu-hardening-22.04/src/image/4.png" />
---

## ✅ 5. Nonaktifkan IPv6 jika Tidak Digunakan (CIS 3.2.)

Jika tidak menggunakan IPv6, sebaiknya dinonaktifkan untuk mencegah eksploitasi di jaringan.

```bash
nano /etc/sysctl.conf
```

Tambahkan:

```conf
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
```

Lalu terapkan:

```bash
sysctl -p
```

📄 **Dokumentasi:**  
IPv6 berhasil dinonaktifkan. Dapat diverifikasi dengan `ip a` atau `cat /proc/sys/net/ipv6/conf/all/disable_ipv6`.
<img width="100%" align="centre" src="https://github.com/inrexna/ubuntu-hardening-22.04/src/image/5.png" />
<img width="100%" align="centre" src="https://github.com/inrexna/ubuntu-hardening-22.04/src/image/5.1.png" />
---

## ✅ 6. Nonaktifkan USB Storage (Opsional)

Mencegah penggunaan perangkat USB yang tidak sah.

```bash
echo "install usb-storage /bin/true" >> /etc/modprobe.d/usb.conf
update-initramfs -u
```

📄 **Dokumentasi:**  
USB storage tidak bisa digunakan. Diuji dengan mencolokkan flashdisk dan dicek melalui `dmesg` (tidak terdeteksi sebagai storage).

---

## ✅ 7. Mengaktifkan Fail2Ban (CIS 5.3.)

Fail2Ban membantu melindungi server dari serangan brute-force seperti login SSH berulang dengan memblokir IP yang mencurigakan secara otomatis berdasarkan log autentikasi.

```bash
apt install -y fail2ban
systemctl enable fail2ban
systemctl start fail2ban
```

Kemudian edit konfigurasi dasar:

```bash
nano /etc/fail2ban/jail.local
```

Isi:

```conf
[sshd]
enabled = true
port = 22
logpath = /var/log/auth.log
maxretry = 3
```

Restart Fail2Ban agar konfigurasi diterapkan:

```bash
systemctl restart fail2ban
```

📄 **Dokumentasi:**  
Fail2Ban berhasil berjalan dan dikonfigurasi untuk melindungi SSH. Uji coba brute-force login gagal sebanyak 3 kali memicu pemblokiran IP. Status dapat dicek dengan:

```bash
fail2ban-client status sshd
systemctl status fail2ban
```
<img width="100%" align="centre" src="https://github.com/inrexna/ubuntu-hardening-22.04/src/image/7.png" />
<img width="100%" align="centre" src="https://github.com/inrexna/ubuntu-hardening-22.04/src/image/7.1.png" />
<img width="100%" align="centre" src="https://github.com/inrexna/ubuntu-hardening-22.04/src/image/7.2.png" />
<img width="100%" align="centre" src="https://github.com/inrexna/ubuntu-hardening-22.04/src/image/7.3.png" />

---

## ✅ 8. Mengaktifkan Auditd (CIS 4.1.)

Auditd (Audit Daemon) digunakan untuk mencatat aktivitas penting di sistem, seperti perubahan file konfigurasi, aktivitas user, hingga eksekusi perintah tertentu.

```bash
apt install -y auditd audispd-plugins
systemctl enable auditd
systemctl start auditd
```

Contoh menambahkan aturan untuk memantau perubahan file sensitif:

```bash
auditctl -w /etc/passwd -p wa -k passwd_changes
auditctl -w /etc/shadow -p wa -k shadow_changes
```

📄 **Dokumentasi:**  
Auditd berhasil aktif dan digunakan untuk mencatat perubahan pada file `/etc/passwd` dan `/etc/shadow`.  
Log hasil audit dapat dilihat dengan:

```bash
ausearch -k passwd_changes
```

atau langsung dari file:

```bash
cat /var/log/audit/audit.log
```
<img width="100%" align="centre" src="https://github.com/inrexna/ubuntu-hardening-22.04/src/image/8.png" />
<img width="100%" align="centre" src="https://github.com/inrexna/ubuntu-hardening-22.04/src/image/8.1.png" />
<img width="100%" align="centre" src="https://github.com/inrexna/ubuntu-hardening-22.04/src/image/8.2.png" />
---

## ✅ 9. Mengatur Izin Akses Cron (CIS 5.1.)

Cron adalah layanan penjadwalan tugas otomatis. Tanpa pembatasan, pengguna biasa bisa menjadwalkan perintah berbahaya. Maka, hanya user tertentu yang boleh mengakses `cron`.

```bash
touch /etc/cron.allow
chmod 600 /etc/cron.allow
chown root:root /etc/cron.allow

touch /etc/at.allow
chmod 600 /etc/at.allow
chown root:root /etc/at.allow
```

📄 **Dokumentasi:**  
Setelah konfigurasi, hanya user yang ditambahkan ke dalam file `/etc/cron.allow` yang dapat menggunakan `cron`. Hal ini meminimalkan risiko penyalahgunaan cron oleh user biasa.
<img width="100%" align="centre" src="https://github.com/inrexna/ubuntu-hardening-22.04/src/image/9.png" />
---

## ✅ 10. Mengatur Hak Akses File Penting (CIS 6.1.)

Beberapa file sistem menyimpan konfigurasi sensitif yang **tidak boleh bisa diubah oleh user biasa**. Berikut adalah langkah umum untuk meninjau dan mengamankan file kritikal:

### `/etc/passwd`

```bash
chmod 644 /etc/passwd
chown root:root /etc/passwd
```

### `/etc/shadow`

```bash
chmod 640 /etc/shadow
chown root:shadow /etc/shadow
```

### `/etc/group`

```bash
chmod 644 /etc/group
chown root:root /etc/group
```

📄 **Dokumentasi:**  
Setelah audit dan penyesuaian permission:
- File `/etc/shadow` hanya bisa dibaca oleh user `root` dan group `shadow`, tidak bisa diakses publik.
- Hak akses diverifikasi dengan `stat` dan `ls -l`, memastikan tidak ada permission berlebih (`world-writable`).


## ✅ 11. Konfigurasi Kernel Parameter (sysctl) (CIS 3.1.)

Menyesuaikan parameter kernel menggunakan `sysctl` untuk memperketat keamanan jaringan dan sistem.

```bash
nano /etc/sysctl.conf
```

Tambahkan konfigurasi berikut:

```conf
# Disable IP forwarding
net.ipv4.ip_forward = 0
net.ipv6.conf.all.forwarding = 0

# Disable packet redirect
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.conf.default.accept_redirects = 0

# Enable source routing protection
net.ipv4.conf.all.send_redirects = 0
net.ipv4.conf.default.send_redirects = 0

# Enable IP spoofing protection
net.ipv4.conf.all.rp_filter = 1
net.ipv4.conf.default.rp_filter = 1

# Log suspicious packets
net.ipv4.conf.all.log_martians = 1
net.ipv4.conf.default.log_martians = 1
```

Terapkan perubahan:

```bash
sysctl -p
```

📄 **Dokumentasi:**  
Parameter berhasil dimuat ulang. Pengaturan ini membantu memperketat keamanan jaringan dan menonaktifkan fitur yang tidak dibutuhkan.

---

## ✅ 12. Mengaktifkan dan Mengelola AppArmor (CIS 1.6.)

AppArmor adalah mandatory access control (MAC) yang membatasi program berdasarkan profil aksesnya.

Aktifkan AppArmor jika belum:

```bash
apt install apparmor apparmor-utils -y
systemctl enable apparmor
systemctl start apparmor
```

Cek status profil:

```bash
aa-status
```

📄 **Dokumentasi:**  
AppArmor aktif dan berjalan. Semua profil dalam mode `enforce` membatasi akses aplikasi seperti `sshd`, `apache2`, dan `mysql` sesuai kebijakan keamanan.
<img width="100%" align="centre" src="https://github.com/inrexna/ubuntu-hardening-22.04/src/image/12.png" />
<img width="100%" align="centre" src="https://github.com/inrexna/ubuntu-hardening-22.04/src/image/12.1.png" />
---

## ✅ 13. Audit Keamanan Sistem dengan Lynis

Lynis adalah tool audit keamanan untuk sistem berbasis Unix/Linux.

Install dan jalankan Lynis:

```bash
apt install lynis -y
lynis audit system
```

📄 **Dokumentasi:**  
Lynis akan memeriksa konfigurasi, permission, status layanan, dan memberikan rekomendasi perbaikan. Hasil laporan dapat ditemukan di:
<img width="100%" align="centre" src="https://github.com/inrexna/ubuntu-hardening-22.04/src/image/13.png" />
<img width="100%" align="centre" src="https://github.com/inrexna/ubuntu-hardening-22.04/src/image/13.1.png" />
<img width="100%" align="centre" src="https://github.com/inrexna/ubuntu-hardening-22.04/src/image/12.2.png" />
```
/var/log/lynis.log
/var/log/lynis-report.dat
```

---

## 📚 Referensi

- [CIS Ubuntu Linux 22.04 LTS Benchmark v2.0.0](https://www.cisecurity.org/benchmark/ubuntu_linux)
- Dokumentasi Ubuntu: https://ubuntu.com/security
- Log Audit: `/var/log/audit/audit.log`
- Fail2Ban Log: `/var/log/fail2ban.log`

---