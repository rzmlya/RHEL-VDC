# 🔧 Integrasi Red Hat VDC License di Huawei DCS menggunakan `virt-who`

Dokumentasi ini menjelaskan cara mengatur `virt-who` pada server RHEL untuk memanfaatkan **lisensi Red Hat Enterprise Linux for Virtual Datacenters (VDC)** di lingkungan **Huawei DCS** (misalnya FusionCompute), yang secara resmi **tidak didukung langsung oleh Red Hat**.

Karena Huawei DCS tidak menyediakan integrasi langsung dengan `virt-who`, kita akan menggunakan **mode manual mapping**.

---

## 🧱 Arsitektur Sistem

| RHEL + virt-who        | Huwawei DCS Host(s)                | RHEL Guest VMs    | 
| :--------------------- | :--------------------------------- | :-------------    |
| (manual mapping file)  | (FusionCompute)                    | (VM01, VM02, ...) |

## 1. Daftarkan RHEL ke Red Hat Subscription Manager
```zsh
sudo subscription-manager register
```
Masukkan Red Hat Customer Portal credentials.

Lalu, attach subscription:
```zsh
sudo subscription-manager attach --auto
```
Verifikasi:
```zsh
sudo subscription-manager status
```

## 2. Install `virt-who`
```zsh
sudo dnf install virt-who -y
```

## 3. Buat Direktori Konfigurasi (jika belum ada)
```zsh
sudo mkdir -p /etc/virt-who.d
```

## 4. Nonaktifkan konfigurasi default
Agar tidak konflik dengan konfigurasi manual:
```zsh
sudo sed -i 's/^enabled *=.*/enabled=0/' /etc/virt-who.conf
```
Atau kamu bisa menambahkan:
```yaml
[global]
interval=3600
debug=true
oneshot=false
```

## 5. Buat File Konfigurasi Manual Mapping
Buat file konfigurasi `.conf` di `/etc/virt-who.d/huawei.conf`:
```yaml
[huawei-manual]
type=file
file=/etc/virt-who.d/huawei-mapping.json
owner=your_org_id
env=Library
hypervisor_id=hostname
```
Ganti `your_org_id` dengan output dari perintah:
```zsh
subscription-manager identity
```

## 6. Buat File Mapping JSON
Misalnya kamu punya 3 VM RHEL di host Huawei bernama huawei-dcs-node-1, dengan UUID VM sebagai berikut:
```zsh
# Dijalankan dalam masing-masing VM RHEL
sudo dmidecode -s system-uuid
```
Contoh `/etc/virt-who.d/huawei-mapping.json`:
```json
[
  {
    "guestId": "12345678-1234-5678-1234-567812345678",
    "hostId": "huawei-dcs-node-1"
  },
  {
    "guestId": "23456789-2345-6789-2345-678923456789",
    "hostId": "huawei-dcs-node-1"
  },
  {
    "guestId": "34567890-3456-7890-3456-789034567890",
    "hostId": "huawei-dcs-node-1"
  }
]
```

## 7. Set Hak Akses File
```zsh
sudo chmod 600 /etc/virt-who.d/huawei.conf
sudo chmod 600 /etc/virt-who.d/huawei-mapping.json
```

## 8. Enable dan Jalankan `virt-who`
```zsh
sudo systemctl enable virt-who
sudo systemctl start virt-who
```

## 9. Cek Log untuk Verifikasi
```zsh
sudo tail -f /var/log/virt-who/virt-who.log
```
Pastikan log menunjukkan tidak ada error dan mapping berhasil dikirim.
Contoh log berhasil:
```zsh
INFO: Reporting mapping for host 'huawei-dcs-node-1' and guest '12345678-1234-5678-1234-567812345678'
```

## 10. Verifikasi di Red Hat Customer Portal (atau Satellite)
- Buka https://access.redhat.com
- Masuk ke menu Subscriptions > Systems
- Pilih salah satu guest RHEL → pastikan ada mapping ke Hypervisor

## 📌 Tips Tambahan
- Pastikan timezone server 'virt-who' sinkron dengan NTP ('timedatectl status')
- Jika ingin debug lebih lanjut:
```zsh
sudo journalctl -u virt-who
```



# ✅ LANGKAH KONFIGURASI DI SETIAP VM RHEL (Agar Bisa Gunakan Repo Resmi)

## 1. Pastikan VM Terkoneksi ke Internet
Minimal bisa curl `https://cdn.redhat.com`
Jika pakai proxy/firewall, pastikan outbound port 443 (HTTPS) ke Red Hat dibuka.

## 2. Daftarkan VM ke Red Hat Subscription Manager
```zsh
sudo subscription-manager register --auto-attach
```
- Jangan khawatir, meskipun kamu tidak attach manual, RHSM akan otomatis menempelkan VDC subscription jika `virt-who` sudah kirim mapping host-guest yang valid.
- Jika ditanya username/password, masukkan yang sama seperti di `virt-who` server.

## 3. Verifikasi Status Subscription
```zsh
sudo subscription-manager status
```
Jika berhasil, harusnya keluar seperti ini:
```zsh
Overall Status: Current
```
Lalu:
```zsh
sudo subscription-manager list --consumed
```
Harusnya terlihat sesuatu seperti:
```zsh
Subscription Name: Red Hat Enterprise Linux for Virtual Datacenters
```

## 4. Aktifkan Repositori Sesuai Kebutuhan
Contoh: Untuk RHEL 9
```zsh
sudo subscription-manager repos --enable=rhel-9-for-x86_64-baseos-rpms
sudo subscription-manager repos --enable=rhel-9-for-x86_64-appstream-rpms
```
Untuk RHEL 8:
```zsh
sudo subscription-manager repos --enable=rhel-8-for-x86_64-baseos-rpms
sudo subscription-manager repos --enable=rhel-8-for-x86_64-appstream-rpms
```

## 5. Update dan Uji Repo
```zsh
sudo dnf clean all
sudo dnf repolist
sudo dnf update -y
```
Harusnya muncul daftar repos resmi Red Hat seperti:
```zsh
repo id                               repo name
rhel-9-for-x86_64-baseos-rpms        Red Hat Enterprise Linux 9 for x86_64 - BaseOS
rhel-9-for-x86_64-appstream-rpms     Red Hat Enterprise Linux 9 for x86_64 - AppStream
```
## ⚠️ Jika `subscription-manager` register Gagal
Beberapa hal yang perlu dicek:
- Apakah `virt-who` sudah mengirim mapping ke RHSM (cek log `virt-who`)
- Apakah VM bisa `ping` atau `curl` ke `cdn.redhat.com`
- Coba register ulang pakai username/password jika `--auto-attach` gagal




















