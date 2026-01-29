# Component Logical Grouping

This document describes the logical grouping of components in the RFID-LoRa PCB design.

## Overview

The circuit consists of 5 functional blocks:

1. **Power Supply** - Battery input and 5V boost conversion
2. **Power Switch** - Software-controlled power for RDM6300
3. **Level Shifter** - 3.3V/5V UART translation
4. **Decoupling** - Power rail filtering
5. **Modules/Connectors** - MCU, LoRa, and RDM6300 interface

---

## 1. Power Supply (Battery + Boost Converter)

Converts 3.7V LiPo to 5V for the RDM6300 RFID reader.

| Ref | Part | Value | Function |
|-----|------|-------|----------|
| U1 | JST-PH 2-pin | - | LiPo battery connector |
| U2 | MT3608 | - | Boost converter IC (3.7V → 5V) |
| L1 | Inductor | 22µH | Boost converter energy storage |
| D1 | SS34 | - | Schottky rectifier diode |
| C1 | Capacitor | 22µF | Input decoupling |
| C2 | Capacitor | 22µF | Output filtering |
| C3 | Capacitor | 22µF | Output filtering |
| C5 | Capacitor | 47µF | 5V rail bulk capacitor |

### Circuit Flow
```
LiPo (3.7V) → U1 → C1 → U2+L1+D1 → C2/C3/C5 → 5V Rail
```

---

## 2. Power Switch (RDM6300 Power Control)

High-side P-FET switch controlled by MCU GPIO6. Allows software control of RDM6300 power for energy saving.

| Ref | Part | Value | Function |
|-----|------|-------|----------|
| Q2 | AO3401A | - | P-FET high-side switch |
| Q1 | 2N7002 | - | N-FET gate driver |
| R1 | Resistor | 220kΩ | P-FET gate pull-up |
| R2 | Resistor | 30kΩ | Voltage divider |
| R3 | Resistor | 1kΩ | N-FET gate resistor |

### Operation
- **GPIO6 LOW (default)**: Q1 OFF → Q2 gate pulled HIGH → Q2 OFF → RDM6300 unpowered
- **GPIO6 HIGH**: Q1 ON → Q2 gate pulled LOW → Q2 ON → RDM6300 powered

---

## 3. UART Level Shifter (3.3V ↔ 5V)

Bidirectional level translation between Pro Micro (3.3V) and RDM6300 (5V) using BSS138 MOSFETs.

| Ref | Part | Value | Function |
|-----|------|-------|----------|
| Q3 | BSS138 | - | TX channel shifter |
| Q4 | BSS138 | - | RX channel shifter |
| R4 | Resistor | 10kΩ | TX low-voltage pull-up |
| R5 | Resistor | 10kΩ | TX high-voltage pull-up |
| R6 | Resistor | 10kΩ | RX low-voltage pull-up |
| R7 | Resistor | 10kΩ | RX high-voltage pull-up |
| R8 | Resistor | 10kΩ | Additional pull-up |
| R9 | Resistor | 10kΩ | Additional pull-up |

### Circuit Topology
Each channel uses one BSS138 N-FET with pull-up resistors on both voltage rails:
```
3.3V --[R_LV]--+--[Q]--+--[R_HV]-- 5V
               |       |
              LV      HV
             side    side
```

---

## 4. Decoupling / Power Filtering

Local decoupling capacitors for noise suppression.

| Ref | Part | Value | Function |
|-----|------|-------|----------|
| C4 | Capacitor | 100nF | 3.3V rail high-frequency decoupling |
| C6 | Capacitor | 10µF | Switched 5V rail decoupling (near RDM6300) |

---

## 5. Modules and Connectors

Main functional modules and external interfaces.

| Ref | Part | Function |
|-----|------|----------|
| U4 | Pro Micro nRF52840 | Main MCU (3.3V logic, BLE capable) |
| U3 | Wio SX1262 | LoRa module for Meshtastic mesh network |
| H1 | 4-pin header | RDM6300 connection (VCC, GND, TX, RX) |

### Pin Assignments

**H1 - RDM6300 Header:**
| Pin | Signal | Description |
|-----|--------|-------------|
| 1 | VCC | 5V switched power |
| 2 | GND | Ground |
| 3 | TX | RDM6300 data output (→ MCU RX via level shifter) |
| 4 | RX | RDM6300 data input (← MCU TX via level shifter) |

**U3 - WioSX1262 LoRa Module:**
| Signal | MCU Pin | Description |
|--------|---------|-------------|
| MISO | SPI MISO | SPI data from module |
| MOSI | SPI MOSI | SPI data to module |
| SCK | SPI SCK | SPI clock |
| CS | GPIO10 | Chip select |
| RST | GPIO9 | Reset |
| BUSY | GPIO8 | Busy indicator |
| DIO1 | GPIO7 | Interrupt |

---

## PCB Layout Suggestion

```
┌─────────────────────────────────────────────────┐
│  POWER SUPPLY              │  RDM6300 HEADER   │
│  U1 U2 L1 D1               │       H1          │
│  C1 C5 C2 C3               │                   │
├────────────────────────────┼───────────────────┤
│  POWER SWITCH              │  LORA MODULE      │
│  Q1 Q2 R1 R2 R3            │       U3          │
├────────────────────────────┤       C6          │
│  LEVEL SHIFTER             │                   │
│  Q3 Q4 R4-R9 C4            │                   │
├────────────────────────────┴───────────────────┤
│              PRO MICRO (U4)                    │
│         (largest component, center)            │
└────────────────────────────────────────────────┘
```

### Placement Guidelines

1. **Power Supply** - Place near battery connector, keep boost converter loop tight (U2-L1-D1-C2)
2. **Power Switch** - Place between boost output and RDM6300 header
3. **Level Shifter** - Place between MCU UART pins and RDM6300 header
4. **Decoupling caps** - Place as close as possible to their respective ICs
5. **LoRa module** - Keep away from switching noise, antenna area clear
