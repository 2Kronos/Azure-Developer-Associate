# README.md

## Solar-Powered Washing Machine Control System

An Arduino-based control system for a solar-powered washing machine that adapts its operational cycles to variable energy availability.

---

## Project Overview

This project implements an adaptive control system for a washing machine that operates under variable solar power conditions. The system uses IR communication to receive power availability data (levels 0-3), allowing the washing machine to pause operations when insufficient power is available. The system demonstrates complete washing cycles including filling, washing, rinsing, draining, and spinning with proper water level management and drum motor control.

The system was developed incrementally across five phases:
- Phase 1: Spin Cycle Management
- Phase 2: Power Management Integration
- Phase 3: Spin Cycle + Power Management
- Phase 4: Water Management System
- Phase 5: Complete System Integration

---

## Component List

### Simulation Components (Tinkercad)

| Component | Quantity | Pin Connection | Purpose |
|-----------|----------|----------------|---------|
| Arduino Uno R3 | 1 | - | Main controller |
| Green LED | 4 | Pins 4,5,6,7 | Drum motor phase simulation |
| Blue LED | 1 | Pin 8 | Water inlet valve indicator |
| IR Receiver | 1 | Pin 3 | Receives power level signals |
| DC Motor | 1 | Pin 9 | Water pump simulation |
| Push Button | 1 | Pin 10 | Start/Pause cycle |
| Potentiometer | 1 | Pin A5 | Water level sensor simulation |
| Resistor (220 Ohm) | 5 | - | Current limiting for LEDs |
| Resistor (10k Ohm) | 1 | - | Pull-down for button |
| Breadboard | 1 | - | Prototyping |
| Jumper Wires | As needed | - | Connections |

### Physical Circuit Components

| Component | Quantity | Pin/Connection | Purpose | Power Requirement |
|-----------|----------|----------------|---------|-------------------|
| Arduino Uno | 1 | - | Main controller | 5V USB |
| Stepper Motor (28BYJ-48) | 1 | Pins 4-7 via ULN2003 | Drum motor | 5V-12V external |
| ULN2003 Driver Board | 1 | Pins 4,5,6,7 | Stepper motor control | 5V-12V external |
| H-Bridge (L298N) | 1 | Pin 9 | Pump motor control | 12V external |
| DC Motor | 1 | Via L298N | Water pump | 12V external |
| IR Receiver (TSOP38238) | 1 | Pin 3 | Power signal reception | 5V |
| Blue LED | 1 | Pin 8 via 220 Ohm | Valve indicator | 5V |
| Push Button | 1 | Pin 10 | Start/Pause | 5V |
| Potentiometer (10k Ohm) | 1 | Pin A5 | Water level sensor | 5V |
| Resistor (220 Ohm) | 5 | - | LED current limiting | - |
| Resistor (10k Ohm) | 1 | - | Pull-down resistor | - |
| Breadboard | 1 | - | Prototyping | - |
| 12V Power Supply | 1 | L298N & ULN2003 | Motor power | 12V DC |
| Jumper Wires | Many | - | Connections | - |
| USB Cable | 1 | Arduino | Programming & power | 5V |

---

## Circuit Images

### Simulated Circuit (Tinkercad)

- Circuit
<img width="1284" height="780" alt="image" src="https://github.com/user-attachments/assets/298b72e0-8254-4c58-a31e-514d805a67df" />


- Schematic view of circuit
<img width="1033" height="791" alt="image" src="https://github.com/user-attachments/assets/cf17e9bc-9312-40ad-a8c4-de8e4588c8d5" />


**Simulation Features:**
- 4 Green LEDs simulating stepper motor phases
- IR receiver for power level communication
- Blue LED for water valve indication
- DC motor for pump simulation
- Push button with debouncing
- Potentiometer for water level sensing
- Complete cycle execution with power verification

---

### Physical Circuit Implementation

