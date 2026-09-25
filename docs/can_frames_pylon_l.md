# Pylontech Compatibility CAN Frames Documentation (Dyness-L Implementation)

This document describes the standard 11-bit CAN frames (`Standard ID`) used by the **Dyness-L** Master BMS to emulate the Pylontech communication protocol. These frames are broadcast to hybrid inverters to regulate charge/discharge limits, system states, and alarms.

Unlike internal cascade frames, most data parameters in this group follow the standard **Little-Endian** byte ordering (least significant byte first), except where specified as simple single-byte flags or signed arrays.

---

## 📟 Comprehensive Frame Registry (11-bit Standard ID)

### Frame `0x351` — Charge/Discharge Operational Limits
This is the primary control frame read by solar inverters to dynamically throttle performance based on chemistry status.
- **Byte 0-1:** Overvoltage Protection Limit (Scale: `0.1 V`, Little-Endian, e.g., `535` = 53.5 V)
- **Byte 2-3:** Max Allowed Charging Current (Scale: `0.1 A`, Signed Little-Endian, e.g., `400` = 40.0 A)
- **Byte 4-5:** Max Allowed Discharging Current (Scale: `0.1 A`, Signed Little-Endian, e.g., `500` = 50.0 A)

### Frame `0x355` — State of Charge (SoC) & State of Health (SoH)
- **Byte 0:** Current Battery SoC Capacity (Scale: `1 %`, raw integer value, e.g., `80` = 80%)
- **Byte 1:** Reserved
- **Byte 2-3:** Current Stack SoH Health (Scale: `1 %`, Little-Endian, e.g., `100` = 100%)

### Frame `0x356` — Dynamic Stack Pack Parameters
Real-time physical values measured across the entire combined battery stack.
- **Byte 0-1:** Total Combined Stack Voltage (Scale: `0.01 V`, Signed Little-Endian, e.g., `5015` = 50.15 V)
- **Byte 2-3:** Total Stack Active Current (Scale: `0.1 A`, Signed Little-Endian, negative values indicate discharge, e.g., `-12` = -1.2 A)
- **Byte 4-5:** Average Battery Stack Temperature (Scale: `0.1 °C`, Signed Little-Endian, e.g., `182` = 18.2 °C)

### Frame `0x359` — System Alarms, Protection Bits & Total Cycles
- **Byte 0:** Alarm Flags Bit 1 (Bit `0x02` -> Cell Overvoltage Alarm, Bit `0x04` -> Cell Undervoltage Alarm)
- **Byte 1:** Alarm Flags Bit 2 (Bit `0x08` -> Hardware System Error Active)
- **Byte 2-3:** System Protection Bitmask (Used for warnings and sensor error flags)
- **Byte 4-5:** **Aggregated Stack Cycles** (Big-Endian integer read by the inverter, e.g., `258 cycles`)

### Frame `0x35C` — Operational Mode Control Flags
Direct digital commands enabling or disabling inverter hardware channels.
- **Byte 0:** Binary Control Mask:
  - `0x80` (`10000000`) -> Charge Enable (Inverter allowed to charge the pack)
  - `0x40` (`01000000`) -> Discharge Enable (Inverter allowed to pull power from the pack)

### Frame `0x35E` — Manufacturer Hardware Identifier
- **Data Payload:** ASCII encoded string up to 8 characters.
- **Value:** Injected text translates strictly to `'DYNESS-L'` to guarantee plug-and-play identification during inverter handshake.

### Frame `0x70D` — Stack Identity & Nominal Constants
- **Byte 4-5:** Dynamic Recommended Charging Current (Scale: `1 A`, Little-Endian, e.g., `48` = 48 A)
- **Byte 6-7:** Total Nominal Capacity Registry (Scale: `1 Ah`, Little-Endian, e.g., `100` = 100 Ah)

---

## 🛑 Administrative Interface Frames

The following 11-bit frames are part of the standard Pylontech network matrix but are utilized purely for keep-alive timeouts, background pinging, and clock synch. They contain no real-time telemetry variables and are bypassed by the ESPHome parser to maintain silence in the logs:

- **`0x305`:** Manufacturer Keep-Alive / Handshake Ping
- **`0x30F`:** BMS Hardware Pulse (Heartbeat Loop)
- **`0x35A`:** Generic Warning Registry Frame
- **`0x35F`:** Compatibility Matrix Layer Frame
- **`0x36D`:** Diagnostic Stream Allocation Frame
- **`0x370` / `0x371`:** Extended ASCII Device Type Labeling Stream (`"DYNESS-L Battery"`)
- **`0x63E` / `0x63F` / `0x78F` / `0x7C7`:** Manufacturer Diagnostic Command Mapping
