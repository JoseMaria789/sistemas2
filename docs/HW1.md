# RTOS Tasks — Labs 1, 2 and 3

---

## 1) Summary

- **Project Name:** LAB 1, 2 and 3
- **Author:** José María Meneses Avendaño
- **Course / Subject:** Embedded Systems II
- **Date:** 10/02/2026
- **Brief Description:**  
  These activities introduce the use of FreeRTOS on the ESP32 by creating concurrent tasks, managing priorities and delays, using queues for inter-task communication, and protecting shared resources with mutexes. Through Labs 1, 2, and 3, common issues such as task starvation, queue overflow, and race conditions are analyzed and solved.

---

## 2) Objectives

### General Objective

- To understand and apply fundamental FreeRTOS concepts for multitasking, inter-task communication, and synchronization in embedded systems.

### Specific Objectives

- **Lab 1 — Two Tasks, Delays, and Priorities:**  
  Understand task creation, scheduling, priorities, and how delays prevent CPU starvation.

- **Lab 2 — Queue: Producer / Consumer:**  
  Learn how to safely exchange data between tasks using queues and analyze buffering behavior.

- **Lab 3 — Mutex: Protect a Shared Resource:**  
  Identify race conditions and solve them using mutexes to guarantee data consistency.

---

## 3) Lab 1 — Two Tasks, Delays, and Priorities

### Original Code (Lab 1)

```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "driver/gpio.h"
#include "esp_log.h"

#define LED_GPIO GPIO_NUM_2   // CHANGE for your board

static const char *TAG = "LAB1";

static void blink_task(void *pvParameters)
{
    gpio_reset_pin(LED_GPIO);
    gpio_set_direction(LED_GPIO, GPIO_MODE_OUTPUT);

    while (1) {
        gpio_set_level(LED_GPIO, 1);
        vTaskDelay(pdMS_TO_TICKS(300));
        gpio_set_level(LED_GPIO, 0);
        vTaskDelay(pdMS_TO_TICKS(300));
    }
}

static void hello_task(void *pvParameters)
{
    int n = 0;
    while (1) {
        ESP_LOGI(TAG, "hello_task says hi, n=%d", n++);
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}

void app_main(void)
{
    ESP_LOGI(TAG, "Starting Lab 1 (two tasks)");

    // Stack size in ESP-IDF FreeRTOS is in BYTES
    xTaskCreate(blink_task, "blink_task", 2048, NULL, 5, NULL);
    xTaskCreate(hello_task, "hello_task", 2048, NULL, 5, NULL);
}
---
## Exercise 1 — Priority Experiment

Instruction:
Change hello_task priority from 5 → 2.
```c
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "driver/gpio.h"
#include "esp_log.h"

#define LED_GPIO GPIO_NUM_2

static const char *TAG = "LAB1";

static void blink_task(void *pvParameters)
{
    gpio_reset_pin(LED_GPIO);
    gpio_set_direction(LED_GPIO, GPIO_MODE_OUTPUT);

    while (1) {
        gpio_set_level(LED_GPIO, 1);
        vTaskDelay(pdMS_TO_TICKS(300));
        gpio_set_level(LED_GPIO, 0);
        vTaskDelay(pdMS_TO_TICKS(300));
    }
}

static void hello_task(void *pvParameters)
{
    int n = 0;
    while (1) {
        ESP_LOGI(TAG, "hello_task says hi, n=%d", n++);
        vTaskDelay(pdMS_TO_TICKS(1000));
    }
}

void app_main(void)
{
    ESP_LOGI(TAG, "Starting Lab 1 (two tasks)");

    xTaskCreate(blink_task, "blink_task", 2048, NULL, 5, NULL);
    xTaskCreate(hello_task, "hello_task", 2048, NULL, 2, NULL);
}

**Result & Explanation**

The behavior does not change noticeably because both tasks use vTaskDelay(), allowing FreeRTOS to switch between tasks even when priorities differ.
---
## Exercise 2 — Starvation Demo

Instruction:
Temporarily remove vTaskDelay() from hello_task.

