# MQTT

**Session Purpose:**  
The general goal of this lab session is to develop the necessary skills to implement an ESP32-based embedded system with Wi-Fi connectivity and remote control capability via a web interface.

## Lab 1   
### 1) Activity Objectives    
_For this lab, we will create a list that will give us data about nearby Wi-Fi connections and modify the code to sort the results by RSSI (from strongest to weakest signal)._  
1) SSID  
2) RSSI  
3) Channel  
4) Authentication mode (Open, WPA2, etc.)
---

### 2) Questions pt.1  
1) _What is the SSID of the strongest AP detected in your scan?_  
2) _What is the RSSI value of that AP?_  
3) _What authentication mode does that AP use?_  
4) _On which channel is that AP operating?_  
5) _Which channel had the most APs detected in your scan?_  
---

### 3) Materials & Setup  
**BOM (Bill of Materials)**

|#|Item|Qty|Link/Source|Cost (MXN)|Notes|
|---------|--------|------|--------|--------|--------|
|1|ESP32|1|amazon|$365|None|

**_Tools / Software_**   

* _OS/Environment: ESP-IDF with FreeRTOS on ESP32 (Windows)_  
* _Editors: VS Code with ESP-IDF extension, C/C++_  
* _Debug/Flash: ESP-IDF Monitor and flashing tools_  

**_Wiring / Safety_**  

* _Board power: USB 5 V from host PC_  
* _Peripherals: None required for this lab_  
* _Safety notes: Check USB connection and avoid disconnections during flashing_  
---

### 4) Procedure   
* _**Step 1:** Initialize the ESP32 Wi-Fi system._  
* _**Step 2:** Execute a scan of nearby Wi-Fi networks._  
* _**Step 3:** Retrieve information of the detected access points._  
* _**Step 4:** Sort the results by signal strength (RSSI)._  
* _**Step 5:** Display the results on the serial monitor._  
---

### 5) Analysis   
_The system allows scanning nearby Wi-Fi networks using the ESP32 Wi-Fi driver._  

_NVS initialization is required for the proper operation of the Wi-Fi subsystem._  

_The ESP32 operates in station mode to detect nearby access points._  

_RSSI allows determining the signal strength of each detected network._  

_The results were sorted by RSSI to show the network with the strongest signal first._  

_This allows identifying the best available network based on signal strength._  

---

### 6) Questions pt.2
1) _What is the SSID of the strongest AP detected in your scan?_    
IBERO    
2) _What is the RSSI value of that AP?_    
-52 dBm    
3) _What authentication mode does that AP use?_    
WPA2-ENT    
4) _On which channel is that AP operating?_    
Channel 1  
5) _Which channel had the most APs detected in your scan?_  
Channel 1     
---

### 7) Code  