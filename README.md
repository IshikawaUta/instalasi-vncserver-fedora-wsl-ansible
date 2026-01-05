# Remote Desktop Gateway Fedora WSL

Proyek ini menggunakan **Ansible** untuk mengotomatisasi instalasi lingkungan desktop XFCE di Fedora WSL, serta menyediakan akses melalui web (noVNC) dan Cloudflare Tunnel agar bisa diakses dari mana saja secara aman.

## 📋 Prasyarat

Sebelum memulai, pastikan Anda berada di distro Fedora WSL dan memiliki hak akses `sudo`.

### 1. Instalasi Ansible
Jalankan perintah berikut untuk memasang Ansible di Fedora:
```bash
sudo dnf update -y
sudo dnf install git ansible -y
git clone https://github.com/IshikawaUta/instalasi-vncserver-fedora-wsl-ansible
cd instalasi-vncserver-fedora-wsl-ansible
```

---

## 🔐 Manajemen Rahasia (secrets.yml)

Proyek ini memerlukan file `secrets.yml` untuk menyimpan token Cloudflare dan password VNC.

### A. Membuat file secrets.yml

Buat file baru bernama `secrets.yml` di folder utama proyek:

```bash
ansible-vault create secrets.yml
```

Isi dengan format berikut:


```yaml
vault_cloudflare_token: "masukkan_token_tunnel_anda"
vault_vnc_password: "masukkan_password_vnc_anda"

```

## Cara Mendapatkan vault_cloudflare_token

Untuk mendapatkan **vault_cloudflare_token**, ikuti langkah berikut:

1. **Buka Dashboard:** Masuk ke [Cloudflare Zero Trust](https://one.dash.cloudflare.com/).
2. **Buat Tunnel:** Pergi ke **Networks** > **Tunnels** > **Create a Tunnel**.
3. **Pilih Cloudflared:** Beri nama tunnel (misal: `fedora-wsl`), lalu klik **Save**.
4. **Salin Token:** Di bagian "Install and run a connector", pilih environment Docker. Salin kode token panjang yang muncul (setelah teks `--token`).
5. **Set Up Hostname:** * Klik tab **Public Hostname** pada tunnel yang baru dibuat.
* Klik **Add a public hostname**.
* Isi **Subdomain** (misal: `desktop`) dan pilih **Domain** Anda.
* Pilih **Service Type:** `HTTP` dan **URL:** `novnc-fedora:8080`.
* Klik **Save**.

---

## 🚀 Instalasi Proyek

Ikuti urutan perintah berikut untuk menjalankan otomatisasi:

### 1. Siapkan Struktur Folder

Pastikan file `inventory.ini`, `playbook.yml`, dan folder `templates/` yang berisi `docker-compose.j2` serta `xstartup.j2`  sudah tersedia.

### 2. Jalankan Playbook

Pastikan anda berada pada directory proyek ini:

```bash
ansible-playbook -i inventory.ini playbook.yml -K --ask-vault-pass

```

### 3. Melihat proses install

Karena tidak ada output dari paket dan ukuran yang diinstall oleh ansible maka untuk melihat apakah instalasi sedang berjalan kita bisa memantau lalu lintas jaringan:

```bash
grep "eth0" /proc/net/dev
```

Output dari command diatas:

### Statistik Network Interface (`eth0`)

| Interface | Bagian | Bytes | Packets | Errs | Drop | FIFO | Frame/Colls | Compressed |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **eth0** | **Receive (RX)** | 54,321,010 | 125,430 | 0 | 0 | 0 | 0 | 0 |
|  | **Transmit (TX)** | 9,876,543 | 85,420 | 0 | 0 | 0 | 0 | 0 |

---

### Penjelasan Singkat Kolom:

* **Receive (RX):** Data yang masuk ke komputer Anda (Download).
* **Transmit (TX):** Data yang dikirim dari komputer Anda (Upload).
* **Bytes:** Ukuran total data dalam satuan Byte. (Contoh: 54.321.010 bytes  54.3 MB).
* **Packets:** Jumlah paket data yang dikirim/terima.
* **Errs/Drop:** Jika angka ini di atas 0, biasanya menunjukkan adanya gangguan pada jaringan atau kabel.

---

## 🛠️ Detail Teknis

Setelah perintah selesai dijalankan, sistem akan otomatis melakukan:

* **Desktop**: Memasang XFCE Desktop Environment.
* **VNC Server**: Menjalankan TigerVNC pada display `:5` (port 5905).
* 
**noVNC**: Menjalankan kontainer Docker yang mengubah VNC ke protokol HTTP di port 6080.


* **Cloudflare**: Menghubungkan port lokal tersebut ke internet melalui Tunnel.

---

## 🖥️ Cara Akses

1. **Lokal**: Buka browser dan akses `http://localhost:6080`.
2. **Publik**: Akses melalui domain yang telah Anda hubungkan di Cloudflare Zero Trust Dashboard.

---

## 🔧 Troubleshooting

Jika anda mendapat masalah seperti layar hitam anda bisa lakukan langkah-langkah ini:

1. **Hentikan Vncserver**: `vncserver -kill :5`
2. **Hapus Cache**: `sudo rm -f /tmp/.X5-lock && sudo rm -f /tmp/.X11-unix/X5`
3. **Izin Akses File**: `chmod +x ~/.vnc/xstartup`, `sudo chown -R $USER:$USER ~/.vnc`, `sudo chown $USER:$USER ~/.Xauthority`
4. **Jalankan Kembali Vnc**: `WAYLAND_DISPLAY= DISPLAY=:5 vncserver :5 -geometry 1366x768 -depth 24`

---

## 🐳 Check Container Docker

Buat Izin untuk user dan check container docker:

1. **Beri Izin Untuk User**: `sudo usermod -aG docker $USER`
2. **Terapkan Perubahan Tanpa Restart**: `newgrp docker`
3. **Check Container Docker**: `docker ps -a`

Pastikan status container **Up** bukan **Exited**

---

jikan ada pertanyaan atau ingin kolaborasi anda bisa menghubungi [Disini.](https://wa.me/+62895701060973)
