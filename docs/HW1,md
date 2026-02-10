
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
  To understand basic task creation and scheduling in FreeRTOS by running two concurrent tasks, analyzing how task delays and priorities affect execution order, and observing how the absence of delays can lead to CPU starvation.

- **Lab 2 — Queue: Producer / Consumer:**  
  To learn how to safely pass data between tasks using FreeRTOS queues, observe the behavior of producer and consumer tasks under different execution speeds, and understand buffering and backlog effects when queue capacity or task timing changes.

- **Lab 3 — Mutex: Protect a Shared Resource:**  
  To identify race conditions caused by concurrent access to a shared variable and to correctly solve them using a mutex, demonstrating how mutual exclusion guarantees data consistency even when task priorities differ.

---

## 3) Exercise 1 — Identify Logical Tasks

List the logical tasks that exist in this system.

**Assumptions:**
- The system runs on a microcontroller  
- Timing matters  
- Some operations may block (Wi-Fi, storage)

| Task Name | Trigger | Periodic or Event-Based |
|---------|--------|-------------------------|
| Reads a temperature sensor | 50 ms timer | Periodic |
| Sends sensor data via Wi-Fi | 2 s timer | Periodic |
| Monitors an emergency button | Button press | Event-Based |
| Blinks a status LED | 1 Hz | Periodic |
| Stores error messages | Error event | Event-Based |

---

## 4) Exercise 2 — Task Characteristics

| Task | Time-Critical | Block Safely | Effect if Delayed |
|-----|--------------|--------------|-------------------|
| Reads a temperature sensor | No | Yes | Sampling jitter increases, making trend analysis less reliable. |
| Sends sensor data via Wi-Fi | No | No | Other tasks may starve while the system waits for network completion. |
| Monitors an emergency button | Yes | No | The system may continue operating in an unsafe state longer than allowed. |
| Blinks a status LED | No | Yes | User feedback becomes misleading about the actual system state. |
| Stores error messages | No | Yes | Faults may occur without being traceable during post-mortem analysis. |

---

## 5) Exercise 3 — Priority Reasoning

| Task | Priority (H/M/L) | Justification |
|-----|-----------------|---------------|
| Reads a temperature sensor | Medium | Late execution affects data quality but does not compromise system operation. |
| Sends sensor data via Wi-Fi | Medium | Delays reduce data timeliness but do not immediately impact control logic. |
| Monitors an emergency button | High | Any latency increases the time the system remains in a potentially dangerous state. |
| Blinks a status LED | Low | Incorrect blinking only affects user perception, not system behavior. |
| Stores error messages | Low | Post-event analysis may be incomplete without immediate logging. |

---

## 6) Exercise 4 — Design Judgment (Trick Question)

**Task that should NOT necessarily be implemented as a FreeRTOS task:**  
**Emergency button monitoring**

**Explanation:**  
Emergency button monitoring is better handled using a hardware interrupt to ensure immediate response. A FreeRTOS task may be notified afterward if additional processing is required.

---

## 7) Exercise 5 — Identifying Hidden Tasks in Pseudo-Code

### Original Pseudo-Code

```c
while (1) {
    read_temperature_sensor();          // takes ~2 ms

    if (button_pressed()) {
        emergency_shutdown();           // must react immediately
    }

    if (time_since_last_send() > 2000) {
        send_data_over_wifi();          // may block for 100–300 ms
    }

    toggle_status_led();                // 1 Hz blink rate

    delay_ms(10);
}

---

## 7.1) Exercise 5.1 — Identifying Hidden Tasks

| Hidden Task | Trigger | Why it should be a Task |
|------------|--------|-------------------------|
| Reads a temperature sensor | Periodic | Ensures consistent sampling independent of other operations. |
| Monitors an emergency button | Event | Guarantees fast response to safety-critical events. |
| Sends sensor data via Wi-Fi | Time-based | Prevents long blocking delays from affecting other tasks. |
| Blinks a status LED | Periodic | Maintains a stable and predictable blink rate. |
| Execution delay control | Loop timing | RTOS timing avoids blocking the entire application. |
---
