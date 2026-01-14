---
icon: pen-to-square
---

# Library Configuration

Memahami konfigurasi apa saja yang disediakan oleh library ini.

```c
/*
    source : SAMLink Library
    * : required
*/

typedef struct 
{
    samlink_developer_config developer; // *
    samlink_project_config project;     // *
    samlink_ap_config ap;
    samlink_build_config build;
} sam_element_config;
```

## Developer Config (Required)

Konfigurasi developer merupakan pengaturan yang esensial dan harus diisi oleh pengembang. Variabel yang perlu diisi adalah identitas **root** developer, **username** token dan **password** token. Semuanya bisa didapat pada laman Developer Dashboard.

```c
/*
    source : SAMLink Library
    * : required
*/

typedef struct
{
    int root;            // *
    char username[1024]; // *
    char password[1024]; // *
} samlink_developer_config;
```

## Project Config (Required)

Konfigurasi project merupakan pengaturan mengenai project yang saat ini sedang dikerjakan seperti nama device, model device, nama firmware dan versi firmware.

```c
/*
    source : SAMLink Library
    * : required
*/

typedef struct
{
    char name[32];       // *
    char model[32];      // *
    char fw_name[32];    // *
    char fw_version[32]; // *
} samlink_project_config;
```

## Access Point Config (Optional)

Konfigurasi penamaan access point SSID dan IP yang digunakan sebagai web server.

```c
/*
    source : SAMLink Library
    (( )) : default value
*/

typedef struct
{
    char ssid[16];    // (("SAMLink"))
    char ip[16];      // (("192.168.1.1"))
    char gateway[16]; // (("192.168.1.1"))
    char netmask[16]; // (("255.255.255.0"))

} samlink_ap_config;
```

## Build Config (Optional)

Jika akan digunakan untuk lingkungan produksi, aktifkan pengaturan ini. Saat ini diaktifkan maka visibility log berada pada level info. Informasi debug yang untuk developer gunakan saat pengembangan tidak lagi muncul.

<pre class="language-c"><code class="lang-c"><strong>/*
</strong><strong>    source : SAMLink Library
</strong>    (( )) : default value
*/

<strong>typedef struct
</strong>{
    int prod; // ((0)) or ((false))
    samlink_qos default_qos; // ((AT_MOST_ONCE))
    samlink_session session; // ((CLEAN))
} samlink_build_config;
</code></pre>

### QOS

QoS mempunyai singkatan _Quality of Service_ (kualitas pelayanan). Dalam MQTT, ada 3 macam QoS, yaitu 0, 1, dan 2. Dalam library ini QOS dienkapsulasi menjadi sebuah enum yaitu **samlink\_qos**.

```c
typedef enum {
    AT_MOST_ONCE,
    AT_LEAST_ONCE,
    EXACTLY_ONCE
} samlink_qos;
```

QOS 0 atau **AT\_MOST\_ONCE** : Sama dengan halnya kita melakukan fire and forget. Artinya tidak ada jaminan kalau pengiriman pesan tersebut akan sampai ke subscriber.

<figure><img src="https://www.hivemq.com/sb-assets/f/243938/1024x360/41d4e98134/qos-levels_qos0.webp" alt="" width="375"><figcaption></figcaption></figure>

QOS 1 atau **AT\_LEAST\_ONCE** : Pesan dijamin sampai minimal satu kali ke subscriber. Jika menggunakan QOS ini perlu memperhatikan kemungkinan menerima pesan duplikat.

<figure><img src="https://www.hivemq.com/sb-assets/f/243938/1024x360/529ab80be0/qos-levels_qos1.webp" alt="" width="375"><figcaption></figcaption></figure>

QOS 2 atau **EXACTLY\_ONCE** : Pesan akan dijamin untuk sampai tepat satu kali ke subscriber. Namun proses ini melibatkan proses yang lebih panjang dan lama dibandingkan yang lainnya. Namun QOS ini paling reliable.

<figure><img src="https://www.hivemq.com/sb-assets/f/243938/1024x360/3b314a5496/qos-levels_qos2.webp" alt="" width="375"><figcaption></figcaption></figure>
