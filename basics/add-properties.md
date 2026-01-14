---
icon: genderless
---

# Add Properties

Properties merupakan bagian dari nodes. Sebuah node dapat mengekspos beberapa properties. Properties dapat diartikan sebagai kemampuan sebuah device berupa sensor atau aktuator.

<figure><img src="https://ascencione.com/wp-content/uploads/2023/08/Car-Sensors-AC-Video-GPS-Camera-Fuel-Ultrasonic-Radar-Infrared-Pressure.jpg" alt=""><figcaption></figcaption></figure>

Berikut adalah atribut-atribut dari dari sebuah properties yang ditentukan oleh library ini.

```c
/*
    source : SAMLink Library
    * : required
    (( )) : default value
*/

typedef struct {
    char *id;                  // *
    char *name;                // *
    samlink_settable settable; // ((READ_ONLY))
    samlink_retained retained; // ((NON_RETAINED))
    samlink_datatype datatype; // ((NO_DATATYPE))
    char *unit;                            // ((NULL))
    char *format;                          // ((NULL))
    void (*last_value_callback)(const char *value); // required if RETAINED
    void (*on_set_callback)(const char *value);     // required if READ_WRITE 
    int (*validator)(const char *value);            // custom validator
} sam_element_properties;
```

Melanjutkan dari code sebelumnya

```c
/*
    file : main_example.c
*/

#include "sam_link.h"

SAMLink *samlink;

// define a node
sam_element_node *sensor_node;

static const sam_element_config config = {...};

// define temperature properties
static sam_element_properties temp = {
     .id = "temp",
     .name = "Temperature"
};

// define humidity properties
static sam_element_properties humid = {
     .id = "humid",
     .name = "Humidity"
}; 

void app_main(void)
{
    samlink = SAMLink_new(&config);
    
    sensor_node = SAMLink_create_node("sensor", "Sensor", "Sensor-01", NON_CONFIG);
    
    // add properties
    sensor_node->add_properties(sensor_node, &temp);
    sensor_node->add_properties(sensor_node, &humid);

    samlink->add_node(sensor_node);
    samlink->start();
}
```

Build, Flash dan Monitor maka Anda akan mendapati jumlah properties bertambah menjadi dua.

```
# Console Output

Summary of your device

                Serial Number |         Project |          Model | Nodes | Properties    
**************************************************************************************
               1SCCDBA79B7988 |     Smart Relay |      SAM-SMR01 |     1 |    2
**************************************************************************************
```

## Create Widget

Setelah mendefinisikan nodes dan properties, mari coba untuk membuat sebuah widget. Pada Step 3 pada tahapan membuat widget, properties yang didefinisikan sebelumnya sudah terlihat yaitu Temperature dan Humidity. Namun saat ingin membuat widget dari salah satu atau keduanya tidak ada model widget yang cocok. Hal itu dikarenakan tipe data properties belum dicantumkan.

<div><figure><img src="../.gitbook/assets/SmartSelect_20250411_145641_SAM IoT.jpg" alt="" width="360"><figcaption></figcaption></figure> <figure><img src="../.gitbook/assets/SmartSelect_20250411_145657_SAM IoT.jpg" alt="" width="358"><figcaption></figcaption></figure></div>

## Define Datatype

Mendefinisikan tipe data pada masing-masing properties penting karena pengecekan tipe data dilakukan pada aplikasi SAM IoT begitu juga pada library SAMLink.

Berikut tipe-tipe data yang didukung oleh library ini.

```c
/* 
    source : SAMLink Library
*/

typedef enum {
    NO_DATATYPE,
    STRING,
    BOOLEAN,
    FLOAT,
    INTEGER,
    ENUM,
    COLOR,
    LOCATION
} samlink_datatype;
```

Melanjutkan contoh sebelumnya.

