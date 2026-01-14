---
icon: arrows-up-to-line
---

# Over the Air Updates (OTA)

Mekanisme pembaruan OTA memungkinkan perangkat memperbarui dirinya sendiri berdasarkan data yang diterima saat firmware normal berjalan melalui perantara nirkabel seperti WiFi maupun Bluetooth.

Sebelum membahas upgrade firmware, satu topik penting yang perlu dibahas adalah partisi flash.

## Flash Partition

Framework ESP-IDF membagi flash menjadi beberapa partisi logis untuk menyimpan berbagai komponen. Cara umum untuk melakukannya ditunjukkan pada gambar.

<figure><img src="../.gitbook/assets/flash_partitions_intro.png" alt=""><figcaption></figcaption></figure>

Seperti yang terlihat pada gambar di atas, partisi fixed hanya sampai alamat 0x9000. Bootloader adalah bagian pertama dari firmware yang dijalankan setelah Embedded System dinyalakan/diatur ulang. Tujuan utama Bootloader adalah untuk menginisialisasi Embedded System dan memberikan kontrol kepada Aplikasi/RTOS. Partition table kemudian menyimpan bagaimana sisa flash akan diinterpretasikan. Biasanya instalasi akan memiliki setidaknya 1 partisi NVS dan 1 partisi firmware.

## OTA Mechanism

Untuk upgrade firmware, skema partisi aktif-pasif digunakan. Dua partisi flash dicadangkan untuk komponen 'firmware', seperti yang ditunjukkan pada gambar. Partisi Data OTA mengingat partisi mana yang aktif.

<figure><img src="../.gitbook/assets/flash_partitions_upgrade.png" alt=""><figcaption></figcaption></figure>

Proses alur upgrade dengan dua partisi OTA seperti gambar di bawah ini. OTA Data menyimpan informasi partisi mana yang aktif selagi partisi lain sedang melakukan proses penulisan firmware baru.

<figure><img src="../.gitbook/assets/upgrade_flow.png" alt=""><figcaption></figcaption></figure>

## Updating Flash Partition

OTA mengharuskan konfigurasi Tabel Partisi perangkat dengan setidaknya dua partisi slot aplikasi OTA (yaitu, ota\_0 dan ota\_1) dan Partisi Data OTA. Pengaturan seperti ini mengakibatkan ketika terjadi kesalahan error update, device dapat kembali ke firmware lama yang tidak error.

Berikut merupakan contoh pengaturan custom partition melalui menuconfig. Pada contoh ini offset partition table dimulai pada offset 0x9000 sehingga konfigurasi selanjutnya ada pada offset 0x10000.

<figure><img src="../.gitbook/assets/Screenshot 2025-04-28 171219.png" alt=""><figcaption></figcaption></figure>

Anda perlu menambahkan partitions.csv pada root folder project Anda seperti ini.

```csv
//
    file: partitions.csv
//

nvs,        data,   nvs,    0x10000,     0x6000
otadata,    data,   ota,    0x16000,     0x2000
phy_init,   data,   phy,    0x18000,     0x2000
factory,    0,      0,      0x20000,     0x130000
ota_0,      0,      ota_0,  0x150000,    0x130000
ota_1,      0,      ota_1,  0x280000,    0x130000
```

ESP-IDF juga telah menyediakan template partition yang dapat digunakan langsung.  Biasanya untuk kebutuhan OTA, pilihan template partition yang digunakan adalah Factory app, two OTA definitions

<figure><img src="../.gitbook/assets/Screenshot 2025-04-28 171256.png" alt=""><figcaption></figcaption></figure>

## Enable Rollback

Gunakan fitur rollback jika terjadi sesuatu yang tidak diinginkan. Aktifkan pada menuconfig **Bootloader config -> Enable app rollback support**

<figure><img src="../.gitbook/assets/Screenshot 2025-04-29 215439.png" alt=""><figcaption></figcaption></figure>

## Signed Image

Device perlu diatur untuk mengaktifkan pengaturan agar file firmware yang diperbolehkan untuk dieksekusi saat update adalah file yang valid. Caranya yaitu sebagai berikut.

1. Buka ESP-IDF Terminal
2. Ketikkan idf.py menuconfig maka akan muncul seperti ini. Lalu buka **Security features**
3. Centang pada bagian **Require signed app images.**

<figure><img src="../.gitbook/assets/Screenshot 2025-04-28 171737.png" alt=""><figcaption></figcaption></figure>

4. Lalu simpan dengan tekan S. Dengan begitu device tidak bisa begitu saja melakukan update OTA dengan file sembarangan. File yang diijinkan hanya yang telah di signed ketika proses build.
5. Build project agar terbentuk file .bin pada folder /build.
6. Buka lagi terminal ESP-IDF dan ketikkan perintah berikut untuk membuat key yang dibutuhkan saat signing images.

{% code overflow="wrap" %}
```bash
// Generate Signing Key
espsecure.py generate_signing_key ./secure_boot_signing_key.pem
```
{% endcode %}

7. Sign images

{% code overflow="wrap" %}
```
// Sign Binary
espsecure.py sign_data --version 1 --keyfile ./secure_boot_signing_key.pem --output ./App-esp32_signed.bin ./App-esp32.bin
```
{% endcode %}