![Physical implentation](https://github.com/user-attachments/assets/bd3ffb68-312b-4818-8d0e-a18f7c2a6896)

![3](https://github.com/user-attachments/assets/9ff48bdb-d0b0-4aa7-9dc7-493bdb6eb336)

![2](https://github.com/user-attachments/assets/d1ea34fd-c9d0-4699-a9ad-e25c2f33fd96)

![1](https://github.com/user-attachments/assets/726d43ce-64d1-49ab-b456-963670c40712)

- Schematic of physical circuit (Fritzing)

<img width="690" height="744" alt="image" src="https://github.com/user-attachments/assets/35cdd249-eff5-4a98-93d8-11f5973a82a1" />






**Physical Circuit Features:**
- 28BYJ-48 Stepper motor with ULN2003 driver for drum rotation
- L298N H-bridge controlling DC motor pump
- External 12V power supply for motors
- TSOP38238 IR receiver for power signals
- Potentiometer-based water level sensing

---

## Pin Mapping Reference

| Function | Pin | Component | Notes |
|----------|-----|----------|-------|
| IR Receiver | 3 | TSOP38238 | Signal pin |
| LED1 (Phase A) | 4 | Green LED / ULN2003 IN1 | Drum phase A |
| LED3 (Phase C) | 5 | Green LED / ULN2003 IN2 | Drum phase C |
| LED4 (Phase D) | 6 | Green LED / ULN2003 IN3 | Drum phase D |
| LED2 (Phase B) | 7 | Green LED / ULN2003 IN4 | Drum phase B |
| Valve LED | 8 | Blue LED | Water inlet indicator |
| Pump | 9 | DC Motor via L298N | Water drain |
| Start Button | 10 | Push Button | Cycle start |
| Water Sensor | A5 | Potentiometer | 0-100% water level |

---

## Software Requirements

- Arduino IDE
- IRremote library by Armin Joachimsmeyer

---

## Simple Operating Instructions

### Before You Start

1. Open Serial Monitor and set baud rate to 9600
2. Connect external 12V power supply for motors
3. Ensure IR remote has batteries
4. Set potentiometer to minimum position (fully counter-clockwise)

---

### Step-by-Step Operation

**Step 1: Start the Cycle**
Press the physical start button once.
- Serial Monitor displays: "START BUTTON PRESSED! Beginning washing cycle..."

**Step 2: Fill Drum for Wash**
- Read the instruction: "Please TURN the POT to INCREASE water level until it is MORE THAN 80%"
- Turn the potentiometer clockwise
- Blue LED turns ON (valve open)
- Water level percentage displays in real-time
- System automatically continues when water level exceeds 80%

**Step 3: Provide Power for Wash**
- System displays: ">>> Please press IR button 2 to continue <<<"
- Point IR remote at receiver and press button 2
- System confirms: "IR Signal Received: Power Level 2: Moderate Power"
- Wash cycle begins automatically

**Step 4: Wash Cycle**
- Green LEDs rotate clockwise 3 times
- 1 second pause
- Green LEDs rotate counter-clockwise 3 times
- Pattern repeats 3 times total
- System displays: "WASH cycle completed!"

**Step 5: Drain After Wash**
- Read: "Please TURN the POT in the OPPOSITE DIRECTION to DECREASE water level until it is LESS THAN 10%"
- Turn potentiometer counter-clockwise
- DC motor runs (pump active)
- System automatically continues when water level drops below 10%

**Step 6: Fill Drum for Rinse**
- Repeat Step 2

**Step 7: Provide Power for Rinse**
- Repeat Step 3 (press IR button 2)

**Step 8: Rinse Cycle**
- Repeat Step 4

**Step 9: Drain After Rinse**
- Repeat Step 5

**Step 10: Provide Power for Final Spin**
- System displays: ">>> Please press IR button 3 to continue <<<"
- Press IR button 3 (Maximum Power)
- System confirms: "Power Level 3: Maximum Power"

**Step 11: Final Spin Cycle**
- Green LEDs rotate rapidly clockwise only
- 20 revolutions at high speed
- System displays: "SPIN cycle completed!"

**Step 12: Cycle Complete**
- System displays: "Finished. Ready for next cycle - Press START button"
- System ready for next cycle

---

## Power Level Reference

| IR Button | Power Level | Name | Required For |
|-----------|------------|------|--------------|
| Button 0 | Level 0 | Battery charging | System idle |
| Button 1 | Level 1 | Low power | Not sufficient for operation |
| Button 2 | Level 2 | Moderate power | Wash and Rinse cycles |
| Button 3 | Level 3 | Maximum power | Final Spin cycle |

**Important:** Only press IR buttons when the system explicitly requests power.

---

## Troubleshooting

| Problem | Possible Solution |
|---------|-------------------|
| No response to start button | Check wiring on Pin 10; verify button connections |
| IR not detected | Point remote directly at receiver; hold button for 1 second; use buttons 0-3 only |
| Water level not changing | Check potentiometer wiring (5V, GND, A5); verify potentiometer is functional |
| Pump not running | Check external 12V power supply; verify L298N connections |
| LEDs not rotating | Confirm pin mapping (4,5,6,7); check LED polarity |
| Cycle will not continue | Press required IR button when prompted; check Serial Monitor for instructions |
| System unresponsive | Press Arduino reset button; restart cycle |

---

## Development Phases

| Phase | Description |
|-------|-------------|
| Phase 1 | Spin cycle management with 8-step LED sequencing and bidirectional rotation |
| Phase 2 | Power management with IR remote control and potentiometer simulation |
| Phase 3 | Integration of spin cycle and power management with power-aware drum cycles |
| Phase 4 | Water management system with level sensing, valve, and pump control |
| Phase 5 | Complete system integration with full 7-step washing sequence |

---

## Acknowledgments

- Arduino IRremote Library by Armin Joachimsmeyer
- Tinkercad Circuits simulation platform
- Fritzing Circuit simulation platform
- Embedded Systems 3 course staff

---

## License

This project is for academic purposes.
