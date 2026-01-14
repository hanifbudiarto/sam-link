---
icon: desktop
---

# Monitor Log

Untuk memonitor log yang dihasilkan oleh firmware, library ini menyediakan sarana untuk memantau melalui browser. Anda cukup buka alamat IP dari device tersebut setelah device berhasil terhubung ke lokal WiFi.&#x20;

Sebelumnya, cari alamat ip dari device Anda. Pada console output cari "got ip" maka Anda akan menemukan baris seperti ini

```
SAMLink: got ip:192.168.100.73
```

Buka browser lalu ketikkan alamat IP tersebut.

<figure><img src="../.gitbook/assets/Screenshot 2025-04-29 162411.png" alt="" width="563"><figcaption></figcaption></figure>

Tidak semua log akan tampil pada halaman ini, namun log-log yang memang sengaja disimpan dengan cara memanggil function tertentu pada library. Function yang disediakan adalah **store\_log**. Anda dapat menggunakan function ini layaknya menggunakan printf pada bahasa pemrograman C.

```c
/*
    file : main_example.c
*/

samlink->store_log("Humidity: %.1f%% Temp: %.1fC", humidity, temperature);
samlink->store_log("GPIO %d was detect motion", pinNumber);
samlink->store_log("Lamp %s", lamp ? "ON" : "OFF");
```
