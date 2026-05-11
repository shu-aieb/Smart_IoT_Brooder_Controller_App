# Smart Brooder - IoT Climate Controller

Automated environmental acclimatization of post-hatch livestock designed as a native application for android smartphones. 

**`Dynamic Sensor-to-Relay Mapping`** and `**Time-Series Climate Phasing**` are two different engineering problems addressed in this Brooder application, as compared to my last Incubator project on Static Environmental Maintenance & Multi-batch Tracking.

## 📱 Interface Showcase
Learn how to configure the Dashboard, Temperature Phase and Hardware Configuration screens side-by-side.

## 🎯 The Problem & Purpose
Thermal shock is fatal and occurs when livestock is first removed from the hatching room and placed at ambient room temperature. They need a brooder setting with a gradual decrease in temperature over a period of weeks. Moreover, industrial brooders are so many different sizes that zoning, or multiple heaters and sensors placed in various corners of the brooder is very complex.

**The Solution:**
Designed an automatic Step-Down climate engine that monitors a single batch cycle and automatically sets the temperature setpoints. I designed a dynamic mapping matrix for the hardware in the app for varying farm sizes, so the user can use up to 4 sensors to control up to 4 different heating arrays.

## 🏗️ Architecture & Tech Stack

The platform is Native Android (Java, XML).
*   **Frontend / Thermal Bridge:** Website / Application
The ESP8266 Microcontroller serves as the hardware interface.The Hardware Interface is ESP8266 Microcontroller.
There are many methods for communicating with the application.The application can be communicated to several ways:

### Core Modules
```text
app/src/main/java/com/iramelectronics/brooder/
├── core/                 # App-wide utilities, Base Activities
├── network/              # Triple-fallback networking (Cloud, Socket, SMS)
├── engine/               # The Step-Down climate calculation logic
├── matrix/               # Hardware sensor-to-relay mapping configs
└── ui/
    ├── dashboard/        # Live telemetry and active day tracker
    ├── phase_config/     # UI for programming day-by-day temperature drops
    └── settings/         # PIN management, USSD controls, Smart Warranty
```
## ⚡ Key Engineering Decisions
### 1. Automated Climate Phasing (Step-Down Engine)
Created a time-series configuration tool to define phase blocks (Days 1-3 = 37.7°C; Days 4-6 = 37.5°C; Days 7-9 = 37.3°C). This array is written to the hardware by the app. The app is not dependent on the Android device being connected to push the daily update, but rather sets up the phase array on the ESP8266, which is then able to perform the temperature step-downs safely on its own without being connected, with reference to its internal day counter.

### 2. Dynamic Hardware Mapping Matrix
Industrial brooders vary in size, ranging from the small single bulb unit to large rooms with multiple zones heated. I created a very flexible implementation of the configuration layer, supporting:

* 1 Sensor → 1 Device
* 1 Sensor → 2 Devices
* 4 Sensors → 1 Central Device
* 4 Sensors → 2 Zoned Devices
* 4 Sensors → 4 Zoned Devices

The app UI dynamically adapts to display the telemetry for the configuration that was selected by the user; it can aggregate the telemetry to output the room averages, or isolate a zone to use those specific telemetry and get a heating alarm on that zone.

### 3. Triple-Redundancy Network
Re-used and modified the strong communication pipeline developed for the Incubator project. It is defaulted to Firebase, with fallback capability to direct Local-Wi-Fi socket connection for on-site "offline" control, and a secure SMS/GSM command protocol for extreme rural deployment.

### 4. Global Hardware Utilities
* **Remote USSD Execution**: Users can use the app to ping the hardware's SIM card and check the balance of the carrier or buy data packages to keep IoT connectivity.

Smart Warranty Sync: Reads the encrypted, first-boot time stamp from the hardware to show a tamper-resistant, digital warranty expiration screen.

Access Control: Multi-tiers, owner gives out different revocable PINs for 6 operators.
## 🐛 Edge Cases Handled

4 Sensor: If one sensor fails, or the sensor reports impossible sensor data (e.g., -99°C), the app will indicate a Sensor Fault Error (Critical Zone), the UI will indicate the specific hardware fault, and the aggregated data will continue to be displayed from the remaining sensors.

The logic for the day-counter is such that it considers the hardware's internal RTC (Real Time Clock) as the true source of truth to avoid the app and the hardware from disagreeing about the start and end of the “Day 3” to “Day 4” transition.
