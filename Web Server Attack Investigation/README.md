# 🎯 Web Server Attack Investigation — Slingshot

> **Seri Lab:** Blue Team SOC Lab  
> **Platform:** [TryHackMe — Room: Slingshot](https://tryhackme.com/room/slingshot)  
> **YouTube:** [▶️ Tonton Tutorial di YouTube](https://youtu.be/CTpS5tEP4fo?si=trFyDxxBDtluVlJO)  
> **Kategori:** `Incident Response, Log Analysis, Web Attack Investigation`

---

## 📋 Deskripsi

Lab ini adalah investigasi insiden nyata terhadap **web server e-commerce milik Slingway Inc.** yang diduga telah dikompromasi. Menggunakan **Elastic Stack (Kibana)**, kita menganalisis Apache access logs untuk merekonstruksi setiap langkah attacker — mulai dari reconnaissance, eksploitasi, hingga exfiltrasi data.

---

## 🎬 Video Tutorial

[![Web Server Attack Investigation - Slingshot](https://img.youtube.com/vi/CTpS5tEP4fo/maxresdefault.jpg)](https://youtu.be/CTpS5tEP4fo?si=trFyDxxBDtluVlJO)

> 📺 **[Tonton di YouTube → Web Server Attack Investigation: Slingshot](https://youtu.be/CTpS5tEP4fo?si=trFyDxxBDtluVlJO)**

---

## 🏢 Latar Belakang Kasus

**Slingway Inc.**, sebuah perusahaan mainan terkemuka, mendeteksi aktivitas mencurigakan pada web server e-commerce-nya beserta potensi modifikasi tidak sah pada database. Tim IT mencatat aktivitas mencurigakan mulai pada **26 Juli 2023**.

**Pertanyaan utama investigasi:**
- Teknik reconnaissance dan enumeration apa yang digunakan attacker?
- Kerentanan apa yang dieksploitasi di web server?
- Bagaimana attacker mendapatkan akses administratif?
- Data sensitif apa yang diakses atau dieksfiltrasi?

---

## ⚙️ Setup Lab

### Akses Kibana

```
URL      : https://LAB_WEB_URL.p.thmlabs.com/
Username : elastic
Password : raCK0W**BLlW66oNlKAk
```

### Konfigurasi Awal

```
1. Buka Kibana → Discover
2. Pilih Data View  : apache_logs
3. Set Time Filter  : Jul 26, 2023 @ 00:00:00.000 → now
4. Mulai investigasi
```

---

## 🔍 Metodologi Investigasi

Investigasi ini mengikuti urutan **attack chain** — dari awal attacker masuk hingga eksfiltrasi data:

```
[1] Reconnaissance     → Scanner & enumeration tools
        │
        ▼
[2] Enumeration        → Directory bruteforce, 404 responses
        │
        ▼
[3] Login Page Found   → Halaman admin ditemukan
        │
        ▼
[4] Credential Brute Force → Username & password admin
        │
        ▼
[5] File Upload        → Web shell diupload
        │
        ▼
[6] Remote Code Exec   → Command dijalankan via web shell
        │
        ▼
[7] Local File Inclusion → Credential database dicuri
        │
        ▼
[8] Database Exfiltration → Export database via phpMyAdmin
        │
        ▼
[9] Data Manipulation  → Insert data berbahaya ke database
```

---

## 🧪 Investigasi — Query & Temuan

### Persiapan Query Dasar

Sebelum investigasi, kenali data yang tersedia:

```kql
# Lihat semua log
*

# Filter ke status tertentu
response : 404

# Filter berdasarkan IP
clientip : <IP_ATTACKER>
```

---

### Pertanyaan 1 — IP Address Attacker

**Pendekatan:** Cari IP dengan jumlah request tertinggi — terutama yang menghasilkan banyak error 404 (tanda enumeration).

```kql
# Lihat top IP berdasarkan jumlah request
# Gunakan Fields Pane → klik field "clientip"
```

```
Jawaban: (temukan saat investigasi di lab)
```

---

### Pertanyaan 2 — Scanner Pertama yang Digunakan

**Pendekatan:** Filter log dari IP attacker, urutkan dari yang paling awal, lihat field `agent` (User-Agent).

```kql
clientip : <IP_ATTACKER>
```

Urutkan berdasarkan `@timestamp` ascending → lihat User-Agent pertama.

```
Jawaban: (temukan dari field agent/User-Agent di log pertama)
```

> 💡 Scanner umum yang sering muncul: Nmap, Nikto, WhatWeb, curl dengan header khas.

---

### Pertanyaan 3 — User-Agent Tool Enumerasi Direktori

**Pendekatan:** Cari log dengan banyak request ke path yang berbeda-beda dalam waktu singkat — ciri khas directory bruteforce.

```kql
clientip : <IP_ATTACKER> AND response : 404
```

Perhatikan field `agent` — tool enumerasi direktori yang umum: **Gobuster**, **DirBuster**, **ffuf**, **dirsearch**.

```
Jawaban: (temukan dari field agent pada log 404)
```

---

### Pertanyaan 4 — Total Respons 404 saat Enumerasi

**Pendekatan:** Hitung semua response 404 dari IP attacker.

```kql
clientip : <IP_ATTACKER> AND response : 404
```

Lihat jumlah hits di pojok kiri atas Kibana.

```
Jawaban: (temukan dari total hits query di atas)
```

---

### Pertanyaan 5 — Flag di Direktori yang Ditemukan

**Pendekatan:** Cari response 200 dari IP attacker saat enumerasi — ini adalah direktori yang berhasil ditemukan.

```kql
clientip : <IP_ATTACKER> AND response : 200
```

Periksa path yang berhasil diakses — cari yang mengandung flag.

```
Jawaban: (temukan dari request yang mengembalikan response 200)
```

---

### Pertanyaan 6 — Halaman Login yang Ditemukan

**Pendekatan:** Cari akses ke path yang mengandung kata kunci login atau admin dengan response 200.

```kql
clientip : <IP_ATTACKER> AND response : 200 AND request : *login*
```

atau

```kql
clientip : <IP_ATTACKER> AND response : 200 AND request : *admin*
```

```
Jawaban: (temukan dari field request pada response 200)
```

---

### Pertanyaan 7 — User-Agent Tool Brute Force Admin Panel

**Pendekatan:** Setelah halaman login ditemukan, cari log dengan banyak POST request ke halaman tersebut — ciri khas brute force.

```kql
clientip : <IP_ATTACKER> AND request : *<LOGIN_PAGE>* AND verb : POST
```

Perhatikan field `agent` — tool brute force yang umum: **Hydra**, **Medusa**, **Burp Suite Intruder**.

```
Jawaban: (temukan dari field agent pada POST requests)
```

---

### Pertanyaan 8 — Kombinasi Username:Password yang Berhasil

**Pendekatan:** Setelah banyak POST request gagal (response 401/403), cari POST request yang berhasil (response 200/302).

```kql
clientip : <IP_ATTACKER> AND verb : POST AND (response : 200 OR response : 302)
```

Inspect log tersebut untuk menemukan credential yang digunakan.

```
Jawaban: (temukan dari log POST yang berhasil)
```

---

### Pertanyaan 9 — Flag di File yang Diupload ke Web Shell

**Pendekatan:** Cari akses ke `/admin/upload.php` dari IP attacker.

```kql
clientip : <IP_ATTACKER> AND request : *upload.php*
```

```
Jawaban: (temukan dari log upload request)
```

---

### Pertanyaan 10 — Perintah Pertama via Web Shell

**Pendekatan:** Setelah web shell diupload, attacker mengaksesnya dengan parameter command. Cari GET request ke file web shell yang mengandung parameter.

```kql
clientip : <IP_ATTACKER> AND request : *shell* AND verb : GET
```

atau

```kql
clientip : <IP_ATTACKER> AND request : *cmd*
```

Urutkan berdasarkan timestamp ascending untuk menemukan command pertama.

```
Jawaban: (temukan dari parameter di URL request web shell)
```

---

### Pertanyaan 11 — File yang Diakses via LFI untuk Credential Database

**Pendekatan:** LFI (Local File Inclusion) biasanya terlihat di URL sebagai path traversal (`../`) atau akses ke file sistem.

```kql
clientip : <IP_ATTACKER> AND request : *../*
```

atau

```kql
clientip : <IP_ATTACKER> AND request : *etc/passwd*
```

File database credential yang umum: `/etc/passwd`, `config.php`, `database.php`, `wp-config.php`.

```
Jawaban: (temukan dari field request yang mengandung path traversal)
```

---

### Pertanyaan 12 — Nama Database yang Diekspor

**Pendekatan:** Cari akses ke `/phpmyadmin` dari IP attacker.

```kql
clientip : <IP_ATTACKER> AND request : *phpmyadmin*
```

Perhatikan parameter di URL yang mengandung nama database (biasanya `db=` atau `database=`).

```
Jawaban: (temukan dari parameter URL di request phpmyadmin)
```

---

### Pertanyaan 13 — Flag yang Diinsert ke Database via import.php

**Pendekatan:** Cari akses ke `import.php` dari IP attacker.

```kql
clientip : <IP_ATTACKER> AND request : *import.php*
```

```
Jawaban: (temukan dari request ke import.php)
```

---

## 📊 Timeline Serangan (Attack Timeline)

Isi tabel ini dengan temuan investigasi:

| Waktu | Aktivitas | Tool/Method | Hasil |
|-------|-----------|-------------|-------|
| T+0 | Reconnaissance awal | Scanner 1 | Identifikasi web server |
| T+1 | Directory enumeration | Tool enum | Temukan halaman login + flag |
| T+2 | Brute force login | Tool BF | Berhasil masuk admin |
| T+3 | Upload web shell | `/admin/upload.php` | Shell aktif |
| T+4 | Remote code execution | Web shell | Command pertama dijalankan |
| T+5 | Local File Inclusion | Path traversal | Credential database dicuri |
| T+6 | Database exfiltration | phpMyAdmin | Database diekspor |
| T+7 | Data manipulation | import.php | Flag diinsert ke database |

---

## 📋 Indicators of Compromise (IoC)

| Tipe | Nilai | Keterangan |
|------|-------|------------|
| **IP Attacker** | *(dari investigasi)* | IP yang melakukan seluruh serangan |
| **Scanner** | *(dari investigasi)* | Tool reconnaissance pertama |
| **Enum Tool UA** | *(dari investigasi)* | User-Agent tool enumerasi direktori |
| **BF Tool UA** | *(dari investigasi)* | User-Agent tool brute force |
| **Credential** | *(dari investigasi)* | Username:password admin yang dicuri |
| **Web Shell Path** | `/admin/upload.php` | Lokasi upload web shell |
| **LFI Target** | *(dari investigasi)* | File yang diakses via LFI |
| **Database** | *(dari investigasi)* | Nama database yang diekspor |

---

## 🔍 Query Referensi Lengkap

```kql
# Semua aktivitas attacker
clientip : <IP_ATTACKER>

# Enumeration (404 responses)
clientip : <IP_ATTACKER> AND response : 404

# Akses berhasil (200)
clientip : <IP_ATTACKER> AND response : 200

# Brute force (POST requests)
clientip : <IP_ATTACKER> AND verb : POST

# Web shell activity
clientip : <IP_ATTACKER> AND request : *shell*

# LFI attempts
clientip : <IP_ATTACKER> AND request : *..*

# phpMyAdmin activity
clientip : <IP_ATTACKER> AND request : *phpmyadmin*
```

---

## 📌 Kesimpulan

| Fase | Teknik | Dampak |
|------|--------|--------|
| **Reconnaissance** | Web scanner, directory enum | Pemetaan struktur web server |
| **Initial Access** | Brute force admin login | Akses panel administrasi |
| **Execution** | Web shell upload + RCE | Kontrol penuh atas web server |
| **Credential Access** | Local File Inclusion | Database credential dicuri |
| **Exfiltration** | phpMyAdmin export | Data sensitif diekspor |
| **Impact** | Database manipulation | Integritas data terganggu |

---

## 📚 Referensi

- [TryHackMe — Slingshot Room](https://tryhackme.com/room/slingshot)
- [MITRE ATT&CK — Web Shell (T1505.003)](https://attack.mitre.org/techniques/T1505/003/)
- [MITRE ATT&CK — Brute Force (T1110)](https://attack.mitre.org/techniques/T1110/)
- [MITRE ATT&CK — LFI (T1059)](https://attack.mitre.org/techniques/T1059/)
- [Elastic KQL Documentation](https://www.elastic.co/guide/en/kibana/current/kuery-query.html)

---

*📺 Ikuti tutorialnya di [YouTube](https://youtu.be/CTpS5tEP4fo?si=trFyDxxBDtluVlJO) | ⭐ Star repo ini jika membantu!*
