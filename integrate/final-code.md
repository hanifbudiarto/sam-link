---
icon: code
---

# Final Code

{% code lineNumbers="true" %}
```c
/*
    file : main_example.c
*/

#include "sam_link.h"
#include "dht.h"

// define our library
SAMLink *samlink;

// define required library configuration
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

// define a node
sam_element_node *sensor_node;

// common callback for our testing purpose
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

// available pin
const int reset_btn_pin = 15;
const int yellow_led_pin = 33;

// Init pin output dan pin reset
static const sam_element_pin pin = {
     .output_pin = yellow_led_pin,
     .reset_pin = reset_btn_pin
};

// our temperature and humidity task
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

// our relay task
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

// queue for send and receive signal
QueueHandle_t queue1;

// pin for motion sensor
int motion_pin = 27;

// Interrupt Service Routine (ISR) for Motion Sensor
static void IRAM_ATTR motion_gpio_interrupt_handler(void *args)
{
    int pin = (int) args;
    
    // send signal
    xQueueSendFromISR(queue1, &pin, NULL);
}

// our motion task
void motion_sensor_task(void *pvParameter) {
    int pinNumber;
    while(1) {
        // on hold, waiting to receive signal 
        if(xQueueReceive(queue1, &pinNumber, portMAX_DELAY )) {
            ESP_LOGI("main_example", "GPIO %d was pressed", pinNumber);
        }
    }
}

void app_main(void)
{
    // init our library
    samlink = SAMLink_new(&config);

    // register callbacks and pins
    samlink->register_callback(&callback);
    samlink->assign_pin(&pin);

    // create a node
    sensor_node = SAMLink_create_node("sensor", "Sensor", "Sensor-01", NON_CONFIG);

    // add properties to the node
    sensor_node->add_properties(sensor_node, &temp);
    sensor_node->add_properties(sensor_node, &humid);
    sensor_node->add_properties(sensor_node, &relay);
    sensor_node->add_properties(sensor_node, &motion);

    // add node to our library
    samlink->add_node(sensor_node);

    // start the library
    samlink->start();

    // init queue
    queue1 = xQueueCreate(5, sizeof(int));

    // set pin mode
    gpio_set_direction(motion_pin, GPIO_MODE_INPUT);
    gpio_pulldown_en(motion_pin); // enable pulldown
    gpio_pullup_dis(motion_pin); // disable pullup
    // set interrupt type
    gpio_set_intr_type(motion_pin, GPIO_INTR_POSEDGE);
    
    // create a DHT and Btn Task
    xTaskCreate(&dht_task, "DHT task", configMINIMAL_STACK_SIZE * 3, NULL, 11, NULL);
    xTaskCreate(&btn_task, "Button task", configMINIMAL_STACK_SIZE * 3, NULL, 10, NULL);
    
    // motion sensor task to receive signal from ISR
    xTaskCreate(&motion_sensor_task, "Motion Sensor Task", configMINIMAL_STACK_SIZE * 5, NULL, 13, NULL);

    // add handler for interrupt
    gpio_isr_handler_add(motion_pin, motion_gpio_interrupt_handler, (void *)motion_pin);

}

```
{% endcode %}
