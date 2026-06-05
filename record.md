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


## Logika Sistem
State 1: Recording Mode (Durasi: 1 Hari)
    Inisialisasi kartu SD (SD.begin()) dan bus I2S.
    Matikan total modul Wi-Fi menggunakan perintah WiFi.mode(WIFI_OFF); untuk menghemat daya dan mencegah interferensi internal chip.
    ESP32-C3 membaca data audio digital dari INMP441 melalui DMA (Direct Memory Access) buffer dan langsung menulisnya ke SD Card secara berkala.

State 2: Upload Mode (Durasi: Sampai Selesai)
    Matikan fungsi pembacaan I2S (i2s_driver_uninstall) dan tutup akses file perekaman terakhir (file.close()).
    Aktifkan modul Wi-Fi, lakukan koneksi ke Access Point, dan inisialisasi protokol unggah (HTTP/MQTT/FTP).
    Buka file dari SD Card dalam mode read, lalu lakukan streaming data menuju server hingga selesai.
    Setelah sukses, hapus atau tandai file lama, matikan Wi-Fi, dan kembali ke State 1.
