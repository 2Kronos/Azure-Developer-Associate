## Simulation 1: Power Management System (Corrected)

### Components and Pin Configuration:

**Power Input/Communication:**
- IR Receiver (TSOP38238) - **Pin 3** (Digital Input)
- Potentiometer (Power level simulator) - **Pin A0** (Analog Input)

**Visual Feedback:**
- **Serial Monitor** will be used to display the current power level (0-3)

**Additional Components:**
- Potentiometer (for manual power level simulation)
- Breadboard and jumper wires

### Circuit Layout:
```
Arduino Uno Connections:
----------------------
Digital Pin 3  → IR Receiver Data Pin
Analog Pin A0  → Potentiometer Middle Pin (Power Simulator)

Potentiometer:
- Left Pin → 5V
- Right Pin → GND
- Middle Pin → A0

IR Receiver:
- VCC → 5V
- GND → GND
- Data → Pin 3
```

### Tinkercad Setup Instructions:
1. Place Arduino Uno on the workspace
2. Add IR receiver and connect to Pin 3
3. Add potentiometer and connect to Pin A0
4. **No LEDs are needed for power indicators** - use Serial Monitor instead

### How It Will Work:
1. You will manually adjust the potentiometer to simulate different power levels (0-3)
2. The Arduino will read this value from Pin A0
3. The Serial Monitor will display:
   - "Power Level: 0 - Batteries Charging"
   - "Power Level: 1 - Low Power Available"
   - "Power Level: 2 - Medium Power Available" 
   - "Power Level: 3 - Maximum Power Available"
4. The IR receiver on Pin 3 is ready for when you implement remote control


# IR Remote FUNCTIONALITY

## Primary Function of the IR Remote

The IR remote **simulates the solar inverter** sending infrared signals to communicate how much unused electrical energy is available.

### What You Must Do With the Remote:

1. **Manually send power level values (0-3)** by pressing buttons on the remote during the simulation.
    
2. **Simulate changing solar conditions** by changing the power level value during different washing cycles.
    

### What the Signals Mean (Value → Meaning):

- **Value 0**: Batteries are charging/flat → **No extra power available**
    
- **Value 1**: **Small amount** of electricity available
    
- **Value 2**: **Moderate amount** of electricity available
    
- **Value 3**: **Maximum** electricity available (solar panels producing excess power)
    

### How It Works in Practice:

1. Your Arduino code will call the `checkPower(requiredPowerLevel, pinNum)` function
    
2. This function reads the current available power from the IR receiver (Pin 3)
    
3. **You press buttons on the remote** (e.g., buttons 0, 1, 2, 3) to send signals
    
4. The washing machine should:
    
    - **Pause** if available power is **insufficient** for the current cycle
        
    - **Resume** when enough power becomes available
        

### Example Scenario:

- **Washing cycle** requires power level 2
    
- You send **value 1** from the remote (insufficient power)
    
- Washing machine **pauses** and displays "Waiting for power" on Serial Monitor
    
- You later send **value 3** from the remote (sufficient power)
    
- Washing machine **resumes** the washing cycle
    

### Key Implementation Details:

- The IR receiver is connected to **Pin 3**
    
- You need to **map buttons** on your Tinkercad remote to values 0, 1, 2, 3
    
- The remote gives you manual control to test how your washing machine responds to **changing energy availability**
    

The remote is essential for testing the energy-saving functionality of your solar-powered washing machine design.


