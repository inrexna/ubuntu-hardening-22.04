# 🔐 Hardening Ubuntu Server 22.04 (CIS Benchmark)

Dokumentasi ini berisi langkah-langkah hardening terhadap server Ubuntu 22.04 LTS berdasarkan panduan **CIS Benchmark v2.0.0**. Tujuannya adalah memperkuat keamanan sistem dari ancaman umum dengan menyesuaikan konfigurasi sistem dan layanan penting secara manual.

---

## ✅ 1. Update dan Patch Sistem (CIS 1.1.1)

```bash
apt update && apt upgrade -y
```

📄 **Dokumentasi:**  
Setelah perintah dijalankan, seluruh paket diperbarui. Dapat diverifikasi dengan `apt list --upgradable`.

<details>
<summary>📷 Lihat Screenshot</summary>
<img src="https://raw.githubusercontent.com/inrexna/ubuntu-hardening-22.04/main/src/image/1.png" alt="Update & Patch Screenshot" />
</details>

---

## ✅ 2. Nonaktifkan Layanan yang Tidak Digunakan (CIS 2.1.)

```bash
systemctl list-unit-files --type=service --state=enabled
systemctl disable apache2
```

📄 **Dokumentasi:**
<details>
<summary>📷 Lihat Screenshot</summary>
<img src="https://raw.githubusercontent.com/inrexna/ubuntu-hardening-22.04/main/src/image/2.png" />
<img src="https://raw.githubusercontent.com/inrexna/ubuntu-hardening-22.04/main/src/image/2.1.png" />
</details>

---

## ✅ 3. Konfigurasi SSH (CIS 5.2.)

```bash
nano /etc/ssh/sshd_config
```

```conf
PermitRootLogin no
PasswordAuthentication no
Protocol 2
AllowUsers adminuser
```

```bash
systemctl restart ssh
```

📄 **Dokumentasi:**
<details>
<summary>📷 Lihat Screenshot</summary>
<img src="https://raw.githubusercontent.com/inrexna/ubuntu-hardening-22.04/main/src/image/3.png" />
</details>

---

## ✅ 4. Konfigurasi Firewall dengan UFW (CIS 3.5.)

```bash
apt install ufw -y
ufw default deny incoming
ufw default allow outgoing
ufw allow ssh
ufw enable
```

📄 **Dokumentasi:**
<details>
<summary>📷 Lihat Screenshot</summary>
<img src="https://raw.githubusercontent.com/inrexna/ubuntu-hardening-22.04/main/src/image/4.png" />
</details>

---

## ✅ 5. Nonaktifkan IPv6 jika Tidak Digunakan (CIS 3.2.)

```conf
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
```

```bash
sysctl -p
```

📄 **Dokumentasi:**
<details>
<summary>📷 Lihat Screenshot</summary>
<img src="https://raw.githubusercontent.com/inrexna/ubuntu-hardening-22.04/main/src/image/5.png" />
<img src="https://raw.githubusercontent.com/inrexna/ubuntu-hardening-22.04/main/src/image/5.1.png" />
</details>

---

## ✅ 6. Nonaktifkan USB Storage (Opsional)

```bash
echo "install usb-storage /bin/true" >> /etc/modprobe.d/usb.conf
update-initramfs -u
```

📄 **Dokumentasi:**
(Deskripsi sudah cukup, tidak ada gambar)

---

## ✅ 7. Mengaktifkan Fail2Ban (CIS 5.3.)

```bash
apt install -y fail2ban
systemctl enable fail2ban
systemctl start fail2ban
```

📄 **Dokumentasi:**
<details>
<summary>📷 Lihat Screenshot</summary>
<img src="https://raw.githubusercontent.com/inrexna/ubuntu-hardening-22.04/main/src/image/7.png" />
<img src="https://raw.githubusercontent.com/inrexna/ubuntu-hardening-22.04/main/src/image/7.1.png" />
<img src="https://raw.githubusercontent.com/inrexna/ubuntu-hardening-22.04/main/src/image/7.2.png" />
<img src="https://raw.githubusercontent.com/inrexna/ubuntu-hardening-22.04/main/src/image/7.3.png" />
</details>

---

## ✅ 8. Mengaktifkan Auditd (CIS 4.1.)

```bash
apt install -y auditd audispd-plugins
systemctl enable auditd
systemctl start auditd
```

📄 **Dokumentasi:**
<details>
<summary>📷 Lihat Screenshot</summary>
<img src="https://raw.githubusercontent.com/inrexna/ubuntu-hardening-22.04/main/src/image/8.png" />
<img src="https://raw.githubusercontent.com/inrexna/ubuntu-hardening-22.04/main/src/image/8.1.png" />
<img src="https://raw.githubusercontent.com/inrexna/ubuntu-hardening-22.04/main/src/image/8.2.png" />
</details>

---

## ✅ 9. Mengatur Izin Akses Cron (CIS 5.1.)

```bash
touch /etc/cron.allow
chmod 600 /etc/cron.allow
chown root:root /etc/cron.allow
```

📄 **Dokumentasi:**
<details>
<summary>📷 Lihat Screenshot</summary>
<img src="https://raw.githubusercontent.com/inrexna/ubuntu-hardening-22.04/main/src/image/9.png" />
</details>

---

## ✅ 10. Mengatur Hak Akses File Penting (CIS 6.1.)

```bash
chmod 644 /etc/passwd
chmod 640 /etc/shadow
chmod 644 /etc/group
```

📄 **Dokumentasi:**
(Deskripsi sudah cukup, tidak ada gambar)

---

## ✅ 11. Konfigurasi Kernel Parameter (sysctl) (CIS 3.1.)

```bash
sysctl -p
```

📄 **Dokumentasi:**
(Deskripsi sudah cukup, tidak ada gambar)

---

## ✅ 12. Mengaktifkan dan Mengelola AppArmor (CIS 1.6.)

```bash
apt install apparmor apparmor-utils -y
systemctl enable apparmor
systemctl start apparmor
```

📄 **Dokumentasi:**
<details>
<summary>📷 Lihat Screenshot</summary>
<img src="https://raw.githubusercontent.com/inrexna/ubuntu-hardening-22.04/main/src/image/12.png" />
<img src="https://raw.githubusercontent.com/inrexna/ubuntu-hardening-22.04/main/src/image/12.1.png" />
</details>

---

## ✅ 13. Audit Keamanan Sistem dengan Lynis

```bash
apt install lynis -y
lynis audit system
```

📄 **Dokumentasi:**
<details>
<summary>📷 Lihat Screenshot</summary>
<img src="https://raw.githubusercontent.com/inrexna/ubuntu-hardening-22.04/main/src/image/13.png" />
<img src="https://raw.githubusercontent.com/inrexna/ubuntu-hardening-22.04/main/src/image/13.1.png" />
<img src="https://raw.githubusercontent.com/inrexna/ubuntu-hardening-22.04/main/src/image/12.2.png" />
</details>

---

## 📚 Referensi

- [CIS Ubuntu Linux 22.04 LTS Benchmark v2.0.0](https://www.cisecurity.org/benchmark/ubuntu_linux)
- Dokumentasi Ubuntu: https://ubuntu.com/security
- Log Audit: `/var/log/audit/audit.log`
- Fail2Ban Log: `/var/log/fail2ban.log`

---