-Modified Code (Problematic Version)
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "driver/gpio.h"
#include "esp_log.h"

#define LED_GPIO GPIO_NUM_2

static const char *TAG = "LAB1";

static void blink_task(void *pvParameters)
{
    gpio_reset_pin(LED_GPIO);
    gpio_set_direction(LED_GPIO, GPIO_MODE_OUTPUT);

    while (1) {
        gpio_set_level(LED_GPIO, 1);
        vTaskDelay(pdMS_TO_TICKS(300));
        gpio_set_level(LED_GPIO, 0);
        vTaskDelay(pdMS_TO_TICKS(300));
    }
}

static void hello_task(void *pvParameters)
{
    int n = 0;
    while (1) {
        ESP_LOGI(TAG, "hello_task says hi, n=%d", n++);
    }
}

void app_main(void)
{
    ESP_LOGI(TAG, "Starting Lab 1 (two tasks)");

    xTaskCreate(blink_task, "blink_task", 2048, NULL, 5, NULL);
    xTaskCreate(hello_task, "hello_task", 2048, NULL, 2, NULL);
}

**What Happens?**

- hello_task runs continuously.

- The LED stops blinking or behaves erratically.

- hello_task monopolizes the CPU.

**Fix (Restore Delay)**
vTaskDelay(pdMS_TO_TICKS(1000));

One-Sentence Explanation

Blocking allows FreeRTOS to schedule other tasks, preventing CPU starvation.
---
## Lab 2 — Queue: Producer / Consumer
Original Code (Lab 2)
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/queue.h"
#include "esp_log.h"

static const char *TAG = "LAB2";
static QueueHandle_t q_numbers;

static void producer_task(void *pvParameters)
{
    int value = 0;

    while (1) {
        value++;

        // Send to queue; wait up to 50ms if full
        if (xQueueSend(q_numbers, &value, pdMS_TO_TICKS(50)) == pdPASS) {
            ESP_LOGI(TAG, "Produced %d", value);
        } else {
            ESP_LOGW(TAG, "Queue full, dropped %d", value);
        }

        vTaskDelay(pdMS_TO_TICKS(200));
    }
}

static void consumer_task(void *pvParameters)
{
    int rx = 0;

    while (1) {
        // Wait up to 1000ms for data
        if (xQueueReceive(q_numbers, &rx, pdMS_TO_TICKS(1000)) == pdPASS) {
            ESP_LOGI(TAG, "Consumed %d", rx);
        } else {
            ESP_LOGW(TAG, "No data in 1s");
        }
    }
}

void app_main(void)
{
    ESP_LOGI(TAG, "Starting Lab 2 (queue)");

    q_numbers = xQueueCreate(5, sizeof(int)); // length 5
    if (q_numbers == NULL) {
        ESP_LOGE(TAG, "Queue create failed");
        return;
    }

    xTaskCreate(producer_task, "producer_task", 2048, NULL, 5, NULL);
    xTaskCreate(consumer_task, "consumer_task", 2048, NULL, 5, NULL);
}

### Exercise 1 — Make the Producer Faster

**Instruction:**
Change producer delay 200 ms → 20 ms.

Modified Code
vTaskDelay(pdMS_TO_TICKS(20));

#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/queue.h"
#include "esp_log.h"

static const char *TAG = "LAB2";
static QueueHandle_t q_numbers;

static void producer_task(void *pvParameters)
{
    int value = 0;

    while (1) {
        value++;

        // Send to queue; wait up to 50ms if full
        if (xQueueSend(q_numbers, &value, pdMS_TO_TICKS(50)) == pdPASS) {
            ESP_LOGI(TAG, "Produced %d", value);
        } else {
            ESP_LOGW(TAG, "Queue full, dropped %d", value);
        }

        vTaskDelay(pdMS_TO_TICKS(20));
    }
}

static void consumer_task(void *pvParameters)
{
    int rx = 0;

    while (1) {
        // Wait up to 1000ms for data
        if (xQueueReceive(q_numbers, &rx, pdMS_TO_TICKS(1000)) == pdPASS) {
            ESP_LOGI(TAG, "Consumed %d", rx);
        } else {
            ESP_LOGW(TAG, "No data in 1s");
        }
    }
}

