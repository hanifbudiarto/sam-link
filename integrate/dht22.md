---
icon: fire
---

# DHT22

Pada bagian sebelumnya, yang telah didefinisikan hanyalah perwujudan kemampuan device secara virtual yaitu berupa node dan properties agar dapat terhubung ke platform SAM Element. Selanjutnya yang perlu dilakukan adalah menambahkan modul sensor yang dibutuhkan yang dihubungkan ke ESP32.&#x20;

## Temperature and Humidity Sensor

Untuk sensor temperature dan humidity dalam kasus ini menggunakan ASAIR AM2302. Sensor DHT22 (AM2302) adalah sensor suhu dan kelembaban sederhana dan murah yang menggunakan sensor kelembaban kapasitif dan termistor untuk menghitung data sensor di atmosfer sekitar. Ini termasuk kelembaban relatif dan suhu sekitar.

<figure><img src="https://i0.wp.com/esp32tutorials.com/wp-content/uploads/2022/09/DHT22-Sensor-Pinout.jpg" alt="" width="375"><figcaption></figcaption></figure>

Sebelum mulai, tambahkan dulu kumpulan library lain yang memudahkan untuk mengintegrasikan berbagai macam sensor yaitu [https://github.com/UncleRus/esp-idf-lib](https://github.com/UncleRus/esp-idf-lib). Cara menggunakannya sudah dijelaskan pada halaman Github tersebut.

Pada code example tambahkan sebuah Task yang bertugas untuk membaca data suhu dan kelembapan sekitar.

```c
/*
    file : main_example.c
*/

#include "dht.h"

void dht_task (void *pvParameter) {
    int dht_pin = 18;
    float temperature, humidity;

    while (1)
    {
        if (dht_read_float_data(DHT_TYPE_AM2301, dht_pin, &humidity, &temperature) == ESP_OK)
            ESP_LOGI("main_example", "Humidity: %.1f%% Temp: %.1fC\n", humidity, temperature);
        else
            ESP_LOGE("main_example", "Could not read data from sensor");

        // If you read the sensor data too often, it will heat up
        // http://www.kandrsmith.org/RJS/Misc/Hygrometers/dht_sht_how_fast.html
        vTaskDelay(pdMS_TO_TICKS(2000));
    }
}

void app_main(void) {
...

xTaskCreate(&dht_task, "DHT task", configMINIMAL_STACK_SIZE * 3, NULL, 11, NULL);

...
}
```

Build, Flash dan Monitor maka Anda akan mendapatkan hasil seperti ini pada console.

```
I (739696) main_example: Humidity: 81.3% Temp: 27.9C
I (739696) main_example: Humidity: 81.0% Temp: 27.9C
I (739696) main_example: Humidity: 80.8% Temp: 27.9C
I (739696) main_example: Humidity: 80.1% Temp: 27.9C
I (739696) main_example: Humidity: 79.5% Temp: 27.9C
I (739696) main_example: Humidity: 79.1% Temp: 27.9C
```

## Send Data to Server

Data temperature dan humidity berhasil didapatkan. Selanjutnya data tersebut perlu dikirim ke server agar disimpan dalam database.

```c
/*
    file : main_example.c
*/

void dht_task (void *pvParameter) {
    int dht_pin = 18;
    float temperature, humidity;
    
    // holds string values
    char humidity_str[10]; 
    char temperature_str[10]; 

    while (1)
    {
        if (dht_read_float_data(DHT_TYPE_AM2301, dht_pin, &humidity, &temperature) == ESP_OK) {
            ESP_LOGI("main_example", "Humidity: %.1f%% Temp: %.1fC\n", humidity, temperature);

            // reinitialize
            temperature_str[0] = '\0';
             
            // convert float to string
            sprintf(temperature_str, "%.1f", temperature);
            
            // publish temperature to server
            samlink->publish(sensor_node, &temp, temperature_str);

            // reinitialize
            humidity_str[0] = '\0'; 
            
            // convert float to string
            sprintf(humidity_str, "%.1f", humidity); 
            
            // publish humidity to server
            samlink->publish(sensor_node, &humid, humidity_str);
        }
        else {
            ESP_LOGE("main_example", "Could not read data from sensor");
        }

        // If you read the sensor data too often, it will heat up
        // http://www.kandrsmith.org/RJS/Misc/Hygrometers/dht_sht_how_fast.html
        vTaskDelay(pdMS_TO_TICKS(2000));
    }
}
```
