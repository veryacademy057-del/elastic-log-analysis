# 🔍 KQL Basics — Kibana Query Language untuk Analisis Log

> **Seri Lab:** Blue Team SOC Lab  
> **Platform:** [TryHackMe](https://tryhackme.com)  
> **YouTube:** [▶️ Tonton Tutorial di YouTube](https://youtu.be/EH_6984Uzqs?si=7qXcq7EPh4MqP_nV)  
> **Kategori:** `SIEM Analysis, Log Analysis, ELK Stack`

---

## 📋 Deskripsi

Lab ini memperkenalkan **KQL (Kibana Query Language)** — bahasa query yang digunakan di Search Bar Kibana untuk mencari dan memfilter log di Elasticsearch. Menguasai KQL adalah kemampuan dasar yang wajib dimiliki setiap SOC Analyst yang bekerja dengan ELK Stack.

---

## 🎬 Video Tutorial

[![KQL Basics - Kibana Query Language](https://img.youtube.com/vi/EH_6984Uzqs/maxresdefault.jpg)](https://youtu.be/EH_6984Uzqs?si=7qXcq7EPh4MqP_nV)

> 📺 **[Tonton di YouTube → KQL Basics: Kibana Query Language untuk Analisis Log](https://youtu.be/EH_6984Uzqs?si=7qXcq7EPh4MqP_nV)**

---

## 🧠 Apa itu KQL?

**KQL (Kibana Query Language)** adalah bahasa query khusus yang digunakan di Search Bar Kibana untuk mencari log/dokumen yang sudah diingest ke Elasticsearch.

KQL mendukung dua metode pencarian:

| Metode | Penjelasan |
|--------|------------|
| **Free Text Search** | Cari berdasarkan teks — mencari di semua field |
| **Field-based Search** | Cari berdasarkan field spesifik dengan sintaks `Field: Value` |

---

## 📝 Metode 1 — Free Text Search

Free text search mencari teks di **seluruh field** dalam log — tidak perlu tahu field mana yang mengandung nilai tersebut.

### Pencarian Kata Penuh

```kql
United States
```

> 💡 KQL mencari **kata/term lengkap**. Pencarian `United` saja tidak akan mengembalikan hasil karena KQL tidak mencari sebagian kata.

---

### Wildcard `*`

Gunakan wildcard `*` untuk mencocokkan bagian dari kata:

```kql
United*
```

Ini akan mengembalikan semua log yang mengandung kata dimulai dengan `United` — seperti `United States`, `United Kingdom`, `United Nations`, dll.

| Query | Hasil |
|-------|-------|
| `United` | ❌ Tidak ada hasil — KQL butuh kata lengkap |
| `United*` | ✅ Semua log yang mengandung kata diawali "United" |
| `*States` | ✅ Semua log yang mengandung kata diakhiri "States" |

---

## 🔗 Operator Logika

KQL mendukung operator logika untuk membangun query yang lebih kompleks.

### AND Operator

Menampilkan log yang mengandung **semua** kondisi:

```kql
"United States" AND "Virginia"
```

> Mengembalikan log yang mengandung **keduanya** — `United States` DAN `Virginia`.

---

### OR Operator

Menampilkan log yang mengandung **salah satu** kondisi:

```kql
"United States" OR "England"
```

> Mengembalikan log yang mengandung `United States` **ATAU** `England`.

---

### NOT Operator

Mengecualikan kondisi tertentu dari hasil pencarian:

```kql
"United States" AND NOT ("Florida")
```

> Mengembalikan log dari `United States` tapi **mengecualikan** log dari `Florida`.

---

### Kombinasi Operator

Operator bisa dikombinasikan untuk query yang lebih spesifik:

```kql
"United States" AND ("Virginia" OR "Texas") AND NOT ("Florida")
```

> ✅ Log dari US, hanya state Virginia atau Texas, tapi bukan Florida.

---

## 🎯 Metode 2 — Field-based Search

Field-based search memungkinkan pencarian pada **field spesifik** menggunakan sintaks:

```
Field : Value
```

Kolom dipisahkan oleh tanda titik dua (`:`) antara nama field dan nilainya.

### Contoh Dasar

```kql
Source_ip : 238.163.231.224
```

```kql
UserName : Suleman
```

---

### Kombinasi Field-based Search

```kql
Source_ip : 238.163.231.224 AND UserName : Suleman
```

> Menampilkan semua log di mana `Source_ip` adalah `238.163.231.224` **DAN** `UserName` adalah `Suleman`.

---

### Field-based + Operator Logika

```kql
Source_Country : "United States" AND (UserName : James OR UserName : Albert)
```

> Log dari US yang dilakukan oleh user James **ATAU** Albert.

---

### Field-based + Filter Waktu

```kql
UserName : "Johny Brown"
```

Dikombinasikan dengan **Time Filter** di Kibana:
```
Set Time Filter: mulai 1 Januari 2022 → sekarang
```

> Berguna untuk investigasi aktivitas user setelah tanggal tertentu — misalnya akun yang seharusnya sudah dinonaktifkan.

---

## 📋 Referensi Cepat Sintaks KQL

| Kebutuhan | Sintaks | Contoh |
|-----------|---------|--------|
| Cari teks bebas | `term` | `administrator` |
| Cari frasa (lebih dari satu kata) | `"frasa lengkap"` | `"United States"` |
| Wildcard akhir | `term*` | `admin*` |
| Wildcard awal | `*term` | `*admin` |
| AND — semua kondisi harus terpenuhi | `A AND B` | `"James" AND "New York"` |
| OR — salah satu kondisi | `A OR B` | `"James" OR "Albert"` |
| NOT — kecualikan kondisi | `NOT (A)` | `NOT ("Florida")` |
| Field spesifik | `Field : Value` | `UserName : James` |
| Field + AND | `Field : A AND Field : B` | `UserName : James AND Source_Country : "United States"` |
| Grup kondisi | `(A OR B)` | `(UserName : James OR UserName : Albert)` |

---

## 🧪 Investigasi Lab — Praktik KQL

### Soal 1 — User dari United States (James atau Albert)

**Tujuan:** Temukan log di mana Source_Country adalah United States dan user adalah James atau Albert.

```kql
Source_Country : "United States" AND (UserName : James OR UserName : Albert)
```

```
Jawaban: (temukan saat menjalankan query di lab)
```

---

### Soal 2 — Aktivitas Johny Brown Setelah Pemutusan Kerja

**Konteks:** User `Johny Brown` diberhentikan pada **1 Januari 2022**. Investigasi apakah masih ada koneksi VPN setelah tanggal tersebut.

**Langkah:**
```
1. Ketik query di Search Bar:
   UserName : "Johny Brown"

2. Set Time Filter:
   From: 1 Januari 2022 00:00:00
   To  : sekarang (atau tanggal akhir data)

3. Hitung total hits yang muncul
```

```kql
UserName : "Johny Brown"
```

```
Jawaban: (temukan saat menjalankan query di lab)
```

> 💡 **Insight SOC:** Jika akun user yang sudah diberhentikan masih menunjukkan aktivitas login, ini adalah **indikasi serius** — bisa berarti credential dicuri, akun belum dinonaktifkan, atau insider threat. Kasus ini wajib dieskalasi.

---

## 💡 Tips Menggunakan KQL

| Tips | Penjelasan |
|------|------------|
| **Gunakan tanda kutip untuk frasa** | `"United States"` bukan `United States` — tanpa kutip dianggap dua term terpisah |
| **Manfaatkan autocomplete** | Klik Search Bar → Kibana menampilkan semua field yang tersedia |
| **Kombinasikan dengan Time Filter** | Query + rentang waktu = investigasi yang jauh lebih presisi |
| **Gunakan wildcard dengan hati-hati** | `admin*` bisa mengembalikan terlalu banyak hasil — persempit dengan field |
| **Simpan query penting** | Gunakan fitur Save di Top Bar untuk query yang sering dipakai |

---

## 📌 Kesimpulan

| Konsep | Poin Penting |
|--------|-------------|
| **Free Text Search** | Mencari di semua field — mudah tapi kurang presisi |
| **Field-based Search** | `Field : Value` — lebih presisi, standar untuk investigasi SOC |
| **Wildcard `*`** | Mencocokkan sebagian kata — berguna untuk variasi ejaan |
| **AND / OR / NOT** | Operator logika untuk membangun query yang kompleks |
| **Kombinasi Time Filter + KQL** | Standar investigasi SOC — sempitkan scope berdasarkan waktu dan kondisi |

---

## 📚 Referensi

- [Kibana Query Language (KQL) Documentation](https://www.elastic.co/guide/en/kibana/current/kuery-query.html)
- [Elasticsearch Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)
- [TryHackMe — ELK Room](https://tryhackme.com)

---

*📺 Ikuti tutorialnya di [YouTube](https://youtu.be/EH_6984Uzqs?si=7qXcq7EPh4MqP_nV) | ⭐ Star repo ini jika membantu!*