void app_main(void)
{
    ESP_LOGI(TAG, "Starting Lab 2 (queue)");

    q_numbers = xQueueCreate(5, sizeof(int)); // length 5
    if (q_numbers == NULL) {
        ESP_LOGE(TAG, "Queue create failed");
        return;
    }

    xTaskCreate(producer_task, "producer_task", 2048, NULL, 5, NULL);
    xTaskCreate(consumer_task, "consumer_task", 2048, NULL, 5, NULL);
}

- **Result & Explanation** 

The producer generates data faster than the consumer can process it, causing the queue to fill up quickly and overflow.

### Exercise 2 — Increase Queue Length
-**Modified Code**

#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/queue.h"
#include "esp_log.h"

static const char *TAG = "LAB2";
static QueueHandle_t q_numbers;

static void producer_task(void *pvParameters)
{
    int value = 0;

    while (1) {
        value++;

        // Send to queue; wait up to 50ms if full
        if (xQueueSend(q_numbers, &value, pdMS_TO_TICKS(50)) == pdPASS) {
            ESP_LOGI(TAG, "Produced %d", value);
        } else {
            ESP_LOGW(TAG, "Queue full, dropped %d", value);
        }

        vTaskDelay(pdMS_TO_TICKS(20));
    }
}

static void consumer_task(void *pvParameters)
{
    int rx = 0;

    while (1) {
        // Wait up to 1000ms for data
        if (xQueueReceive(q_numbers, &rx, pdMS_TO_TICKS(1000)) == pdPASS) {
            ESP_LOGI(TAG, "Consumed %d", rx);
        } else {
            ESP_LOGW(TAG, "No data in 1s");
        }
    }
}

void app_main(void)
{
    ESP_LOGI(TAG, "Starting Lab 2 (queue)");

    q_numbers = xQueueCreate(20, sizeof(int)); // length 5
    if (q_numbers == NULL) {
        ESP_LOGE(TAG, "Queue create failed");
        return;
    }

    xTaskCreate(producer_task, "producer_task", 2048, NULL, 5, NULL);
    xTaskCreate(consumer_task, "consumer_task", 2048, NULL, 5, NULL);
}

-**Result & Explanation**
Queue overflow occurs less frequently because more elements can be buffered before reaching capacity.
---

- ##Exercise 3 — Make the Consumer Slow

- **Instruction:**
Add a delay after a successful receive.

Modified Code
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/queue.h"
#include "esp_log.h"

static const char *TAG = "LAB2";
static QueueHandle_t q_numbers;

static void producer_task(void *pvParameters)
{
    int value = 0;

    while (1) {
        value++;

        // Send to queue; wait up to 50ms if full
        if (xQueueSend(q_numbers, &value, pdMS_TO_TICKS(50)) == pdPASS) {
            ESP_LOGI(TAG, "Produced %d", value);
        } else {
            ESP_LOGW(TAG, "Queue full, dropped %d", value);
        }

        vTaskDelay(pdMS_TO_TICKS(20));
    }
}

static void consumer_task(void *pvParameters)
{
    int rx = 0;

    while (1) {
        // Wait up to 1000ms for data
        if (xQueueReceive(q_numbers, &rx, pdMS_TO_TICKS(1000)) == pdPASS) {
            ESP_LOGI(TAG, "Consumed %d", rx);
            vTaskDelay(pdMS_TO_TICKS(300));
        } else {
            ESP_LOGW(TAG, "No data in 1s");
        }
    }
}

void app_main(void)
{
    ESP_LOGI(TAG, "Starting Lab 2 (queue)");

    q_numbers = xQueueCreate(20, sizeof(int)); // length 5
    if (q_numbers == NULL) {
        ESP_LOGE(TAG, "Queue create failed");
        return;
    }

    xTaskCreate(producer_task, "producer_task", 2048, NULL, 5, NULL);
    xTaskCreate(consumer_task, "consumer_task", 2048, NULL, 5, NULL);
}
- **Pattern Observed**
A backlog forms because the producer generates data faster than the consumer can process it.
---
## Lab 3 — Mutex: Protect a Shared Resource
- **Part A — Race Condition (No Mutex)**
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "esp_log.h"

