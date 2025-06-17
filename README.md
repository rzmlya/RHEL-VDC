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
