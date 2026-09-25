# dyness-l-cascade-esphome
ESPHome configuration for reading Dyness-L battery Master/Slave cascade data via ESP32 TWAI/CAN bus. Decodes both standard Pylontech inverter compatibility frames and 29-bit extended engineering telemetry. Local monitoring without cloud.
------------------------------
## docs/can_frames.md## Dyness-L Internal Cascade CAN Frames Documentation
This document describes the 29-bit extended CAN frames (Extended ID) used for internal communication between Master (Unit #1) and Slave (Unit #2) BMS boards in a Dyness-L battery stack.
All engineering telemetry values in this group are encoded using strict Big-Endian byte ordering (most significant byte first).
------------------------------
## Master BMS Node 1 Frames (0x18F211xx)## Frame 0x18F21121 — Master Pack Status & Average Cell

* Byte 0-1: Total Pack Voltage (Scale: 0.01 V, e.g., 5009 = 50.09 V)
* Byte 2-3: Hardcoded Upper Cell Safety Limit (Scale: 1 mV, constant 4000 = 4.000 V)
* Byte 4-5: True Average Cell Voltage (Scale: 1 mV, e.g., 3334 = 3.334 V)
* Byte 6-7: Balancing Hardware Current Limit (Scale: 0.1 mA, constant 250 = 25.0 mA)

## Frame 0x18F21122 — Master Cell Extremums & Protection

* Byte 0-1: Maximum Single Cell Voltage (Scale: 1 mV, e.g., 3348 = 3.348 V)
* Byte 2-3: Cell Streaming Multiplexer Status Flag (HEX constant 0x0E0D in idle state)
* Byte 4-5: Low-end Voltage Protection Floor Threshold (Scale: 1 mV, calculated dynamically by BMS)
* Byte 6-7: Reserved / Static padding

## Frame 0x18F21123 — Master Hardware Odometer & Health

* Byte 0-1: Real Hardware Battery Cycles (Extracted directly from MCU EEPROM, e.g., 584)
* Byte 2-3: BMS Board Hardware Revision (e.g., 258 = Firmware v1.2)
* Byte 4-5: Deep Discharge Protection Counter (Stored event counts)
* Byte 6-7: Hardware State of Health (SoH, HEX 0x64 = 100%)

## Frame 0x18F21124 — Master Relay Control Flags

* Byte 2: Main Switch Relay Bitmask (Constant 0x03 / 00000011 -> Charge MOSFET Open, Discharge MOSFET Open)
* Other Bytes: Logic status registers

## Frame 0x18F21116 — Master Topology & Microclock Timing

* Byte 0-1: MCU Core Clock Synchronization Tick (Used to prevent CAN collisions in каскад)
* Byte 2-3: Internal MCU Silicon Die Temperature (Raw ADC code)
* Byte 4-5: Topology Mapping Mask (Constant 192 / 0x00C0 -> Master Node Flag Active)

------------------------------
## Slave BMS Node 2 Frames (0x18F212xx)## Frame 0x18F21221 — Slave Pack Status & Average Cell

* Byte 0-1: Total Pack Voltage (Scale: 0.01 V, e.g., 5021 = 50.21 V)
* Byte 2-3: Hardcoded Upper Cell Safety Limit (Constant 4000 = 4.000 V)
* Byte 4-5: True Average Cell Voltage (Scale: 1 mV, e.g., 3353 = 3.353 V)
* Byte 6-7: Balancing Hardware Current Limit (Constant 250 = 25.0 mA)

## Frame 0x18F21222 — Slave Cell Extremums & Protection

* Byte 0-1: Maximum Single Cell Voltage (Scale: 1 mV, e.g., 3355 = 3.355 V)
* Byte 2-3: Cell Streaming Multiplexer Status Flag (HEX constant 0x0E0D in idle state)
* Byte 4-5: Low-end Voltage Protection Floor Threshold (Scale: 1 mV, calculated dynamically by BMS)
* Byte 6-7: Reserved / Static padding

## Frame 0x18F21223 — Slave Hardware Odometer & Health

* Byte 0-1: Real Hardware Battery Cycles (Extracted directly from MCU EEPROM, e.g., 585)
* Byte 2-3: BMS Board Hardware Revision (e.g., 258 = Firmware v1.2)
* Byte 4-5: Deep Discharge Protection Counter (Stored event counts)
* Byte 6-7: Hardware State of Health (SoH, HEX 0x64 = 100%)

## Frame 0x18F21224 — Slave Relay Control Flags

* Byte 2: Main Switch Relay Bitmask (Constant 0x03 -> Operational Mode Normal)
* Other Bytes: Logic status registers

## Frame 0x18F21216 — Slave Topology & Microclock Timing

* Byte 0-1: MCU Core Clock Synchronization Tick (Aligned with Master Node)
* Byte 2-3: Internal MCU Silicon Die Temperature (Raw ADC code)
* Byte 4-5: Topology Mapping Mask (Constant 283 / 0x011B -> Slave Node #1 Registry Valid)

------------------------------
## Global Cascade Synchronization## Frame 0x18F20102 — Cascade Heartbeat Loop

* Data Payload: 00 00 00 00 00 00 00 00
* Interval: Transmitted strictly every 1000ms. If this packet is missing due to a cable disconnect between Master and Slave units, the entire stack triggers an emergency cutoff to prevent inter-pack balancing overcurrent.

------------------------------