```c
/* 
     file : main_example.c
*/

...

// define temperature properties
static sam_element_properties temp = {
     .id = "temp",
     .name = "Temperature", 
     .datatype = FLOAT
};

// define humidity properties
static sam_element_properties humid = {
     .id = "humid",
     .name = "Humidity",
     .datatype = FLOAT
};

... 
```

Rebuild Project lalu lihat kembali pada aplikasi SAM IoT. Lakukan refresh pada halaman Devices dengan menarik ke bawah layar. Coba lagi Create New Widget dengan properties Temperature

<div><figure><img src="../.gitbook/assets/SmartSelect_20250411_152943_SAM IoT.jpg" alt="" width="351"><figcaption></figcaption></figure> <figure><img src="../.gitbook/assets/SmartSelect_20250411_152957_SAM IoT.jpg" alt="" width="351"><figcaption></figcaption></figure></div>

## Settable and Retained

**Settable** adalah indikator yang menunjukkan peran sebuah properties, dapat berupa sensor saja, aktuator saja maupun sensor sekaligus aktuator. Sensor berperan dalam mengumpulkan informasi dari lingkungan sekitar. Di sisi lain, aktuator mengambil instruksi dari aktor lain dan mengubahnya menjadi tindakan fisik yang dapat memengaruhi lingkungan.

Contoh sebelumnya menunjukkan temperature dan humidity berperan hanya sebagai sensor keadaan lingkungan sekitar saja. Sehingga nilai settable kedua properties tadi adalah **READ\_ONLY**. Beda halnya dengan saklar yang berperan sebagai aktuator yang bisa menerima input dari aktor lain dengan memencet tombol. Nilai settablenya yaitu **READ\_WRITE**.

Nilai settable ditentukan oleh enum berikut ini.

```c
/*
    source : SAMLink Library
*/

typedef enum {
    READ_ONLY,
    READ_WRITE
} samlink_settable;
```

Nilai **retained** menunjukkan bahwa state device tersebut berubah dari satu nilai ke nilai yang lainnya. Contohnya adalah sebuah saklar. Ketika saklar dinyalakan maka state device tersebut adalah nyala. Maka state device akan terus dalam posisi nyala / **RETAINED**. Beda halnya dengan device motion sensor. Ketika ada pergerakan maka device akan berada dalam posisi mendeteksi gerakan. Namun itu hanya terjadi beberapa saat saja. Setelah deteksi selesai maka state device berada kembali pada posisi idle. Maka kondisi yang sesuai adalah **NON\_RETAINED**.

Nilai retained ditentukan oleh enum berikut ini.

```c
/*
    source : SAMLink Library
*/

typedef enum {
    NON_RETAINED,
    RETAINED
} samlink_retained;
```

{% hint style="info" %}
Jika properties bisa di set value maka settable = READ\_WRITE, jika tidak maka settable = READ\_ONLY.

Jika state device berubah ke kondisi semula sesaat setelah ada perubahan nilai, maka retained NON\_RETAINED, jika tidak maka RETAINED.
{% endhint %}

Melanjutkan contoh sebelumnya dengan menambah dua properties lagi.

```c
/* 
     file : main_example.c
*/

...

// define temperature properties
static sam_element_properties temp = {
     .id = "temp",
     .name = "Temperature", 
     .datatype = FLOAT,
     .settable = READ_ONLY, // hanya bisa dibaca nilainya
     .retained = RETAINED // nilai temperature persistent dan tidak kembali ke nilai awal
};

// define humidity properties
static sam_element_properties humid = {
     .id = "humid",
     .name = "Humidity",
     .datatype = FLOAT,
     .settable = READ_ONLY, // hanya bisa dibaca nilainya
     .retained = RETAINED // nilai humidity persistent dan tidak kembali ke kondisi awal
}; 

// define relay properties
static sam_element_properties relay = {
     .id = "relay",
     .name = "Relay",
     .datatype = BOOLEAN,
     .settable = READ_WRITE, // bisa di set nilai
     .retained = RETAINED // nilai relay persistent dan tidak kembali ke kondisi awal
};

// define threshold properties
static sam_element_properties motion = {
     .id = "motion",
     .name = "Motion",
     .datatype = BOOLEAN,
     .settable = READ_ONLY, // hanya bisa baca nilai
     .retained = NON_RETAINED // device kembali ke state awal setelah ada pendeteksian gerakan
};

...
```

