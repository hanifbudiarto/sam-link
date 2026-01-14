---
icon: bullseye-arrow
---

# Quickstart

<figure><img src="https://docs.espressif.com/projects/vscode-esp-idf-extension/en/latest/_images/new_project_init.png" alt=""><figcaption></figcaption></figure>

ESP-IDF (Espressif IoT Development Framework) adalah sebuah development environment yang dibuat khusus untuk mengembangkan ESP, atau board dari keluarga Espressif.

{% hint style="info" %}
Ingin mempelajari bagaimana membuat projek pertama kali? Ikuti petunjuk lengkapnya melalui[ Start a Project](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/get-started/windows-start-project.html).
{% endhint %}

### Add Library

Tambahkan library SAM Link dan sematkan pada kode seperti berikut. Ini adalah cara singkat menghubungkan perangkat elektronik Anda dengan Platform IoT SAM Element.

```c
/*
    file : main_example.c
*/

#include "sam_link.h"

// define the library
SAMLink *samlink;

// define configuration
static const sam_element_config config = {...};

void app_main(void)
{
    // initiate the library
    samlink = SAMLink_new(&config);
    
    // start
    samlink->start();
}
```

### Complete the Configuration

Seperti yang terlihat pada poin sebelumnya bahwa untuk menginisiasi library SAMLink diperlukan konfigurasi khusus. Berikut adalah isian lengkap dari konfigurasi tersebut. Namun yang dibutuhkan cukup konfigurasi developer dan project.

```c
/*
    source : SAMLink Library
    * : required
    (( )) : default value
*/

typedef struct 
{
    samlink_developer_config developer; // *
    samlink_project_config project;     // *
    samlink_ap_config ap;
    samlink_build_config build;
} sam_element_config;
```

Contoh konfigurasi :&#x20;

```c
/*
    file : main_example.c
*/

// define configuration
static const sam_element_config config = {
     .developer = {
         .root = 1,
         .username = "thisissampleofdeveloperusername",
         .password = "thisissampleofdeveloperpassword"},
     .project = {
         .name = "Smart Relay", 
         .model = "SAM-SMR01", 
         .fw_name = "WDB01", 
         .fw_version = "1.00"
     }
};
```

{% hint style="info" %}
Untuk mengetahui developer root, username dan password Anda silakan membuat akun terlebih dahulu di Developer Dashboard dan buat API key
{% endhint %}
