Hardware Connection & Pinout Documentation
This document describes the physical wiring requirements to connect an ESP32 development board and a CAN-bus transceiver (such as the SN65HVD230) to the Dyness-L battery stack diagnostic interface.
The setup utilizes the built-in ESP32 hardware CAN controller via the ESPHome esp32_can component, bypassing the need for external SPI modules like the MCP2515.
1. Transceiver to ESP32 Wiring
The SN65HVD230 transceiver operates on 3.3V logic, making it perfectly safe for direct connection to ESP32 GPIO pins without level shifters.
SN65HVD230 Pin	ESP32 Pin	Description
VCC	3V3 / 3.3V	Power Supply (3.3V)
GND	GND	Ground Reference
CTX (TXD)	GPIO14	CAN Transmit Data
CRX (RXD)	GPIO13	CAN Receive Data
Note: You can reassign CTX and CRX to other available hardware pins within your ESPHome configuration file if needed.
2. Dyness-L RJ45 CAN Port Pinout
Dyness-L battery modules use standard RJ45 ports for inverter communication and Master/Slave cascade tracking. The primary CAN communication lines are allocated on pins 4 and 5 matching the standard industrial base.
RJ45 Connector Mapping (T-568B Standard Reference)
RJ45 Pin	Color (T-568B)	Signal Name	Connection Target
Pin 1	Blue-White	RS485-B	Not used for CAN sniffing
Pin 2	Blue	RS485-A	Not used for CAN sniffing
Pin 3	Green-White	Ground	Not strictly required (Optional GND)
Pin 4	Blue	CAN_H	Connect to SN65HVD230 CANH terminal
Pin 5	Blue-White	CAN_L	Connect to SN65HVD230 CANL terminal
Pin 6	Green	Ground	System Ground
Pin 7	Brown-White	Reserved	Do not connect
Pin 8	Brown	Reserved	Do not connect
3. Bus Termination Requirements
Reliable high-speed CAN communication requires proper bus termination to prevent signal reflections over long runs or noisy environments.
• 120 Ohm Resistor: The CAN bus network must be terminated at both logical ends with a 120-ohm resistor between CAN_H and CAN_L.
• Hardware Boards: Most cheap SN65HVD230 breakout boards come with a built-in SMD 120-ohm resistor pre-soldered on the PCB (usually labeled as R1 or R2).
• Verification: Ensure that your sniffer module or the internal circuitry of the connected hybrid inverter provides the proper termination. If communication errors or packet drops occur under heavy electrical load, verify the differential resistance between CAN_H and CAN_L with a multimeter while the system is powered off (it should read approximately 60 Ohms with both ends terminated).
