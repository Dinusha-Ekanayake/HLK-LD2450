# HLK-LD2450

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Compact reference and quick-start guide for working with the HLK-LD2450 Wi‑Fi module.

## Summary

This repository collects wiring notes, example sketches, and troubleshooting tips to help you integrate the HLK-LD2450 module into microcontroller and embedded projects. The HLK-LD2450 is typically used as a UART/AT-command driven Wi‑Fi client module — always verify the exact variant and firmware of your unit.

> WARNING: Confirm voltage and pinout with the module datasheet before powering or wiring. Many modules require a regulated 3.3 V supply and are not 5 V tolerant.

## Features

- Small form factor Wi‑Fi module
- UART (AT-command) interface (typical)
- Low power operation
- Commonly used in IoT and embedded projects

## Repository layout

- hardware/ — wiring diagrams, pinouts, photos (if provided)
- examples/ — example Arduino sketches and terminal scripts
- docs/ — datasheets, firmware notes, configuration tips
- README.md — this file

## Quick Start

1. Power: supply a stable 3.3 V regulator capable of providing peak current (check datasheet; many Wi‑Fi modules need several hundred mA when transmitting).
2. Ground: connect module GND to host GND.
3. UART: connect host TX -> module RX and host RX -> module TX. Use a level shifter if your MCU's UART is 5 V.
4. Boot/EN/RESET: tie enable pins per datasheet (some modules require pull-ups/pull-downs for normal boot).
5. Open serial terminal with the module's baud rate (common defaults: 9600, 115200). 8N1.

Try:
```text
AT
```
Expected response: `OK` (depends on firmware).

## Common AT commands (example; actual commands depend on firmware)

- AT — basic check
- AT+GMR — firmware version
- AT+CWMODE? / AT+CWMODE=1 — get/set Wi‑Fi mode (station/softAP)
- AT+CWJAP="SSID","PASSWORD" — join Wi‑Fi network
- AT+CIFSR — get IP address

Always confirm the exact command set for your firmware version.

## Wiring and power (reference)

- VCC: 3.3 V regulated power supply (not 5 V)
- GND: common ground with host
- TXD: module serial transmit (to host RX)
- RXD: module serial receive (to host TX)
- EN/CH_PD: enable pin (check datasheet; typically high to enable)
- RST: module reset (active low on many modules)

Tips:
- Use decoupling capacitors and keep power traces short.
- If the module reboots or fails to connect, check that the regulator can supply inrush/peak currents.
- If your MCU uses 5 V logic, use a level shifter on RX/TX lines.

## Example: Arduino (HardwareSerial or SoftwareSerial)

HardwareSerial is recommended when available.

Arduino sketch snippet (conceptual):
```cpp
// Example: forward Serial <-> Serial1 (adjust pins/ports per board)
void setup() {
  Serial.begin(115200);    // PC terminal
  Serial1.begin(115200);   // HLK-LD2450 (hardware UART if available)
  Serial.println("HLK-LD2450 serial bridge ready");
}

void loop() {
  if (Serial1.available()) Serial.write(Serial1.read());
  if (Serial.available()) Serial1.write(Serial.read());
}
```

Adjust baud rates for your module. If using SoftwareSerial, be aware of reduced reliability at higher baud rates.

## Example: Linux / macOS (USB‑TTL)

Using screen:
```bash
screen /dev/ttyUSB0 115200
# Press Enter and type AT
```

Using echo + cat:
```bash
stty -F /dev/ttyUSB0 115200 cs8 -cstopb -ixon -ixoff -parenb
echo -e "AT\r\n" > /dev/ttyUSB0
cat /dev/ttyUSB0
```

## Troubleshooting

- No response:
  - Check power (3.3 V regulator, decoupling).
  - Confirm wiring TX↔RX and common GND.
  - Verify correct serial baud and serial settings (8N1).
- Garbage characters:
  - Wrong baud rate or logic level mismatch (5 V vs 3.3 V).
- Module refuses to join AP:
  - Wrong SSID/password, wrong Wi‑Fi mode, or incompatible firmware.
- Module resets under load:
  - Power supply unable to provide peak current; add larger decoupling capacitor or a stronger regulator.
- Command returns ERROR:
  - Verify correct AT command syntax for your firmware version.

## Firmware & Safety Notes

- Different HLK-LD2450 units may ship with different firmwares/AT command sets. Check firmware version with AT+GMR and consult the correct command reference.
- Do not apply 5 V to module pins that are 3.3 V only.
- Use ESD precautions when handling modules.

## Contributing

Contributions, corrections, and examples are welcome. Please open issues for bugs or feature requests. For code contributions, fork the repo and submit a pull request with a clear description and testing steps.

When adding examples, include:
- Hardware used
- Exact wiring
- Software/firmware versions
- How to reproduce results

## Resources

- Vendor datasheet (check the module label or vendor website)
- Firmware/AT-command reference for the shipped firmware version
- Community forum posts and example projects (search for HLK-LD2450 usage examples)

## License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

## Contact

Repository owner: @Dinusha-Ekanayake