Build, Flash dan Monitor maka Anda akan mendapatkan pesan seperti berikut. Property sensor mendapatkan peringatan karena tidak mencantumkan callback **last\_value\_callback**

```
W (1406) SAMLink: node=sensor, prop=temp
```

## Callbacks

Ada dua buah callbacks yang disediakan oleh library SAMLink yaitu **on\_set\_callback** dan **last\_value\_callback**.

**on\_set\_callback** akan dipanggil ketika ada nilai baru yang dikirimkan dari aplikasi. Di dalam callback function ini pengembang bisa mengolah data tersebut sebelum menerapkan ke device. Sedangkan **last\_value\_callback** adalah callback nilai terakhir device yang disimpan pada server.

{% hint style="info" %}
Jika settable READ\_WRITE maka definisikan on\_set\_callback.

Jika retained RETAINED maka definisikan last\_value\_callback&#x20;
{% endhint %}

Terapkan pada contoh sebelumnya.

```c
/*
    file : main_example.c
*/

...

void last_value_callback(const char *value) { }

// define temperature properties
static sam_element_properties temp = {
     .id = "temp",
     .name = "Temperature", 
     .datatype = FLOAT,
     .settable = READ_ONLY, // hanya bisa dibaca nilainya
     .retained = RETAINED, // nilai temperature persistent dan tidak kembali ke nilai awal
     .last_value_callback = last_value_callback
};

// define humidity properties
static sam_element_properties humid = {
     .id = "humid",
     .name = "Humidity",
     .datatype = FLOAT,
     .settable = READ_ONLY, // hanya bisa dibaca nilainya
     .retained = RETAINED, // nilai humidity persistent dan tidak kembali ke kondisi awal
     .last_value_callback = last_value_callback
}; 

...
```

Output yang dihasilkan.

```
W (1406) SAMLink: on_set_callback is required when using settable READ_WRITE
W (1406) SAMLink: last_value_callback is required when using RETAINED
W (1406) SAMLink: node=sensor, prop=relay
```

**last\_value\_callback** sudah diset sehingga warningnya hilang. Selanjutnya kita akan terapkan **on\_set\_callback** pada property relay.

```c
void last_value_callback(const char *value) { }

void on_set_relay(const char *value) {}

// define relay properties
static sam_element_properties relay = {
    .id = "relay",
    .name = "Relay",
    .datatype = BOOLEAN,
    .settable = READ_WRITE, // bisa di set nilai
    .retained = RETAINED, // tidak butuh status relay terakhir
    .on_set_callback = on_set_relay,
    .last_value_callback = last_value_callback
};
```

## Validator

Validator merupakan function yang berguna untuk memvalidasi data yang masuk. Secara umum default validator sudah ada untuk mengecek data yang masuk sesuai dengan tipe datanya. Namun jika Anda ingin membuat validator sendiri juga bisa. Validator yang Anda buat akan mereplace validator defaultnya.

Jika Anda ingin membuat validator sendiri maka validator harus mengembalikan nilai 0 jika valid. Sedangkan jika invalid maka kembalikan nilai negatif.

```c
int float_validator(const char *numbers)
{
    char *foo;

    double d = strtod(numbers, &foo);
    if (foo == numbers) {
        printf("invalid number.\n");
        return -1;
    }
    else if (foo[strspn(foo, " \t\r\n")] != '\0') {
        printf("invalid (non-white-space) trailing characters.\n");
        return -2;
    }
    else {
        printf("valid number: %lf\n", d);
        return 0;
    }
}
```
