# ESP32 + APDS9960 Gesture-Controlled Appliance Prototype

A two-node embedded system in which an **ESP32 reads hand gestures from an APDS9960 sensor** and sends the detected gesture over Wi-Fi to a **Raspberry Pi 3**. The Raspberry Pi receives the command over HTTP and switches corresponding prototype outputs.

> **Implementation note:** This repository is a reproducible reference implementation based on the project architecture you described. The previously established details are: ESP32 sensor/client, Raspberry Pi 3 HTTP server, APDS9960 I²C address `0x39`, ESP32 I²C pins GPIO 21/22, and gesture-based appliance control. Exact appliance-to-gesture mapping, Raspberry Pi GPIO assignments, and final relay wiring were not available in the project notes. The mappings and GPIO numbers below are therefore clearly marked as configurable example defaults. Update them to match your actual prototype before claiming those exact details.

## 1. System architecture

```text
       Hand movement
            |
            v
   +-------------------+
   | APDS9960 sensor   |
   | optical gestures |
   +---------+---------+
             | I²C
             | SDA=GPIO21, SCL=GPIO22
             v
   +-------------------+
   | ESP32             |
   | gesture detection |
   | Wi-Fi HTTP client |
   +---------+---------+
             | Wi-Fi / HTTP POST
             v
   +-------------------+
   | Raspberry Pi 3    |
   | Python HTTP server|
   | command handling  |
   +---------+---------+
             | GPIO
             v
   +---------------------------+
   | Prototype outputs         |
   | LED indicators / drivers  |
   | (relay interface optional)|
   +---------------------------+
```

## 2. Hardware

| Component | Role |
|---|---|
| ESP32 development board | Reads the gesture sensor and sends commands |
| APDS9960 breakout/module | Optical gesture sensor |
| Raspberry Pi 3 | HTTP server and output controller |
| 4 LEDs with current-limiting resistors | Safe appliance-status demonstration outputs |
| Breadboard and jumper wires | Prototype interconnections |
| Common regulated power sources | Power each board according to its specification |

### APDS9960-to-ESP32 wiring

| APDS9960 pin | ESP32 connection |
|---|---|
| VCC | 3.3 V (verify breakout-board requirements) |
| VL | 3.3 V if present and required by the breakout |
| GND | GND |
| SDA | GPIO 21 |
| SCL | GPIO 22 |
| INT | Not connected in this polling-based firmware |

The APDS9960 uses I²C address **`0x39`**. The ESP32 firmware explicitly starts I²C with `Wire.begin(21, 22)`.

**Important:** Breakout boards differ. Check the markings and schematic for your exact APDS9960 module before connecting VCC/VL. Do not apply 5 V to ESP32 GPIO or I²C lines.

### Raspberry Pi outputs (example defaults)

The Python server uses BCM GPIO numbering:

| Output | BCM GPIO | Example connection |
|---|---:|---|
| Appliance/LED 1 | 17 | GPIO17 → resistor → LED anode; LED cathode → GND |
| Appliance/LED 2 | 27 | GPIO27 → resistor → LED anode; LED cathode → GND |
| Appliance/LED 3 | 22 | GPIO22 → resistor → LED anode; LED cathode → GND |
| Appliance/LED 4 | 23 | GPIO23 → resistor → LED anode; LED cathode → GND |

These are **example defaults**, not confirmed historical pin assignments. Change `OUTPUT_PINS` in `raspberry_pi/server.py` to match your build.

Do not connect an appliance, motor, fan, or relay coil directly to a Raspberry Pi GPIO. Use a properly rated transistor/MOSFET or relay-driver module, suitable flyback protection for inductive loads, a separate load supply, and a shared reference ground where appropriate. Mains wiring must be handled by a qualified person using suitable isolation and enclosure practices. The included server defaults to low-voltage LED demonstration outputs.

## 3. Example gesture mapping

The mapping below is an editable reference mapping:

| APDS9960 gesture | HTTP command | Example output |
|---|---|---|
| UP | `UP` | Toggle output 1 |
| DOWN | `DOWN` | Toggle output 2 |
| LEFT | `LEFT` | Toggle output 3 |
| RIGHT | `RIGHT` | Toggle output 4 |

The gesture-to-output mapping is configured in `raspberry_pi/server.py`. If your physical demo used a different mapping, edit `GESTURE_TO_OUTPUT`.

## 4. Software requirements

### ESP32
- Arduino IDE
- ESP32 board support package
- Adafruit APDS9960 library
- Built-in `WiFi`, `HTTPClient`, and `Wire` libraries

### Raspberry Pi 3
- Raspberry Pi OS
- Python 3
- Flask
- gpiozero

## 5. Setup

### A. Raspberry Pi server

1. Connect the Pi and ESP32 to the same Wi-Fi network.
2. On the Pi, find its local IP address:

   ```bash
   hostname -I
   ```

3. Copy `raspberry_pi/config.example.py` to `raspberry_pi/config.py` and edit the Wi-Fi network details only if you use the optional helper configuration. The server itself listens on `0.0.0.0:5000`.
4. Install dependencies:

   ```bash
   cd raspberry_pi
   python3 -m venv .venv
   source .venv/bin/activate
   pip install -r requirements.txt
   ```

