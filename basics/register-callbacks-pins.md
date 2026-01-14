---
icon: phone-arrow-right
---

# Register Callbacks & Pins

Terdapat callbacks lain yang disediakan oleh library dan dapat digunakan oleh pengembang device untuk pemrosesan lebih dalam.

<figure><img src="https://miro.medium.com/v2/resize:fit:1100/format:webp/1*vBp7f-zK1UeO18OHyYt11w.jpeg" alt="" width="375"><figcaption></figcaption></figure>

### Available Callbacks

Berikut adalah daftar callbacks yang disediakan.

```c
/*
    source : SAMLink Library
*/

typedef struct
{
    samlink_void_callback mqtt_on_connected;
    samlink_void_callback mqtt_on_reconnecting;
    samlink_data_callback mqtt_on_raw_data_received;
    samlink_void_callback wifi_on_connect_timeout;
    samlink_void_callback before_device_restart;
    samlink_void_callback before_device_reset;
} sam_element_callback;
```

**before\_device\_reset** merupakan callback yang dipanggil ketika tombol reset ditekan dan sebelum device malakukan reset factory. Namun sebelumnya harus didefinisikan terlebih dahulu pin yang digunakan.

**before\_device\_restart** merupakan callback yang dipanggil ketika device mendapat perintah restart dan akan melakukan restart.&#x20;

**mqtt\_on\_connected** merupakan callback ketika device berhasil terhubung ke mqtt server.

**mqtt\_on\_reconnecting** merupakan callback ketika device berusaha reconnecting ke server.

**mqtt\_on\_raw\_data\_received** merupakan callback lain yang dipanggil ketika ada data yang dikirim ke device dari aplikasi selain **on\_set\_callback**. Bedanya callback **on\_set\_callback** terpaku pada masing-masing properties. Sedangkan callback **mqtt\_on\_raw\_data\_received** merupakan callback umum yang menerima berbagai macam data yang dikirimkan dari berbagai properties. Di dalamnya perlu dilakukan pengecekan lebih lanjut.&#x20;

**wifi\_on\_connect\_timeout** merupakan callback yang dipanggil ketika device tidak berhasil terhubung ke WiFi Access Point.

```c
/*
    file : main_example.c
*/

void on_connect_callback();
void on_reconnect_callback();
void on_raw_data_callback(const char *topic, const char *data, int retained);
void on_wifi_timeout_callback();
void before_reset_callback();
void before_restart_callback();

// Init callback
static const sam_element_callback callback = {
     .mqtt_on_connected = on_connect_callback,
     .mqtt_on_reconnecting = on_reconnect_callback,
     .mqtt_on_raw_data_received = on_raw_data_callback,
     .wifi_on_connect_timeout = on_wifi_timeout_callback
     .before_device_reset = before_reset_callback,
     .before_device_restart = before_restart_callback
};

void app_main(void)
{
     samlink = SAMLink_new(&config);

     samlink->register_callback(&callback);
     
     samlink->start();
}
```

### Available Pins

Pin yang tersedia dan dapat didefinisikan yaitu reset pin dan output pin. Reset pin digunakan oleh tombol reset device sedangkan output pin digunakan untuk led yang dapat menunjukkan status device.&#x20;

```c
/*
    source : SAMLink Library
*/

typedef struct
{
    int reset_pin;
    int output_pin;
} sam_element_pin;
```

Led status mempunyai 3 status berdasarkan blinknya.

* Kedip cepat merupakan saat device berada dalam mode access point. Anda dapat terhubung ke WiFi yang dibuat oleh device dan melakukan pengaturan SSID dan Password melalui browser.

<figure><img src="../.gitbook/assets/1000162571(1).gif" alt="" width="187"><figcaption></figcaption></figure>

* Kedip sedang (1 detik) merupakan kondisi saat device berada dalam fase connecting ke WiFi dan server IoT SAM Element.

<figure><img src="../.gitbook/assets/1000162579(1).gif" alt="" width="187"><figcaption></figcaption></figure>

* Tidak berkedip merupakan kondisi saat device berhasil terhubung ke server dan terkoneksi dengan internet.&#x20;

```c
/*
    file : main_example.c
*/

const int reset_btn_pin = 15;
const int yellow_led_pin = 33;

// Init pin output dan pin reset
static const sam_element_pin pin = {
     .output_pin = yellow_led_pin,
     .reset_pin = reset_btn_pin
};

void app_main(void)
{
     samlink = SAMLink_new(&config);

     samlink->assign_pin(&pin);
     
     samlink->start();
}
```