static const char *TAG = "LAB3A";

static volatile int shared_counter = 0;

static void increment_task(void *pvParameters)
{
    const char *name = (const char *)pvParameters;

    while (1) {
        // NOT safe: read-modify-write without protection
        int local = shared_counter;
        local++;
        shared_counter = local;

        if ((shared_counter % 1000) == 0) {
            ESP_LOGI(TAG, "%s sees counter=%d", name, shared_counter);
        }

        vTaskDelay(pdMS_TO_TICKS(1));
    }
}

void app_main(void)
{
    ESP_LOGI(TAG, "Starting Lab 3A (race demo)");

    xTaskCreate(increment_task, "incA", 2048, "TaskA", 5, NULL);
    xTaskCreate(increment_task, "incB", 2048, "TaskB", 5, NULL);
}
- **Why Can the Counter Be Wrong?**
Both tasks can read and write the shared variable simultaneously, causing lost updates due to race conditions.

**Part B — Fix with a Mutex**
- Modified Code
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/semphr.h"
#include "esp_log.h"

static const char *TAG = "LAB3B";

static volatile int shared_counter = 0;
static SemaphoreHandle_t counter_mutex;

static void increment_task(void *pvParameters)
{
    const char *name = (const char *)pvParameters;

    while (1) {
        xSemaphoreTake(counter_mutex, portMAX_DELAY);

        int local = shared_counter;
        local++;
        shared_counter = local;

        xSemaphoreGive(counter_mutex);

        if ((shared_counter % 1000) == 0) {
            ESP_LOGI(TAG, "%s sees counter=%d", name, shared_counter);
        }

        vTaskDelay(pdMS_TO_TICKS(1));
    }
}

void app_main(void)
{
    ESP_LOGI(TAG, "Starting Lab 3B (mutex fix)");

    counter_mutex = xSemaphoreCreateMutex();
    if (counter_mutex == NULL) {
        ESP_LOGE(TAG, "Mutex create failed");
        return;
    }

    xTaskCreate(increment_task, "incA", 2048, "TaskA", 5, NULL);
    xTaskCreate(increment_task, "incB", 2048, "TaskB", 5, NULL);
}
- **Result & Explanation**
The counter increases correctly because the mutex guarantees exclusive access to the shared resource.

### Exercise — Remove the Mutex Again

Removing the mutex causes the race condition to reappear, producing inconsistent counter values.
---
### Exercise — Change Task Priorities
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "freertos/task.h"
#include "freertos/semphr.h"
#include "esp_log.h"

static const char *TAG = "LAB3B";

static volatile int shared_counter = 0;
static SemaphoreHandle_t counter_mutex;

static void increment_task(void *pvParameters)
{
    const char *name = (const char *)pvParameters;

    while (1) {
        xSemaphoreTake(counter_mutex, portMAX_DELAY);

        int local = shared_counter;
        local++;
        shared_counter = local;

        xSemaphoreGive(counter_mutex);

        if ((shared_counter % 1000) == 0) {
            ESP_LOGI(TAG, "%s sees counter=%d", name, shared_counter);
        }

        vTaskDelay(pdMS_TO_TICKS(1));
    }
}

void app_main(void)
{
    ESP_LOGI(TAG, "Starting Lab 3B (mutex fix)");

    counter_mutex = xSemaphoreCreateMutex();
    if (counter_mutex == NULL) {
        ESP_LOGE(TAG, "Mutex create failed");
        return;
    }

    xTaskCreate(increment_task, "incA", 2048, "TaskA", 6, NULL);
    xTaskCreate(increment_task, "incB", 2048, "TaskB", 4, NULL);
}
- **Expected Result**

TaskA executes more frequently, but data integrity is still preserved because the mutex enforces mutual exclusion.



