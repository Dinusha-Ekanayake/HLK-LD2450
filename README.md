# HLK-LD2450 — ESP32 Example (ESP32_Board_2.ino)

This repository contains an ESP32 sketch (ESP32_Board_2.ino) that reads person-tracking frames from the HLK-LD2450 radar module, tracks up to three people, and controls a servo to point toward detected targets. This README documents wiring, configuration, operation, and troubleshooting based on the ESP32_Board_2.ino sketch.

## Features
- Reads HLK-LD2450 frames via UART2 on the ESP32
- Parses up to 3 tracked targets per frame
- Classifies targets as LEFT / MIDDLE / RIGHT using configurable thresholds
- Computes Euclidean distance (mm) from X/Y coordinates
- Tracks targets with timeout-based persistence
- Rotates a servo smoothly to point to active targets, rotating between multiple targets at intervals
- Rotation can be enabled/disabled via an external control input (e.g., from ESPHome)

## Hardware / Requirements
- ESP32 development board
- HLK-LD2450 radar module
- Standard hobby servo (e.g., SG90/FS90) or equivalent (ensure adequate power)
- Stable servo power supply (recommended separate 5V supply)
- Jumper wires and a common ground between radar, ESP32, and servo
- Arduino IDE with ESP32 core or PlatformIO
- ESP32Servo library (install via Library Manager or PlatformIO)

## Pinout / Wiring (as used by the sketch)
These defines are taken from ESP32_Board_2.ino:

- RXD2: GPIO 4  — connect HLK-LD2450 TX -> ESP32 GPIO4
- TXD2: GPIO 2  — connect HLK-LD2450 RX -> ESP32 GPIO2
- SERVO_PIN: GPIO 18 — servo signal
- CONTROL_PIN: GPIO 19 — input; HIGH enables rotation, LOW stops rotation

Wiring summary:
- HLK-LD2450 VCC -> module VCC (check module voltage; many use 5V)
- HLK-LD2450 GND -> ESP32 GND (common ground with servo power)
- HLK-LD2450 TX -> ESP32 GPIO4 (RXD2)
- HLK-LD2450 RX -> ESP32 GPIO2 (TXD2)
- Servo VCC -> 5V power supply (recommended separate supply capable of servo current)
- Servo GND -> common ground
- Servo signal -> ESP32 GPIO18
- Control switch/signal -> ESP32 GPIO19 (configured INPUT_PULLDOWN in code)

Note: If HLK-LD2450 uses 5V logic, use a level shifter or ensure the ESP32 RX pin is protected.

## Serial / UART settings used by the sketch
- USB Serial monitor (Serial): 115200 baud (Serial.begin(115200))
- HLK-LD2450 UART (mySerial, UART2): 256000 baud
  - mySerial.begin(256000, SERIAL_8N1, RXD2, TXD2);

When viewing debug output in Serial Monitor, set baud to 115200.

## How the protocol is parsed (summary)
- The code expects a 32-byte frame:
  - Header: 0xAA 0xFF 0x03 0x00 (first 4 bytes)
  - Remaining 28 bytes are split into 3 target blocks (8 bytes per target used)
- For each target block the sketch reads:
  - raw_x (2 bytes), raw_y (2 bytes), raw_speed (2 bytes), distance_resolution (2 bytes)
- Coordinates use a sign-bit convention implemented in the sketch:
  - If raw & 0x8000 == 0 → value interpreted as negative (target = -raw)
  - Else → value interpreted as positive after masking (target = raw & 0x7FFF)
- Distance (mm) computed as sqrt(x^2 + y^2)

## Target classification & servo mapping
- Distance classification:
  - CLOSE if distance <= 1000 mm
  - FAR if distance > 1000 mm
- Position thresholds:
  - CLOSE: LEFT if X < -150, RIGHT if X > 150, else MIDDLE
  - FAR: LEFT if X < -500, RIGHT if X > 500, else MIDDLE
- Servo positions (degrees in sketch):
  - SERVO_LEFT = 180
  - SERVO_MIDDLE = 145
  - SERVO_RIGHT = 90
- Servo motion: smooth 1° steps with 20 ms delay per step. The sketch also includes an initial delay(2000) inside moveServoTo.

## Persistence & rotation behavior
- Each tracked target stores active, lastDetectedTime, position, distance, and x
- TARGET_TIMEOUT = 1000 ms: target considered lost if not seen within this timeout
- If one active target → servo points to that target immediately
- If two or more active targets → servo rotates between active positions every ROTATION_INTERVAL (2000 ms)
- CONTROL_PIN (GPIO19) controls rotation:
  - LOW → rotation stopped (moveServoTo returns early)
  - HIGH → rotation enabled

## Key constants (from the sketch)
- mySerial (UART2) baud: 256000
- Monitoring Serial: 115200
- TARGET_TIMEOUT: 1000 ms
- ROTATION_INTERVAL: 2000 ms
- Servo PWM: 50 Hz; pulse width min=500 µs, max=2400 µs
- Smooth step delay: 20 ms
- Initial moveServoTo() includes delay(2000)

## Example console output (representative)
HLK-LD2450 Radar with ESP Home Control Initialized
Control pin: 19
HIGH = Rotate Enabled, LOW = Rotate Disabled
--------------------------
Target 1
Position: LEFT
Range: CLOSE
Distance: 875.32 mm
Raw X: -200
Raw Y: 820
Servo Position: 145
Control Pin State: HIGH (Rotate)
--------------------------
Active targets: 2
Rotating between targets, now at position: 180
Target 2 lost (timeout)

## Uploading and usage
1. Open ESP32_Board_2.ino in Arduino IDE or PlatformIO.
2. Install ESP32 core (if needed) and ESP32Servo library.
3. Confirm pin defines at top of sketch match your wiring (RXD2, TXD2, SERVO_PIN, CONTROL_PIN).
4. Connect ESP32 and select board & port.
5. Upload the sketch.
6. Open Serial Monitor at 115200 to view debug output.

## Troubleshooting
- No frames:
  - Verify HLK-LD2450 wiring (TX->GPIO4, RX->GPIO2) and power.
  - Confirm mySerial baud (256000) matches sensor; try alternative baud rates if unsure.
- Garbled output:
  - Check logic level compatibility (5V vs 3.3V) and add level shifting if needed.
- Servo not moving:
  - Check servo power, ground, SERVO_PIN wiring, and CONTROL_PIN state (LOW prevents rotation).
- Targets flicker:
  - Increase TARGET_TIMEOUT or verify sensor placement / environmental conditions.

## Customization tips
- Tune SERVO_* angle values to match your physical mounting.
- Adjust X thresholds (-150/+150 and -500/+500) for geometry.
- Change TARGET_TIMEOUT or ROTATION_INTERVAL to suit responsiveness/latency trade-offs.

## License
This repository includes an MIT license. See LICENSE.

## Credits
Author: Dinusha-Ekanayake