5. Start the server:

   ```bash
   python server.py
   ```

6. Confirm it is running:

   ```bash
   curl http://127.0.0.1:5000/health
   ```

Expected response:

```json
{"status":"ok","service":"gesture-appliance-controller"}
```

### B. ESP32 firmware

1. Open `esp32_firmware/gesture_client.ino` in Arduino IDE.
2. Install **Adafruit APDS9960** from Library Manager.
3. Select the correct ESP32 board and serial port.
4. Update `WIFI_SSID`, `WIFI_PASSWORD`, and `RASPBERRY_PI_HOST` near the top of the sketch.
5. Upload the firmware and open Serial Monitor at **115200 baud**.
6. Move your hand in front of the sensor. The firmware prints the detected gesture and HTTP response.

Use the Raspberry Pi's IP address from `hostname -I`, for example `192.168.1.50`. Do not include `http://` or a trailing slash in `RASPBERRY_PI_HOST`.

## 6. HTTP API

The ESP32 sends:

```http
POST /gesture
Content-Type: application/json
```

Example body:

```json
{"gesture":"UP","source":"esp32"}
```

The server validates the gesture, toggles the mapped output, and returns the output state:

```json
{"ok":true,"gesture":"UP","output":1,"state":"ON"}
```

Other endpoint:

- `GET /health` — server health check
- `GET /status` — current output states

## 7. Testing and validation

| Test | Procedure | Expected result |
|---|---|---|
| I²C detection | Run an I²C scanner on ESP32 | APDS9960 responds at `0x39` |
| Sensor identity | Read APDS9960 ID using the library/diagnostic sketch | Previously observed chip ID: `0x9E` |
| Gesture recognition | Perform UP, DOWN, LEFT, RIGHT gestures | ESP32 prints the detected direction |
| Network connectivity | Ping Pi from the same LAN | Pi responds |
| HTTP health | `curl http://PI_IP:5000/health` | JSON status `ok` |
| Command handling | POST a test gesture with curl | Server returns success and updates one output |
| End-to-end | Perform a gesture in front of APDS9960 | Corresponding Pi output changes |

Example server-side test:

```bash
curl -X POST http://127.0.0.1:5000/gesture \
  -H "Content-Type: application/json" \
  -d '{"gesture":"UP","source":"manual-test"}'
```

Run the lightweight API tests on a development machine (they use a mock output backend):

```bash
python3 -m unittest discover -s tests -v
```

## 8. Troubleshooting notes

### APDS9960 not detected
- Confirm SDA → GPIO21 and SCL → GPIO22.
- Confirm common ground and correct supply voltage.
- Check whether the exact breakout requires VL to be powered.
- Check pull-up resistors; avoid adding duplicate/too-strong pull-ups.
- Run an I²C scanner and confirm address `0x39`.
- Inspect the module for physical damage. In earlier debugging, the sensor was intermittently detected; the observed address was `0x39`, chip ID was `0x9E`, and a previously used sensor had been damaged and replaced.

### Sensor initializes but gestures are unreliable
- Keep the sensor stable and perform clear, deliberate gestures within its sensing range.
- Avoid strong direct sunlight or reflective interference during testing.
- Confirm the library's gesture engine is enabled.
- Check serial output before investigating the Wi-Fi stage.

### ESP32 cannot reach Raspberry Pi
- Confirm both devices are on the same LAN.
- Use the Pi's current IP address, not `localhost` or `127.0.0.1`, in ESP32 firmware.
- Confirm the server is listening on `0.0.0.0:5000`.
- Check firewall settings and Wi-Fi client isolation.

### Pi output does not change
- Test `/health` and `/status`.
- Send a manual POST using `curl`.
- Verify BCM pin numbering and output wiring.
- Use a driver stage for relays/loads; never drive them directly from GPIO.

## 9. Known limitations and next improvements

- HTTP is unauthenticated and intended only for a trusted local prototype network. Do not expose the server to the public internet.
- The reference firmware uses a simple cooldown to reduce repeated detections.
- The default output implementation is for low-voltage LED indicators. Relay/load integration requires a suitable driver and electrical safety review.
- Gesture mapping and Pi GPIO assignments should be updated to match the final physical prototype.
- Possible improvements: command acknowledgements/retries, configurable gesture mapping, event logging, watchdog/reconnect handling, and a documented relay-driver schematic.

## 10. Repository contents

```text
esp32_firmware/   ESP32 Arduino firmware
raspberry_pi/     Flask HTTP server and GPIO output handling
docs/             Wiring, architecture, API and test documentation
tests/            API unit tests with mock outputs
```

## Project summary

This project demonstrates a distributed embedded architecture: an ESP32 performs local sensor acquisition and gesture detection, Wi-Fi carries high-level commands to a Raspberry Pi 3, and the Pi controls low-voltage output indicators. It combines I²C sensor interfacing, embedded firmware, HTTP communication, GPIO control, and practical hardware debugging.
