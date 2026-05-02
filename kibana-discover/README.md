# 🔎 Kibana Discover — Eksplorasi & Analisis Log di ELK Stack

> **Seri Lab:** Blue Team SOC Lab  
> **Platform:** [TryHackMe](https://tryhackme.com)  
> **YouTube:** [▶️ Tonton Tutorial di YouTube](https://youtu.be/LWhlB3OSENY?si=7xuVeJBrvi6iFqwj)  
> **Kategori:** `SIEM Analysis, Log Analysis, ELK Stack`

---

## 📋 Deskripsi

Lab ini memperkenalkan **Kibana Discover Tab** — antarmuka utama yang digunakan SOC Analyst untuk mengeksplorasi, mencari, dan menganalisis log yang masuk ke ELK Stack. Melalui lab ini kita menganalisis **VPN connection logs** untuk menemukan anomali, IP mencurigakan, dan spike traffic yang tidak normal.

---

## 🎬 Video Tutorial

[![Kibana Discover - Eksplorasi Log di ELK Stack](https://img.youtube.com/vi/LWhlB3OSENY/maxresdefault.jpg)](https://youtu.be/LWhlB3OSENY?si=7xuVeJBrvi6iFqwj)

> 📺 **[Tonton di YouTube → Kibana Discover: Eksplorasi & Analisis Log di ELK Stack](https://youtu.be/LWhlB3OSENY?si=7xuVeJBrvi6iFqwj)**

---

## 🧱 Komponen ELK Stack

Sebelum masuk ke Kibana, penting untuk memahami tiga komponen utama ELK:

| Komponen | Fungsi |
|----------|--------|
| **Elasticsearch** | Database — menyimpan dan mengindeks semua log |
| **Logstash** | Pipeline — memproses dan menormalkan log sebelum disimpan |
| **Kibana** | Frontend — antarmuka visual untuk eksplorasi dan analisis log |

> 💡 **Kibana** adalah komponen yang digunakan SOC Analyst sehari-hari untuk berinteraksi dengan data log di ELK Stack.

---

## 🖥️ Mengenal Kibana Discover Tab

**Discover Tab** adalah tempat SOC Analyst menghabiskan sebagian besar waktunya. Di sinilah log yang sudah diingest ditampilkan, dapat dicari, difilter, dan dianalisis.

### Elemen-Elemen Utama

```
┌─────────────────────────────────────────────────────────────┐
│  TOP BAR   [Save] [Open] [Share] [Inspect]                  │
├──────────┬──────────────────────────────────────────────────┤
│          │  Search Bar  [+ Add Filter]   Time Filter ▼      │
│  Fields  ├──────────────────────────────────────────────────┤
│  Pane    │                                                   │
│          │  Timeline / Event Count Chart                     │
│  field1  ├──────────────────────────────────────────────────┤
│  field2  │  Log 1: { timestamp, IP, user, event... }        │
│  field3  │  Log 2: { timestamp, IP, user, event... }        │
│  ...     │  Log 3: { timestamp, IP, user, event... }        │
│          │  ...                                              │
└──────────┴──────────────────────────────────────────────────┘
```

### Penjelasan Setiap Elemen

| Elemen | Fungsi |
|--------|--------|
| **Logs** | Setiap baris adalah satu log berisi informasi event beserta field dan nilainya |
| **Fields Pane** | Panel kiri — daftar field yang diparsing dari log, klik untuk filter |
| **Index Pattern** | Menentukan sumber data mana yang ditampilkan (contoh: `vpn_connections`) |
| **Search Bar** | Tempat menulis query dan menambahkan filter untuk mempersempit hasil |
| **Time Filter** | Filter log berdasarkan rentang waktu tertentu |
| **Timeline** | Grafik jumlah event dari waktu ke waktu — berguna untuk deteksi spike |
| **Top Bar** | Menyimpan, membuka, dan berbagi search yang sudah dibuat |
| **Add Filter** | Menambahkan filter ke field tertentu tanpa harus mengetik query manual |

---

## ⚙️ Konsep Penting — Index Pattern

**Index Pattern** memberitahu Kibana data mana dari Elasticsearch yang ingin dieksplorasi. Setiap sumber log memiliki struktur yang berbeda, sehingga saat log diingest ke Elasticsearch, log tersebut dinormalisasi ke field-field yang sesuai.

```
Log Masuk (raw)
      │
      ▼
Logstash (normalisasi & parsing)
      │
      ▼
Elasticsearch (disimpan di index)
      │
      ▼
Kibana Index Pattern → pilih index → Discover Tab
```

> 💡 Di lab ini kita menggunakan index pattern **`vpn_connections`** yang berisi log koneksi VPN.

---

## ⚙️ Cara Menggunakan Fields Pane

Panel kiri menampilkan semua field yang tersedia dalam log. Klik pada field mana saja untuk melihat **5 nilai teratas** beserta persentase kemunculannya.

```
Klik field "SourceIP"
→ Tampil: 
   238.163.231.224  │████████  15%
   107.14.1.247     │█████     8%
   172.201.60.191   │████      6%
   ...

Klik [+] → Tambah filter HANYA tampilkan log dengan nilai ini
Klik [-] → Tambah filter KECUALIKAN log dengan nilai ini
```

---

## ⚙️ Cara Membuat Tabel dari Log

Secara default log ditampilkan dalam bentuk raw. Kita bisa membuat tampilan tabel yang lebih rapi:

```
1. Klik salah satu log untuk expand
2. Klik icon [+] di samping field yang ingin ditampilkan
   (contoh: IP, UserName, Source_Country)
3. Field tersebut akan menjadi kolom di tabel
4. Ulangi untuk setiap field yang dibutuhkan
5. Klik Save di Top Bar untuk menyimpan format tabel
```

> 💡 Menyimpan format tabel memastikan tampilan yang sama setiap kali login ke Kibana — sangat berguna untuk monitoring rutin.

---

## 🧪 Investigasi Lab — VPN Connection Logs

### Setup Awal

```
1. Buka Kibana di virtual machine TryHackMe
2. Pilih Index Pattern: vpn_connections
3. Set Time Filter: 31 Desember 2021 — 2 Februari 2022
```

---

### Pertanyaan 1 — Total Log dalam Rentang Waktu

**Filter:** 31 Desember 2021 — 2 Februari 2022  
**Cara:** Set Time Filter di pojok kanan atas

```
Jawaban: 2861 hits
```

---

### Pertanyaan 2 — IP dengan Koneksi Terbanyak

**Cara:** Klik field `SourceIP` di Fields Pane → lihat nilai teratas

```
Jawaban: 238.163.231.224
```

---

### Pertanyaan 3 — User dengan Total Traffic Tertinggi

**Cara:** Klik field `UserName` di Fields Pane → lihat nilai teratas

```
Jawaban: James
```

---

### Pertanyaan 4 — Source IP Terbanyak untuk User Emanda

**Cara:**
```
1. Klik field UserName di Fields Pane
2. Klik [+] di samping nilai "Emanda"
3. Filter aktif → lihat field SourceIP yang muncul paling banyak
```

```
Jawaban: 107.14.1.247
```

---

### Pertanyaan 5 — IP Penyebab Spike pada 11 Januari

**Cara:**
```
1. Lihat Timeline Chart — cari spike pada tanggal 11 Januari 2022
2. Klik bar spike tersebut untuk zoom ke rentang waktu tersebut
3. Analisis field SourceIP pada log yang muncul
```

> 💡 Spike di timeline adalah indikator awal aktivitas tidak normal. SOC Analyst harus selalu memperhatikan lonjakan yang tiba-tiba di grafik ini.

```
Jawaban: 172.201.60.191
```

---

### Pertanyaan 6 — Koneksi dari IP 238.163.231.224 di Luar New York

**Cara:**
```
1. Tambahkan filter: SourceIP = 238.163.231.224
2. Tambahkan filter: Source_State ≠ New York (gunakan "is not")
3. Hitung total hits yang muncul
```

```
Jawaban: 48 koneksi
```

---

### Pertanyaan 7 — Buat & Simpan Tabel

Buat tabel dengan field berikut:

| Field | Keterangan |
|-------|------------|
| `IP` | IP Address sumber koneksi |
| `UserName` | Nama user yang melakukan koneksi |
| `Source_Country` | Negara asal koneksi |

**Cara:**
```
1. Expand salah satu log
2. Klik [+] di samping field IP, UserName, Source_Country
3. Ketiga field akan menjadi kolom tabel
4. Klik Save di Top Bar
5. Beri nama: "VPN Connection Table"
6. Save
```

---

## 📊 Ringkasan Temuan Investigasi

| Pertanyaan | Temuan |
|------------|--------|
| Total log (31 Des 2021 — 2 Feb 2022) | **2.861 hits** |
| IP dengan koneksi terbanyak | **238.163.231.224** |
| User dengan traffic tertinggi | **James** |
| Source IP terbanyak untuk Emanda | **107.14.1.247** |
| IP penyebab spike 11 Januari | **172.201.60.191** |
| Koneksi IP 238.163.231.224 di luar New York | **48 koneksi** |

---

## 💡 Tips SOC Analyst — Kibana Discover

| Tips | Penjelasan |
|------|------------|
| **Selalu perhatikan Timeline** | Spike mendadak hampir selalu mengindikasikan aktivitas tidak normal |
| **Gunakan Fields Pane** | Lebih cepat dari mengetik query manual untuk filter nilai tertentu |
| **Simpan format tabel** | Konsistensi tampilan membantu analisis yang lebih efisien |
| **Kombinasikan filter** | Gunakan beberapa filter sekaligus untuk mempersempit investigasi |
| **Perhatikan Top Values** | Nilai yang jauh di atas rata-rata adalah kandidat investigasi |

---

## 📚 Referensi

- [Kibana Discover Documentation](https://www.elastic.co/guide/en/kibana/current/discover.html)
- [ELK Stack Overview](https://www.elastic.co/what-is/elk-stack)
- [Kibana KQL Query Language](https://www.elastic.co/guide/en/kibana/current/kuery-query.html)
- [TryHackMe — ELK Room](https://tryhackme.com)

---

*📺 Ikuti tutorialnya di [YouTube](https://youtu.be/LWhlB3OSENY?si=7xuVeJBrvi6iFqwj) | ⭐ Star repo ini jika membantu!*
