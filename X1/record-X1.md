Record Project

## Komponen:
    • 2 | ESP32 C3 Super Mini
    • 2 | INMP441
    • MicroSD adapter 3v
    • Sandisk 64 GB
    • 74HC125
    • 10k ohm
    • Jumper Ferrite Bead Filter, low pass filter
    • NEC uPC2933HF
    • MP1584
    • KABEL GRAY ABU 1M FLAT PITA RIBON DUPONT IDC
    • Nippon Chemi-con 10uF 16V


## Sistem



## Wiring Diagram Menggunakan Dua IC 74HC125

### 1. Jalur Kontral Utama (Inter-ESP)
Pilih satu pin pada ESP-A sebagai Master Saklar, misalnya **GPIO 10**.
- Hubungkan ESP-A **GPIO 10** ke Pin **(Pin 1, 4, 10, 13) pada IC-1** (Buffer ESP-A).
- Hubungkan ESP-A **GPIO 10** melewati gerbang NOT (atau logika terbalik di software ESP-B) ke Pin **pada IC-2** (Buffer ESP-B).
- *Cara software yang lebih mudah:* Hubungkan **GPIO 10 (ESP-A)** langsung ke **GPIO 10 (ESP-B)** sebagai pin interupsi/jabat tangan (*handshaking*). Masing-masing ESP mengontrol pin IC-nya sendiri secara bergantian.

### 2. Wiring IC-1 (Sisi Perekam - ESP-A ke SD Card)
- **Input Buffer (A1, A2, A3):** Hubungkan ke pin SPI ESP-A:
    - Pin 2 (1A) → ESP-A **GPIO 6 (MOSI)**
    - Pin 2 (1A) → ESP-A **GPIO 4 (SCK)**
    - Pin 2 (1A) → Hubungkan ke **MOSI SD Card**
    - Pin 2 (1A) → Hubungkan ke **SCK SD Card**
- **Jalur MISO (Pembalikan Arah):** Karena MISO mengalir dari SD Card ke ESP:
    - Pin 2 (1A) → Hubungkan ke **MISO SD Card**
    - Pin 2 (1A) → ESP-A **GPIO 5 (MISO)**
- **Jalur CS (Chip Select):**
     - Pin 2 (1A) → ESP-A **GPIO 1**
     - Pin 2 (1A) → Hubungkan ke **CS SD Card**
- **Pin Kontrol:** Pin 1, 4, 10    - Pin 2 (1A) → Hubungkan ke **GPIO 0 (ESP-A)**.

### 3. Wiring IC-2 (Sisi Pengunggah - ESP-B ke SD Card)
Strukturnya persis sama dengan IC-1, tetapi input dan outputnya dihubungkan ke ESP-B:
    - Pin 2 (1A) → ESP-B **GPIO 6 (MOSI)**
    - Pin 2 (1A) → ESP-B **GPIO 4 (SCK)**
    - Pin 2 (1A) → Hubungkan paralel ke **MOSI SD Card** yang sama
    - Pin 2 (1A) → Hubungkan paralel ke **SCK SD Card** yang sama
    - Pin 2 (1A) → Hubungkan paralel ke **MISO SD Card** yang sama
    - Pin 2 (1A) → ESP-B **GPIO 5 (MISO)**
    - Pin 2 (1A) → ESP-B **GPIO 1**
    - Pin 2 (1A) → Hubungkan paralel ke **CS SD Card** yang sama
- **Pin Kontrol:** Pin 1, 4,     - Pin 2 (1A) → Hubungkan ke **GPIO 0 (ESP-B)**.

## Logika Urutan Kerja (Flowchart Sistem)
1. **Kondisi Awal (Merekam):**
    - ESP-A mengatur Pin GPIO 0 ke `LOW` (IC-1 Aktif).
    - ESP-B mengatur Pin GPIO 0 ke `HIGH` (IC-2 Isolasi/Hi-Z).
    - ESP-A merekam suara dari INMP441 dan menulis ke SD Card dengan aman.
2. **Pergantian Shift (Waktu Upload):**
    - ESP-A selesai merekam file hari itu, lalu menutup akses SD (`SD.end()`).
    - ESP-A mengubah Pin GPIO 0 ke `HIGH` (IC-1 Terisolasi).
    - ESP-A mengirim sinyal *trigger* ke ESP-B lewat kabel interkoneksi (misal GPIO 10 dibuat `HIGH`).
    - ESP-B menerima sinyal, lalu mengubah Pin GPIO 0 miliknya ke `LOW` (IC-2 Aktif).
    - ESP-B melakukan inisialisasi SD (`SD.begin()`), membaca file, dan mengunggahnya via Wi-Fi.
3. **Kembali Merekam:**
    - Setelah upload selesai, ESP-B mematikan jalur SPI, mengubah kembali IC-2 ke posisi `HIGH` (Isolasi), dan memberi tahu ESP-A untuk mulai merekam siklus baru.
