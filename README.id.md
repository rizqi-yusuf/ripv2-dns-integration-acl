<div align="center">

# 🛡️ Jaringan Enterprise Aman: Implementasi RIPv2, ACL, dan DNS

**Simulasi infrastruktur jaringan tingkat enterprise yang mendemonstrasikan routing dinamis, kebijakan penyaringan trafik, dan resolusi domain lokal menggunakan Cisco Packet Tracer.**

![Cisco](https://img.shields.io/badge/Cisco-Packet%20Tracer-1BA0D7?logo=cisco&logoColor=white)
![Routing](https://img.shields.io/badge/Routing-RIPv2-4CAF50)
![Security](https://img.shields.io/badge/Security-ACL-D32F2F)
![DNS](https://img.shields.io/badge/Services-DNS-FFC107)

🇬🇧 [Read in English](README.md)

</div>

---

Proyek ini menampilkan keterampilan *network engineering* secara praktis, khususnya berfokus pada komunikasi antar-situs yang aman, isolasi *endpoint*, dan manajemen server terpusat.

## 🏗️ Arsitektur & Topologi Jaringan

<img width="1128" height="357" alt="Network Topology" src="https://github.com/user-attachments/assets/9977e6d6-8f11-4ed0-99c8-06e18ee96d64" />

Infrastruktur dibagi menjadi tiga area utama yang terhubung melalui *backbone link* berkecepatan tinggi.

### Tabel Pengalamatan IP
| Lokasi | Jaringan / Subnet | Default Gateway | Tujuan / Peran |
| :--- | :--- | :--- | :--- |
| **Site A** | `192.168.33.0/24` | `192.168.33.1` | Jaringan Klien (Karyawan) |
| **Site A** | `192.166.33.0/24` | `192.166.33.1` | Data Center / Server Farm |
| **Site B** | `192.167.33.0/24` | `192.167.33.1` | Jaringan Klien (Cabang Remote) |
| **Core** | `199.166.33.0/30` | N/A | Point-to-Point Backbone Link |

## 📁 Struktur Repo

```text
.
├── Enterprise-Network-Simulation.pkt   # File simulasi Cisco Packet Tracer
├── .gitignore                          # Aturan ignore (backup PT & OS files)
├── LICENSE                             # Lisensi MIT
├── README.md                           # Dokumentasi (Bahasa Inggris)
└── README.id.md                        # Dokumentasi (Bahasa Indonesia)
```

## 🚀 Cara Menjalankan Lab Ini

Untuk menguji konfigurasi dan mengeksplorasi topologi:
1. Pastikan Anda telah menginstal **Cisco Packet Tracer** (Direkomendasikan versi 8.0 ke atas).
2. Unduh file `.pkt` yang ada di repositori ini.
3. Buka file tersebut dan tunggu hingga port STP (Spanning Tree Protocol) konvergen (berwarna hijau).
4. Ikuti **Prosedur Validasi & Pengujian** di bawah ini untuk memverifikasi konektivitas dan kebijakan keamanan.

## ⚙ Teknologi Inti & Layanan

1. **Dynamic Routing (RIPv2):** Diimplementasikan di semua router distribusi untuk membangun konektivitas *end-to-end* secara dinamis. *Auto-summary* dinonaktifkan untuk mendukung VLSM (*Variable Length Subnet Masking*).
2. **Access Control List (ACL):** Kebijakan keamanan standar yang diterapkan pada level antarmuka router untuk memblokir host yang tidak sah agar tidak dapat mengakses *resource* server yang krusial.
3. **Layanan DNS & Web:** Sebuah server terpusat (`192.166.33.2`) dikonfigurasi untuk menjalankan portal web intranet dan memecahkan nama domain lokal (`qyv.com`), memungkinkan klien mengakses layanan menggunakan URL standar.

## 📝 Sorotan Konfigurasi

### 1. Site A (Router 4) - Routing & Keamanan
Router ini menangani trafik lokal, terhubung ke *backbone*, dan menyaring trafik yang tidak sah (PC0: `192.168.33.2`) agar tidak masuk ke jaringan inti (*core*).

```text
! Aktifkan Routing RIPv2
Router(config)# router rip
Router(config-router)# version 2
Router(config-router)# network 192.168.33.0
Router(config-router)# network 192.166.33.0
Router(config-router)# network 199.166.33.0
Router(config-router)# no auto-summary

! Konfigurasi Standard ACL untuk Mengisolasi PC0
Router(config)# access-list 1 deny host 192.168.33.2
Router(config)# access-list 1 permit any

! Terapkan ACL Inbound pada Interface Gateway Klien
Router(config)# interface gigabitEthernet 0/1
Router(config-if)# ip access-group 1 in
```

### 2. Site B (Router 5) - Routing
```text
! Aktifkan Routing RIPv2
Router(config)# router rip
Router(config-router)# version 2
Router(config-router)# network 192.167.33.0
Router(config-router)# network 199.166.33.0
Router(config-router)# no auto-summary
```

### 3. Analisis Command Line
*   **`no auto-summary`**: Mencegah router merangkum rute (summary) ke batasan *classful*, memastikan tabel routing tetap akurat saat menggunakan subnet mask kustom.
*   **`access-list 1 permit any`**: Secara eksplisit mengizinkan semua lalu lintas lainnya. Tanpa perintah ini, aturan *implicit deny* dari Cisco akan membuang semua paket yang melewati antarmuka.
*   **`ip access-group 1 in`**: Menerapkan aturan penyaringan secara langsung saat paket memasuki antarmuka (*inbound*), menghemat siklus CPU router dibandingkan dengan penyaringan *outbound*.

## 🧪 Prosedur Validasi & Pengujian

Verifikasi berikut dilakukan untuk memastikan keandalan jaringan dan kepatuhan terhadap keamanan:

### ✅ Skenario 1: Akses Sah & Resolusi DNS
Klien yang sah berhasil melakukan kueri ke server DNS dan mengakses portal web intranet (`qyv.com`).

<img width="1166" height="64" alt="DNS Resolution" src="https://github.com/user-attachments/assets/60f0d4ae-e974-4240-aa4f-882192cff4dd" />

**Pengujian Web Browser:**
<img width="516" height="211" alt="Web Browser Test" src="https://github.com/user-attachments/assets/3e108d49-88d8-4f89-a77e-2c91497aff89" />

### ❌ Skenario 2: Kepatuhan Keamanan (Blokir ACL)
*Endpoint* yang dibatasi (**PC0**) mencoba mengakses server web. 
*   **Hasil:** Router berhasil membuang lalu lintas tersebut (*dropped*). Browser mengembalikan pesan kesalahan **"Host Name Unresolved"**, mengonfirmasi bahwa ACL *inbound* berfungsi sebagaimana mestinya.

## 💡 Pelajaran & Troubleshooting

Selama implementasi proyek ini, saya menemukan dan menyelesaikan beberapa tantangan arsitektur:
* **Strategi Penempatan ACL:** Awalnya, menempatkan ACL dekat dengan tujuan (Server) menyebabkan lalu lintas *backbone* yang tidak perlu. Dengan menerapkan *Standard ACL* langsung pada antarmuka sumber (`GigabitEthernet 0/1` di Router 4), efisiensi *bandwidth* di seluruh link *Core* meningkat secara signifikan.
* **Masalah Resolusi DNS:** Kueri DNS sempat gagal sampai semua PC klien diperbarui dengan benar untuk menunjuk ke `192.166.33.2` sebagai server DNS utama mereka, memperkuat pentingnya manajemen konfigurasi DHCP/IP Statis yang tepat.

## 📄 Lisensi
Dirilis di bawah lisensi [MIT](LICENSE).

## 👤 Penulis
* **Nama:** Rizqi Maulana Yusuf
* **GitHub:** [@rizqi-yusuf](https://github.com/rizqi-yusuf)
