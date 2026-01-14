---
icon: hand-point-up
---

# Relay & Motion

Selain perangkat yang berfungsi memproduksi data atau sensor device, selanjutnya mari tambahkan modul yang menerima input dari luar seperti relay dan motion sensor.

## Relay

Gunakan push button untuk berfungsi sebagai relay. Push Button Switch, yang juga dikenal sebagai saklar tombol tekan, merupakan perangkat sederhana yang berfungsi untuk menghubungkan atau memutuskan aliran arus listrik dalam mode kerja tekan-_unlock_ (tidak mengunci). Dalam mode kerja _unlocked_ ini, saklar akan berperan sebagai perangkat penghubung atau pemutus aliran arus listrik ketika tombol ditekan. Ketika tombol tidak ditekan (dilepas), saklar akan kembali ke kondisi normal.

Ada hal lain yang perlu diperhatikan ketika menggunakan push button yaitu debouncing.

<figure><img src="https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEjFSuXyDj0uDZNTnIagVVkKfSMb86K-unrST0Yg9winsBYAuSFAkknewibOYam0IhTYL9Tp6yJZkTuw0TtBQlRUy2qed_X2_3ly1fKKMiNxTh9iv_nqRvclQTe6m5vFEZptoosI0oQyWjcI/w200-h200/push-button-tactile.jpg" alt="" width="188"><figcaption></figcaption></figure>

### What is Debouncing?

Debouncing adalah menghilangkan gangguan input yang tidak diinginkan dari tombol, sakelar, atau input pengguna lainnya. Debouncing mencegah aktivasi tambahan atau fungsi yang lambat agar tidak terlalu sering dipicu.

### Debouncing Hardware Switches

Sakelar fisik adalah perangkat mekanis dengan sedikit pantulan alami. Pegas dan kontak logam dapat menyebabkan sambungan tersambung dan terputus beberapa kali setiap kali sakelar ditekan. Untuk hal-hal sederhana, seperti sakelar lampu, pantulan tidak dapat dideteksi oleh manusia, jadi tidak masalah. Namun, dalam elektronik digital, pantulan tambahan dapat dideteksi dan ditafsirkan sebagai masukan yang disengaja dan menyebabkan masalah. Misalnya, jika Anda menekan tombol ganti saluran pada TV dan saluran berubah dua kali, hal itu mungkin disebabkan oleh masukan yang tidak didebounce dengan benar.

<figure><img src="https://makeabilitylab.github.io/physcomp/arduino/assets/images/SwitchBounce_Oscilliscope_Ladyada.jpg" alt="" width="563"><figcaption></figcaption></figure>

Debouncing switch dapat dilakukan dalam perangkat keras atau perangkat lunak. Debouncing perangkat keras sederhana dapat dibuat dengan komponen pasif. Semua debouncing perangkat keras memerlukan penambahan komponen tambahan dan, oleh karena itu, menambah biaya desain. Namun, menambahkan debouncing switch dalam perangkat lunak adalah hal yang umum dilakukan dengan mikrokontroler. Switch dapat dihubungkan langsung ke input mikrokontroler, dan fungsi debouncing dapat diterapkan ke input untuk menyaring pantulan fisik di switch.

### Example

Untuk mengatasi debouncing, dalam library ini sudah disediakan sebuah class tersendiri untuk menginisiasi button yaitu DebounceButton. Tambahkan task seperti contoh dibawah ini.

```c
/*
    file : main_example.c
*/

void btn_task(void *pvParameter)
{
    int btn_pin = 35;
    
    struct DebounceButton btn = DebounceButton.new(btn_pin);
    btn.register_isr(&btn);

    int lamp = 0;
    while (1)
    {
        if (btn.pressed(&btn))
        {
            lamp = !lamp;
            ESP_LOGI("main_example", "Button Pressed");
            ESP_LOGI("main_example", "Lamp %s", lamp ? "ON" : "OFF");
        }

        vTaskDelay(pdMS_TO_TICKS(10));
    }
}

void app_main(void) {
...

xTaskCreate(&btn_task, "Button task", configMINIMAL_STACK_SIZE * 3, NULL, 10, NULL);

...
}
```

Build, Flash dan Monitor maka Anda akan mendapatkan hasil seperti ini pada console.

```
I (739696) main_example: Button Pressed
I (739696) main_example: Lamp ON
I (739946) main_example: Button Pressed
I (739946) main_example: Lamp OFF
```

## Motion Sensors

<figure><img src="https://dashboard.kmte115unitel.com/wp-content/uploads/2023/12/SR602-PIR-MOTION-SENSOR.png" alt="" width="188"><figcaption></figcaption></figure>

Modul motion sensor yang digunakan adalah SR602. SR602 adalah sensor gerak PIR (Passive Infrared) yang digunakan untuk mendeteksi perubahan suhu yang disebabkan oleh pergerakan objek atau manusia dalam suatu area. Berikut adalah beberapa informasi umum tentang SR602 PIR Motion Sensor:

