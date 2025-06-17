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
