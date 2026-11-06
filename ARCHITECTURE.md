# Architecture and Data Flow

1. The APDS9960 senses changes in reflected infrared light as a hand moves.
2. The APDS9960 exposes gesture data to the ESP32 over I²C.
3. The ESP32 firmware reads a gesture direction (UP, DOWN, LEFT, RIGHT).
4. The ESP32 sends a JSON HTTP POST over Wi-Fi to the Raspberry Pi 3.
5. The Raspberry Pi Flask server validates the command.
6. The server maps the gesture to an output and toggles that output.
7. The server returns a JSON acknowledgement.

## Example request

```json
{"gesture":"LEFT","source":"esp32"}
```

## Example response

```json
{"ok":true,"gesture":"LEFT","output":"appliance_3","state":"ON"}
```

The gesture mapping and output pins in this repository are configurable example defaults, not confirmed historical pin assignments.
