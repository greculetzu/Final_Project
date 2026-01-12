# Smart Fermentation Chamber - Final Project

## 1. Motivation & Real-World Application
**Why does this exist?**
In professional environments like pizzerias or artisanal bakeries, consistency is the hardest metric to achieve. Dough is a living organism; its fermentation rate changes drastically with temperature and humidity.
* **The Problem:** A baker usually relies on timers ("leave it for 4 hours"). However, if the room is 2°C colder than usual, the dough will be under-proofed (dense crust). If it's warmer, it over-proofs (collapses). This leads to food waste and inconsistent product quality.
* **The Solution:** This project moves beyond simple timers. By monitoring the **physical volume expansion** (the actual rise) of the dough and actively controlling the temperature, this system guarantees that the dough is perfectly fermented every time, regardless of the external weather. Ideally, this system would alert the baker via phone exactly when the dough reaches peak rise.

## 2. General Description
This project is an automated IoT system designed to monitor and control the environment for sourdough starter or yogurt fermentation. The system tracks the physical rise of the biomass using ultrasonic distance measurement and maintains a precise target temperature using a PID-based feedback loop. All data is processed locally on the ESP32, which hosts a web dashboard for remote monitoring.

## 3. Bill of Materials (BOM)
| Component Name | Quantity | Description/Notes |
| :--- | :---: | :--- |
| **ESP32 Development Board** | 1 | Main Controller (Wi-Fi + Dual Core) |
| Ultrasonic Sensor (HC-SR04) | 1 | Measures the rise (height) of the fermentation |
| DHT22 (or DHT11) | 1 | Temperature & Humidity Sensor |
| Relay Module (5V) | 1 | Controls the heating element |
| Heating Pad / Lamp | 1 | Actuator for temperature control |
| OLED Display (SSD1306) | 1 | Local status display |
| Jumper wires & Breadboard | - | Connectivity |

## 4. Tutorial Source & Modifications
**Is this based on an existing tutorial?**
- [ ] No, it is an original design.
- [X] Yes. Concept inspired by: [RandomNerdTutorials ESP32 Web Server](https://randomnerdtutorials.com/) + Basic Thermostat Logic.

**What I changed:**
* **Logic Extension:** Instead of a simple "If Temp < Target Then Heater ON" (Bang-Bang control), I am implementing a hysteresis loop logic to prevent the relay from switching too rapidly and damaging the heating element.
* **Sensor Fusion:** I added an ultrasonic sensor to correlate temperature changes with the physical rise of the dough, creating a "readiness percentage" index, which standard thermostat tutorials do not have.

## 5. Architecture & Design (The 5 Questions)

### Q1: What is the system boundary?
* **Inside the System:** The fermentation box, the ESP32 controller, the sensors (Ultrasonic, DHT), and the heating element relay. The decision to heat, cool, or measure happens strictly on the device hardware.
* **Outside the System:** The user's smartphone/laptop. The phone is used **only** for visualization and setting the target temperature via a web interface. It performs no processing logic.
* **Boundary:** The HTTP requests over Wi-Fi between the ESP32 and the local router.

### Q2: Where does the intelligence live?
* **Location:** The intelligence is entirely on the **Edge (ESP32)**.
* **Reasoning:** The system must be robust. If the Wi-Fi connection is lost, the fermentation process must continue safely. Relying on cloud intelligence would introduce latency and risks ("Cloud-first thinking" antipattern). The ESP32 handles the sensor reading loop, the control logic, and the web server tasks simultaneously.

### Q3: What is the hardest technical problem?
* **The Challenge:** **Sensor Noise & Environmental Interference.**
* **Why:** The high humidity inside a fermentation chamber can cause condensation on the ultrasonic sensor, leading to erratic readings (spikes). Additionally, soft dough surfaces absorb sound waves, making readings unstable.
* **Solution:** Implementing a software-based "Moving Average Filter" or "Median Filter" to smooth out the readings before making logic decisions. Reliability is prioritized over raw speed.

### Q4: What is the minimum demo?
* **Success Criterion:** The system powers up, displays the current temperature on the OLED, and switches the Relay ON when the temperature drops below the target (simulated with an ice pack), then turns it OFF when the target is reached.
* **The "Boringly Reliable" Check:** It must maintain temperature between 24°C and 26°C for 5 minutes continuously without crashing or disconnecting.

### Q5: Why is this not just a tutorial?
* A tutorial typically shows how to wire *one* sensor or how to blink an LED. This project integrates multiple subsystems (Distance, Temp, Relay, Wi-Fi) that compete for CPU time.
* It requires making architectural decisions about **non-blocking timing** (using `millis()`) because the ESP32 cannot pause execution (`delay()`) while maintaining the web server connection active.

## 6. ESP32 Requirement
**Do I need an ESP32 provided by the lab?**
- [X] **YES**
* **Reason:** The project relies on Wi-Fi connectivity to serve the dashboard. The ESP32 is chosen specifically for its dual-core capability to handle networking stacks and sensor logic simultaneously, fitting the "Connectivity + Simple Control" architecture required for IoT systems.

## 7. Project Visuals
