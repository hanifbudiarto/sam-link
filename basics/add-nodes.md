---
icon: circle-nodes
---

# Add Nodes

Suatu perangkat dapat mengekspos beberapa node. Node merupakan bagian yang independen atau dapat dipisahkan secara logis dari suatu perangkat. Misalnya, sebuah mobil dapat mengekspos node roda, node mesin, dan node lampu.

<figure><img src="https://acko-cms.ackoassets.com/different_types_of_car_sensors_d34f7ed8b5.png" alt="" width="563"><figcaption></figcaption></figure>

Berikut merupakan function yang digunakan untuk menambah node. Parameternya yaitu node yang ingin ditambahkan.

```c
/*
    source : SAMLink Library
*/

void (*add_node)(sam_element_node *node);
```

## Create a Node

Pada tahap sebelumnya device yang baru saja dibuat tidak memiliki node maupun properties satupun atau masih bernilai 0. Mari buat sebuah node untuk menggambarkan sebuah sensor temperature dan kelembapan DHT22. Fungsi yang dipanggil untuk membuat sebuah node adalah sebagai berikut.

<pre class="language-c"><code class="lang-c"><strong>/*
</strong><strong>    source : SAMLink Library
</strong><strong>*/
</strong><strong>
</strong><strong>sam_element_node *SAMLink_create_node(char *id, char *name, char *type, samlink_isconfig is_config)
</strong></code></pre>

Anda harus mengisi **id, name, type** dan **is\_config**. Atribut is\_config menunjukkan apakah atribut tersebut adalah atribut konfigurasi/pengaturan perangkat atau bukan.

```c
/*
    source : SAMLink Library
*/

typedef enum {
    NON_CONFIG,
    CONFIG
} samlink_isconfig;
```

Melanjutkan dari code sebelumnya

```c
/* 
    file : main_example.c
*/

#include "sam_link.h"

// define the library
SAMLink *samlink;

// define a node
sam_element_node *sensor_node;

// define configuration
static const sam_element_config config = {...};

void app_main(void)
{
    // initiate the library
    samlink = SAMLink_new(&config);
    
    // init a node
    sensor_node = SAMLink_create_node("sensor", "Sensor", "Sensor-01", NON_CONFIG);
    
    // call add node function
    samlink->add_node(sensor_node);
    
    // start
    samlink->start();
}
```

Build, Flash dan Monitor maka Anda akan mendapati teks sebagai berikut. Jumlah node yang semula masih kosong sekarang bertambah menjadi satu. Namun node saja tidak cukup, Anda perlu menambahkan property apa saja yang diekspos pada node tersebut.

```
# Console Output

Summary of your device

                Serial Number |         Project |          Model | Nodes | Properties    
**************************************************************************************
               1SCCDBA79B7988 |     Smart Relay |      SAM-SMR01 |     1 |    0
**************************************************************************************
```
