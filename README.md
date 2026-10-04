# 📚 Dokumentasi XDTunnel Xray CLI

## Daftar Isi
1. [Pendahuluan](#pendahuluan)
2. [Instalasi](#instalasi)
3. [Konfigurasi](#konfigurasi)
4. [Perintah Dasar](#perintah-dasar)
5. [Perintah `add`](#perintah-add)
6. [Perintah `list`](#perintah-list)
7. [Perintah `get`](#perintah-get)
8. [Perintah `delete`](#perintah-delete)
9. [Perintah `check`](#perintah-check)
10. [Script Menu Interaktif](#script-menu-interaktif)
11. [Contoh Alur Kerja](#contoh-alur-kerja)
12. [Troubleshooting](#troubleshooting)

---

## Pendahuluan

**XDTunnel Xray CLI** (`xray-cli`) adalah command-line interface untuk manajemen user Xray-core melalui gRPC API. CLI ini mendukung 4 protokol dengan masing-masing 4 transport, sehingga total **16 inbound** dapat dikelola dari satu perintah.

### Fitur Utama
- ✅ Tambah user ke 1 inbound atau semua sekaligus
- ✅ List semua user dengan UUID/password
- ✅ Detail 1 user (cek UUID/password)
- ✅ Hapus user dari semua inbound
- ✅ Cek IP online (login) per user
- ✅ Cek traffic (kuota) per user
- ✅ Output JSON konsisten
- ✅ Proteksi user default (`xd-tunnel-vpns`)

### Protokol & Transport yang Didukung

| Protokol | Transport |
|---|---|
| VMess | ws, grpc, xhttp, httpupgrade |
| VLESS | ws, grpc, xhttp, httpupgrade |
| Trojan | ws, grpc, xhttp, httpupgrade |
| Shadowsocks | ws, grpc, xhttp, httpupgrade |

### Credit
- **XDTunnel**
- Telegram: [t.me/SukaPediaCS](https://t.me/SukaPediaCS)
- WhatsApp: [wa.me/6285935195701](https://wa.me/6285935195701)

---

## Instalasi

### Prasyarat
- **Go** versi **1.24+** (untuk build)
- **Xray-core** versi **25.10.15** atau lebih baru, sudah berjalan dengan config API aktif
- **grpcurl** (opsional, untuk cek API)

### 1. Build Binary

```bash
cd ~/grpc
go mod tidy
go build -o /usr/local/bin/xray-cli main.go
```

### 2. Verifikasi

```bash
xray-cli
```

Jika muncul banner XDTunnel, instalasi berhasil.

### 3. (Opsional) Install Script Menu

```bash
sudo nano /usr/local/bin/xdtunnel
# paste script menu interaktif
sudo chmod +x /usr/local/bin/xdtunnel
```

---

## Konfigurasi

### `config.json` Xray

Pastikan bagian berikut ada di config Xray:

```json
{
  "api": {
    "services": [
      "StatsService",
      "HandlerService",
      "LoggerService",
      "RoutingService",
      "ReflectionService"
    ],
    "tag": "api"
  },
  "inbounds": [
    {
      "listen": "127.0.0.1",
      "port": 1001,
      "protocol": "dokodemo-door",
      "settings": { "address": "127.0.0.1" },
      "tag": "api"
    }
  ],
  "routing": {
    "rules": [
      {
        "type": "field",
        "inboundTag": ["api"],
        "outboundTag": "api"
      }
    ]
  }
}
```

### Alamat API

Default CLI mengarah ke `127.0.0.1:1001`. Jika port berbeda, ubah konstanta di `main.go`:

```go
const apiAddr = "127.0.0.1:1001"
```

### User Default

CLI memproteksi user default bernama **`xd-tunnel-vpns`**. User ini:
- Tidak muncul di `list`
- Tidak bisa di-`get`
- Tidak bisa di-`add` ulang
- Tidak bisa di-`delete`
- Tidak bisa di-`check`

Ubah di `main.go` bila perlu:

```go
var excludedUsers = []string{"xd-tunnel-vpns"}
```

---

## Perintah Dasar

```
xray-cli <command> [args]
```

| Command | Fungsi |
|---|---|
| `add` | Tambah user baru |
| `list` / `ls` | Tampilkan daftar user |
| `get` / `show` | Detail 1 user |
| `delete` / `del` / `rm` | Hapus user |
| `check` | Cek IP login / traffic |
| (kosong) | Tampilkan help |

---

## Perintah `add`

Menambahkan user baru ke 1 atau semua inbound.

### Sintaks

```bash
xray-cli add <target> <user> <uuid|password>
```

### Parameter

| Parameter | Keterangan |
|---|---|
| `<target>` | Target inbound |
| `<user>` | Nama unik user (email) |
| `<uuid\|password>` | UUID untuk vmess/vless, password untuk trojan/shadowsocks |

### Nilai `<target>`

| Nilai | Efek |
|---|---|
| `all` | Tambah ke **semua** 16 inbound |
| `vmess` | Tambah ke 4 inbound VMess |
| `vless` | Tambah ke 4 inbound VLESS |
| `trojan` | Tambah ke 4 inbound Trojan |
| `shadowsocks` | Tambah ke 4 inbound Shadowsocks |
| `vmess-ws` | Tambah ke 1 inbound spesifik |
| `vless-grpc` | Tambah ke 1 inbound spesifik |
| `trojan-upgrade` | Tambah ke 1 inbound spesifik |
| `shadowsocks-xhttp` | Tambah ke 1 inbound spesifik |

### Contoh

```bash
# Tambah user vmess ke semua transport vmess
xray-cli add vmess user1@x.com c41b2434-1e24-4fc3-b5fe-fb9376c72dc0

# Tambah user vmess hanya ke transport ws
xray-cli add vmess-ws user1@x.com c41b2434-1e24-4fc3-b5fe-fb9376c72dc0

# Tambah user trojan
xray-cli add trojan user2@x.com rahasia123

# Tambah user shadowsocks
xray-cli add shadowsocks user3@x.com passsocks123

# Tambah user ke SEMUA protokol & transport
xray-cli add all user4@x.com mypassword
```

### Output Sukses

```json
{
    "status": true,
    "message": "berhasil menambahkan user 'user1@x.com' ke 4 inbound",
    "data": [
        {
            "type": "all",
            "protocol": "vmess",
            "user": "user1@x.com",
            "uuid": "c41b2434-1e24-4fc3-b5fe-fb9376c72dc0"
        }
    ]
}
```

### Output Duplikat

```json
{
    "status": false,
    "message": "user 'user1@x.com' sudah ada di semua target yang diminta (4 inbound), tidak ada yang ditambahkan",
    "data": []
}
```

### Exit Code
- `0` — status `true`
- `1` — status `false`

---

## Perintah `list`

Menampilkan daftar user dari 1 atau semua inbound.

### Sintaks

```bash
xray-cli list [target]
```

### Contoh

```bash
xray-cli list              # semua
xray-cli list vmess        # vmess saja
xray-cli list vmess-ws     # inbound vmess-ws saja
```

### Output

```json
{
    "status": true,
    "message": "ditemukan 2 user pada target 'vmess'",
    "data": [
        {
            "type": "ws",
            "protocol": "vmess",
            "user": "user1@x.com",
            "uuid": "c41b2434-1e24-4fc3-b5fe-fb9376c72dc0"
        }
    ]
}
```

### Catatan
- User default `xd-tunnel-vpns` tidak akan muncul
- Untuk trojan/shadowsocks, field `uuid` diganti `password`

---

## Perintah `get`

Menampilkan detail 1 user beserta UUID/password di semua inbound.

### Sintaks

```bash
xray-cli get <user>
```

### Contoh

```bash
xray-cli get user1@x.com
```

### Output

```json
{
    "status": true,
    "message": "detail user 'user1@x.com' ditemukan di 4 inbound",
    "data": [
        {
            "type": "ws",
            "protocol": "vmess",
            "user": "user1@x.com",
            "uuid": "c41b2434-1e24-4fc3-b5fe-fb9376c72dc0"
        }
    ]
}
```

### Output Tidak Ditemukan

```json
{
    "status": false,
    "message": "user 'tidak-ada@x.com' tidak ditemukan di inbound manapun",
    "data": []
}
```

---

## Perintah `delete`

Menghapus user dari semua inbound tempat user tersebut terdaftar.

### Sintaks

```bash
xray-cli delete <user>
```

### Contoh

```bash
xray-cli delete user1@x.com
```

### Output Sukses

```json
{
    "status": true,
    "message": "user 'user1@x.com' berhasil dihapus dari 4 inbound",
    "data": []
}
```

### Output Tidak Ditemukan

```json
{
    "status": false,
    "message": "user 'tidak-ada@x.com' tidak ditemukan di inbound manapun",
    "data": []
}
```

### Output User Default (Ditolak)

```json
{
    "status": false,
    "message": "user 'xd-tunnel-vpns' adalah user default dan tidak boleh dihapus",
    "data": []
}
```

---

## Perintah `check`

Cek IP online (login) atau traffic (quota) user.

### Sintaks

```bash
xray-cli check <protocol> <mode> <target>
```

### Parameter

| Parameter | Nilai |
|---|---|
| `<protocol>` | `all`, `vmess`, `vless`, `trojan`, `shadowsocks` |
| `<mode>` | `login` (IP online) atau `quota` (traffic) |
| `<target>` | `all` (semua user) atau nama user |

### Mode `login` — Cek IP Online

```bash
xray-cli check vmess login all
xray-cli check vmess login user1@x.com
```

**Output:**

```json
{
    "status": true,
    "message": "ditemukan 1 user dengan total 2 IP online",
    "data": [
        {
            "type": "ws",
            "protocol": "vmess",
            "user": "user1@x.com",
            "ip_count": 2,
            "ips": [
                { "ip": "103.28.12.45", "last_seen": 1745678901 },
                { "ip": "114.79.16.203", "last_seen": 1745678923 }
            ]
        }
    ]
}
```

> **Catatan:** Hanya menampilkan IP yang **sedang online**. IP yang sudah disconnect tidak akan muncul.

### Mode `quota` — Cek Traffic

```bash
xray-cli check vmess quota all
xray-cli check trojan quota user2@x.com
```

**Output:**

```json
{
    "status": true,
    "message": "ditemukan 2 user dengan total pemakaian 15.32 MB",
    "data": [
        {
            "type": "ws",
            "protocol": "vmess",
            "user": "user1@x.com",
            "uplink_bytes": 2520,
            "downlink_bytes": 2040,
            "total_bytes": 4560,
            "uplink": "2.46 KB",
            "downlink": "1.99 KB",
            "total": "4.45 KB"
        }
    ]
}
```

> **Catatan:** Traffic adalah **akumulasi sejak Xray dinyalakan**. Restart Xray akan mereset angka.

### Output Tidak Ditemukan

```json
{
    "status": false,
    "message": "user 'tidak-ada@x.com' tidak ditemukan pada protocol 'vmess'",
    "data": []
}
```

---

## Script Menu Interaktif

Script `xdtunnel` menyediakan antarmuka menu berbasis teks untuk memudahkan operator non-teknis.

### Instalasi

```bash
sudo nano /usr/local/bin/xdtunnel
# paste script menu
sudo chmod +x /usr/local/bin/xdtunnel
```

### Cara Pakai

```bash
xdtunnel
```

### Menu yang Tersedia

| Menu | Fungsi |
|---|---|
| `[1]` Add User | Tambah user baru |
| `[2]` List User | Daftar user |
| `[3]` Get User (Detail UUID) | Cek UUID/password 1 user |
| `[4]` Delete User | Hapus user |
| `[5]` Check Login (IP Online) | Cek IP online |
| `[6]` Check Quota (Traffic) | Cek traffic |
| `[7]` Help / Usage | Bantuan |
| `[8]` API Status | Cek koneksi ke Xray API |
| `[0]` Keluar | Exit |

---

## Contoh Alur Kerja

### Skenario 1: Tambah User Baru

```bash
# 1. Generate UUID
UUID=$(xray uuid)
echo "UUID: $UUID"

# 2. Tambah user ke semua inbound
xray-cli add all user-baru@x.com $UUID

# 3. Cek hasilnya
xray-cli get user-baru@x.com
```

### Skenario 2: Audit Semua User VMess

```bash
xray-cli list vmess
xray-cli check vmess login all
xray-cli check vmess quota all
```

### Skenario 3: Hapus User yang Tidak Aktif

```bash
# Cek user yang tidak online
xray-cli check all login all

# Hapus user
xray-cli delete user-lama@x.com
```

### Skenario 4: Otomasi dari Bash

```bash
#!/usr/bin/env bash
USERS=(
    "alice@x.com:uuid-alice-1234"
    "bob@x.com:uuid-bob-5678"
)

for entry in "${USERS[@]}"; do
    USER="${entry%%:*}"
    UUID="${entry##*:}"
    if xray-cli add vmess "$USER" "$UUID" > /dev/null 2>&1; then
        echo "✅ $USER berhasil ditambahkan"
    else
        echo "❌ $USER gagal ditambahkan"
    fi
done
```

### Skenario 5: Backup Daftar User

```bash
xray-cli list > backup-users-$(date +%Y%m%d).json
xray-cli list vmess > backup-vmess-$(date +%Y%m%d).json
```

---

## Troubleshooting

### Error: `gagal koneksi ke API Xray`

**Penyebab:** Xray tidak berjalan, atau API di port yang berbeda.

**Solusi:**
```bash
systemctl status xray
ss -tlnp | grep 1001
grpcurl -plaintext 127.0.0.1:1001 list
```

### Error: `user 'xxx' sudah ada di semua target yang diminta`

**Penyebab:** User sudah terdaftar di inbound yang dituju.

**Solusi:**
- Gunakan nama user yang berbeda
- Atau hapus dulu: `xray-cli delete user-lama@x.com`

### Error: `server does not support the reflection API`

**Penyebab:** `ReflectionService` tidak aktif.

**Solusi:** Tambahkan `"ReflectionService"` ke `api.services`, lalu restart Xray.

### User Hilang Setelah Restart

**Penyebab:** User yang ditambah via API tidak tertulis ke `config.json`.

**Solusi:** Untuk persistensi, tambahkan user ke `clients` di `config.json`, lalu restart Xray.

### Output JSON Menampilkan `\u003c` atau `\u003e`

**Penyebab:** Version lama yang belum pakai `SetEscapeHTML(false)`.

**Solusi:** Rebuild dengan versi terbaru `main.go`.

### Command `xray-cli` Tidak Ditemukan

**Solusi:**
```bash
sudo cp xray-cli /usr/local/bin/
sudo chmod +x /usr/local/bin/xray-cli
echo 'export PATH=$PATH:/usr/local/bin' >> ~/.bashrc
source ~/.bashrc
```

---

## Ringkasan Perintah

| Perintah | Contoh | Fungsi |
|---|---|---|
| `add` | `xray-cli add vmess user@x.com <uuid>` | Tambah user |
| `list` | `xray-cli list vmess` | List user |
| `get` | `xray-cli get user@x.com` | Detail 1 user |
| `delete` | `xray-cli delete user@x.com` | Hapus user |
| `check login` | `xray-cli check vmess login all` | Cek IP online |
| `check quota` | `xray-cli check all quota all` | Cek traffic |
| Help | `xray-cli` | Tampilkan help |

---

## Dukungan

Untuk pertanyaan, bug report, atau fitur request:

- **Telegram:** [t.me/SukaPediaCS](https://t.me/SukaPediaCS)
- **WhatsApp:** [wa.me/6285935195701](https://wa.me/6285935195701)

---

**© XDTunnel** — Dokumentasi ini dapat berubah sewaktu-waktu mengikuti update binary.
