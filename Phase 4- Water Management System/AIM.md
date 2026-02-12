
# **Simulation 2: Water Management System**

## **Purpose of Simulation 2:**
To add and test the **water control components** of the washing machine while integrating with the power management system from Simulation 1. This simulation focuses on water filling, level sensing, and draining operations.

---

## **Components and Pin Configuration:**

### **New Components Added:**
- **Potentiometer (Water Level Sensor)** - **Pin A5** (Analog Input)
- **Blue LED (Water Inlet Valve)** - **Pin 8** (Digital Output)  
- **DC Motor (Water Pump)** - **Pin 9** (Digital Output)
- **Push Button (Start/Pause)** - **Pin 10** (Digital Input)

### **Carried Over from Simulation 1:**
- IR Receiver - **Pin 3**
- Potentiometer (Power Simulator) - **Pin A0**
- All power management logic

---

## **Circuit Layout:**
```
Arduino Uno Connections:
----------------------
FROM SIMULATION 1:
Digital Pin 3  → IR Receiver Data Pin
Analog Pin A0  → Potentiometer (Power Simulator)

NEW FOR SIMULATION 2:
Analog Pin A5  → Potentiometer (Water Level Sensor) Middle Pin
Digital Pin 8  → Blue LED (Valve) + 220Ω resistor → GND
Digital Pin 9  → DC Motor (Water Pump)
Digital Pin 10 → Push Button → GND (with 10kΩ pull-down resistor)

Water Level Sensor:
- Left Pin → 5V
- Right Pin → GND
- Middle Pin → A5

Start Button:
- One side → Pin 10
- Other side → 5V (through 10kΩ resistor to GND for pull-down)
```

---

## **Theory Notes:**

### **1. Water Level Sensing:**
- The potentiometer on A5 simulates a water level sensor
- **Values mapped:** 0-1023 → 0-100% water level
- **Critical thresholds:** 
  - **80%:** Stop filling (drum full)
  - **10%:** Stop draining (drum empty)
  - **0-10%:** Drum empty state

### **2. Water Valve Control:**
- Blue LED on Pin 8 represents the water inlet valve
- **LED ON** = Valve open, water flowing in
- **LED OFF** = Valve closed, water stopped

### **3. Water Pump Control:**
- DC Motor on Pin 9 represents the drain pump
- **Motor ON** = Pump active, water draining out
- **Motor OFF** = Pump inactive

### **4. Start/Pause Button:**
- Momentary push button on Pin 10
- Triggers the washing machine sequence
- Should work with interrupt-like behavior for responsiveness

---

## **Expected Serial Output:**
```
Solar Powered Washing Machine - Simulation 2
Water Management System Ready
-------------------------------
Start button pressed!
Checking power for filling: 1 | Available: 3 - SUFFICIENT POWER
Filling drum... Current water level: 25%
Filling drum... Current water level: 50% 
Filling drum... Current water level: 75%
Filling drum... Current water level: 80% - Drum full!
Valve closed. Ready for next cycle.
-------------------------------
IR Signal Received: Power Level 0: Battery charging
Draining required: 1 | Available: 0 - INSUFFICIENT POWER
Waiting for power to drain...
IR Signal Received: Power Level 2: Moderate Power
Draining... Current water level: 60%
Draining... Current water level: 30%
Draining... Current water level: 10% - Drum empty!
Pump stopped. Drain complete.
-------------------------------
```

---
