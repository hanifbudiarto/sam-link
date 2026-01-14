---
icon: code-simple
---

# Complete Code

Selamat! Sampai tahap ini Anda berhasil menghubungkan device Anda dengan platform IoT SAM Element.&#x20;

Berikut adalah baris kode lengkap dari contoh sebelumnya.

{% code lineNumbers="true" %}
```c
/*
    file : main_example.c
*/

#include "sam_link.h"

SAMLink *samlink;

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

sam_element_node *sensor_node;

void last_value_callback(const char *value) {}

void on_set_relay(const char *value) {}

// define temperature properties
static sam_element_properties temp = {
    .id = "temp",
    .name = "Temperature", 
    .datatype = FLOAT,
    .settable = READ_ONLY, // hanya bisa dibaca nilainya
    .retained = RETAINED, // tidak butuh nilai temperatur terakhir
    .last_value_callback = last_value_callback
};

// define humidity properties
static sam_element_properties humid = {
    .id = "humid",
    .name = "Humidity",
    .datatype = FLOAT,
    .settable = READ_ONLY, // hanya bisa dibaca nilainya
    .retained = RETAINED, // tidak butuh nilai kelembapan terakhir
    .last_value_callback = last_value_callback
}; 

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

// define motion properties
static sam_element_properties motion = {
    .id = "motion",
    .name = "Motion",
    .datatype = BOOLEAN,
    .settable = READ_ONLY, // hanya bisa baca nilai
    .retained = NON_RETAINED // device kembali ke state awal setelah ada pendeteksian gerakan
};

void on_connect_callback() {}
void on_reconnect_callback() {}
void on_raw_data_callback(const char *topic, const char *data, int retained) {}
void on_wifi_timeout_callback() {}
void before_reset_callback() {}
void before_restart_callback() {}

static const sam_element_callback callback = {
    .mqtt_on_connected = on_connect_callback,
    .mqtt_on_reconnecting = on_reconnect_callback,
    .mqtt_on_raw_data_received = on_raw_data_callback,
    .wifi_on_connect_timeout = on_wifi_timeout_callback, 
    .before_device_reset = before_reset_callback,
    .before_device_restart = before_restart_callback
};

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
    
    samlink->register_callback(&callback);
    samlink->assign_pin(&pin);

    sensor_node = SAMLink_create_node("sensor", "Sensor", "Sensor-01", NON_CONFIG);

    sensor_node->add_properties(sensor_node, &temp);
    sensor_node->add_properties(sensor_node, &humid);
    sensor_node->add_properties(sensor_node, &relay);
    sensor_node->add_properties(sensor_node, &motion);

    samlink->add_node(sensor_node);

    samlink->start();
}

```
{% endcode %}
