# Smart Fermentation Chamber - Final Project

## 1. General Description
This project is an automated IoT system designed to monitor and control the environment for sourdough starter or yogurt fermentation. Unlike standard environmental monitors, this system tracks the physical volume expansion of the biomass using ultrasonic distance measurement and maintains a target temperature using a PID feedback loop, reporting all data to a local web server hosted on the microcontroller.

## 2. Bill of Materials (BOM)
| Component Name | Quantity | Description/Notes |
| :--- | :---: | :--- |
| **ESP32 Development Board** | 1 | Main Controller (Wi-Fi + Dual Core) |
| Ultrasonic Sensor (HC-SR04) | 1 | Measures the rise (height) of the fermentation |
| DHT22 (or DHT11) | 1 | Temperature & Humidity Sensor |
| Relay Module (5V) | 1 | Controls the heating element |
| Heating Pad / Lamp | 1 | Actuator for temperature control |
| OLED Display (SSD1306) | 1 | Local status display |
| Jumper wires & Breadboard | - | Connectivity |

## 3. Tutorial Source & Modifications
**Is this based on an existing tutorial?**
- [ ] No, it is an original design.
- [X] Yes. Concept inspired by: [RandomNerdTutorials ESP32 Web Server](https://randomnerdtutorials.com/) + Basic Thermostat Logic.

**What I changed:**
* **Logic Extension:** Instead of a simple "If Temp < Target Then Heater ON" (Bang-Bang control), I am implementing a hysteresis loop to prevent the relay from switching too rapidly.
* **Sensor Fusion:** I added an ultrasonic sensor to correlate temperature changes with the physical rise of the dough, creating a "readiness" index, which standard thermostat tutorials do not have.

## 4. Architecture & Design (The 5 Questions)

### Q1: What is the system boundary?
* **Inside the System:** The fermentation box, the ESP32 controller, the sensors (Ultrasonic, DHT), and the heating element relay. The decision to heat or cool happens strictly on the device.
* **Outside the System:** The user's smartphone/laptop. The phone is used **only** for visualization and setting the target temperature via a web interface. [cite_start]It performs no processing[cite: 346].
* **Boundary:** The HTTP requests over Wi-Fi between the ESP32 and the router.

### Q2: Where does the intelligence live?
* **Location:** The intelligence is entirely on the **Edge (ESP32)**.
* **Reasoning:** The system must maintain temperature reliability even if the Wi-Fi connection is lost. [cite_start]"Distributed intelligence" would make the system fragile. [cite_start]The ESP32 handles the sensor reading loop, the control logic, and the web server simultaneously[cite: 167].

### Q3: What is the hardest technical problem?
* **The Challenge:** **Sensor Noise & False Positives** on the ultrasonic sensor.
* **Why:** The condensation inside a fermentation chamber can cause the ultrasonic sensor to give erratic readings (spikes).
* **Solution:** Implementing a "Moving Average Filter" or a Median Filter in software to smooth out the readings before making logic decisions. [cite_start]This ensures reliability beats ambition[cite: 476].

### Q4: What is the minimum demo?
* [cite_start]**Success Criterion:** The system powers up, displays the current temperature on the OLED, and switches the Relay ON when I hold an ice pack near the sensor (simulating cold), then turns it OFF when the target is reached[cite: 366].
* [cite_start]**The "Boringly Reliable" Check:** It must simply maintain temperature between 24°C and 26°C for 5 minutes without crashing[cite: 463].

### Q5: Why is this not just a tutorial?
* A tutorial typically shows how to wire *one* sensor or how to blink an LED. This project integrates multiple subsystems (Distance, Temp, Relay, Wi-Fi) that compete for CPU time.
* [cite_start]It requires making architectural decisions about **non-blocking timing** (using `millis()`) because the ESP32 cannot pause (`delay()`) while maintaining the web server connection[cite: 162, 193].

## 5. ESP32 Requirement
**Do I need an ESP32 provided by the lab?**
- [X] **YES**
* **Reason:** The project relies on Wi-Fi connectivity to serve the dashboard. [cite_start]The ESP32 is chosen specifically for its dual-core capability to handle networking and sensor logic simultaneously, which fits the "Connectivity + Simple Control" use case described in the course[cite: 258, 261].

## 6. Project Visuals
*(Place your block diagram here - e.g., Sensor -> ESP32 -> Relay)*
