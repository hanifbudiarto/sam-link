---
icon: bullhorn
---

# Raw Subscribe & Publish

Pada contoh sebelumnya, proses subscribe dan mendapatkan data tidak ditunjukkan dengan jelas karena telah dienkapsulasi oleh library. Semua sudah dilakukan dibaliknya dan data dikirimkan langsung melalui callback yang disediakan.&#x20;

Bagi pengembang yang ingin melakukan subscribe ke topic lain maupun publish ke topic lain, dapat digunakan function **publish\_raw** dan **subscribe\_raw**.

{% code overflow="wrap" %}
```c
/*
    source : SAMLink Library
*/
void (*publish_raw)(const char *topic, const char *value, const samlink_qos qos, const samlink_retained retained);

void (*subscribe_raw)(const char *topic, const samlink_qos qos);
```
{% endcode %}

Publish ke sebuah topic memerlukan atribut topic itu sendiri, nilai, QOS dan retain value. QOS adalah Quality of Service. Anda bisa membaca lebih lanjut mengenai tersebut pada tautan ini [What is QOS](https://www.hivemq.com/blog/mqtt-essentials-part-6-mqtt-quality-of-service-levels/). Sedangkan untuk subscribe hanya membutuhkan topic dan QOS. Data yang diperoleh dari proses subscribe melalui **subscribe\_raw** dapat dilihat pada callback **mqtt\_on\_raw\_data\_received**.&#x20;

{% code overflow="wrap" %}
```c
void on_raw_data_callback(const char *topic, const char *data, int retained);
{
    
}

static const sam_element_callback callback = {
     .mqtt_on_raw_data_received = on_raw_data_callback
};

void app_main(void)
{
     samlink = SAMLink_new(&config);
     samlink->register_callback(&callback);
     samlink->start();
}
```
{% endcode %}