### Deskripsi

* **Prinsip Kerja:** SR602 bekerja berdasarkan prinsip inframerah pasif, yang mendeteksi perubahan suhu di sekitarnya untuk menentukan adanya pergerakan.
* **Rentang Deteksi:** Sensor ini memiliki cakupan deteksi yang luas dan dapat mendeteksi pergerakan dalam radius tertentu, tergantung pada desain dan spesifikasi sensor.
* **Mode Operasi:** SR602 biasanya memiliki beberapa mode operasi yang dapat diatur sesuai dengan kebutuhan pengguna, seperti mode daya dan mode sensitivitas.
* **Output Sinyal:** Memberikan sinyal keluaran ketika pergerakan terdeteksi.

Untuk menerapkan motion sensor Anda dapat juga menggunakan task seperti sebelumnya dengan melakukan pengecekan pin berulang kali. Namun pada kasus saat ini akan digunakan pendekatan lain menggunakan interrupt. Salah satu pin GPIO akan diinisialisasi dan dikonfigurasi sebagai sumber interupsi. Ini akan memicu interupsi pada sisi positif sinyal input.

## ESP32 GPIO Interrupts

Interupsi GPIO adalah bentuk interupsi eksternal di mana sinyal pemicu eksternal terjadi saat tombol ditekan (misalnya). Saat interupsi dipicu, prosesor menghentikan eksekusi program utama. Pada titik ini, ISR (Interrupt Service Routine) ​​dipanggil. Prosesor kemudian bekerja sementara pada tugas yang berbeda (ISR) dan kemudian kembali ke program utama setelah rutin penanganan berakhir. Gambar di bawah ini menggambarkan prosesnya:

<figure><img src="https://i0.wp.com/esp32tutorials.com/wp-content/uploads/2022/09/Interrupt-Handling-Process.jpg" alt="" width="375"><figcaption></figcaption></figure>

Penggunaan jenis interupsi ini sangat berguna karena kita tidak perlu terus-menerus memantau status pin input digital. Diagram di bawah menunjukkan pin GPIO ESP32 yang dapat kita gunakan untuk interupsi eksternal.

<figure><img src="https://i0.wp.com/esp32tutorials.com/wp-content/uploads/2022/09/ESP32-Interrupt-Pins.jpg" alt="" width="563"><figcaption></figcaption></figure>

### Example

Setup queue dan pin terlebih dahulu untuk menggunakan Interrupt. FreeRTOS Queue digunakan untuk mengirim dan menerima sinyal dari Interrupt Service Routine. Lalu definisikan fungsi yang akan dipanggil saat ada interrupt.

Pada ISR function kirim sinyal dan parameter berupa nomor pin melalui queue agar diterima oleh task lain.

<pre class="language-c"><code class="lang-c"><strong>/*
</strong><strong>    file : main_example.c
</strong><strong>*/
</strong><strong>
</strong>QueueHandle_t queue1;

int motion_pin = 27;

static void IRAM_ATTR motion_gpio_interrupt_handler(void *args)
{
    int pin = (int) args;
    xQueueSendFromISR(queue1, &#x26;pin, NULL);
}

void motion_sensor_task(void *pvParameter) {
    int pinNumber;
    while(1) {
        if(xQueueReceive(queue1, &#x26;pinNumber, portMAX_DELAY )) {
            ESP_LOGI("main_example", "GPIO %d was pressed", pinNumber);
        }
    }
}
<strong>
</strong><strong>void app_main(void) {
</strong><strong>    
</strong><strong>    // init queue
</strong><strong>    queue1 = xQueueCreate(5, sizeof(int));
</strong><strong>    
</strong><strong>    gpio_set_direction(motion_pin, GPIO_MODE_INPUT);
</strong><strong>    gpio_pulldown_en(motion_pin); // enable pulldown
</strong>    gpio_pullup_dis(motion_pin); // disable pullup
    gpio_set_intr_type(motion_pin, GPIO_INTR_POSEDGE);
<strong>    
</strong><strong>    // motion sensor task to receive signal from ISR
</strong><strong>    xTaskCreate(&#x26;motion_sensor_task, "Motion Sensor Task", configMINIMAL_STACK_SIZE * 5, NULL, 13, NULL);
</strong><strong>
</strong><strong>    // add handler for interrupt
</strong><strong>    gpio_isr_handler_add(motion_pin, motion_gpio_interrupt_handler, (void *)motion_pin);
</strong><strong>}
</strong></code></pre>

Build, Flash dan Monitor project maka akan dihasilkan output seperti berikut. Coba dekati sensor motion dengan tangan Anda terlebih dahulu.

```
I (11166) main_example: GPIO 27 was pressed
```
