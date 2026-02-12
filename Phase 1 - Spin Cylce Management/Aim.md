## **Objective**

**To implement and validate the core drum motor control system for a washing machine simulator using four LEDs to emulate stepper motor functionality, with bidirectional rotation patterns that replicate real-world washing machine agitation and spinning cycles.**

## **Code Description**

### **Core Functionality**
This code establishes the foundational drum motor simulation system using four LEDs connected to Arduino pins 4-7. The system emulates a stepper motor's behavior without using external libraries, implementing a custom 8-step sequencing algorithm to control "drum" rotation in both clockwise and counter-clockwise directions.

### **Key Components:**

#### **1. Motor Control Engine**
- **`clockwiseRevolution(int stepDelay)`**: Implements an 8-step forward rotation sequence
- **`counterClockwiseRevolution(int stepDelay)`**: Mirrors the 8-step sequence for reverse rotation
- **Step Delay Parameter**: Controls rotation speed (400ms for wash/rinse, 100ms for spin)

#### **2. Washing Cycle Simulation**
- **`wash_rinse()`**: Replicates washing machine agitation with 3 cycles of alternating directions
  - Each cycle: 3 clockwise + 3 counter-clockwise revolutions
  - Includes 1-second pauses between direction changes
- **`spin()`**: Simulates final water extraction with 20 high-speed unidirectional rotations

#### **3. User Interface & State Management**
- **Button Debouncing**: Professional-grade input handling with 50ms debounce delay
- **State Tracking**: `spinningActive` flag prevents overlapping operations
- **Revolution Counting**: Real-time progress tracking via Serial Monitor

### **Technical Implementation Details:**

```cpp
// LED Pin Mapping (Simulating Motor Phases)
LED1_PIN = 4    // Phase A
LED2_PIN = 7    // Phase B (Note: intentionally swapped with LED3 for proper sequence)
LED3_PIN = 5    // Phase C
LED4_PIN = 6    // Phase D

// 8-Step Rotation Pattern:
// Each step activates specific LED pairs to simulate electromagnetic phase changes
// Clockwise: AB → B → BC → C → CD → D → AD → A
// Counter-clockwise: AD → D → CD → C → BC → B → AB → A
```

### **Operational Workflow:**
1. **Initialization**: All LEDs off, Serial communication established
2. **User Activation**: Debounced button press triggers `startWashingMachine()`
3. **Wash Cycle**: Alternating bidirectional agitation (9 revolutions each direction total)
4. **Spin Cycle**: High-speed unidirectional water extraction (20 revolutions)
5. **Reset**: System ready for next cycle

### **Design Significance:**
- **Library-Free Implementation**: Demonstrates understanding of stepper motor fundamentals
- **Variable Speed Control**: Different `stepDelay` values simulate realistic washing machine behavior
- **Modular Architecture**: Clean separation between motor control and cycle orchestration
- **Real-Time Feedback**: Serial output provides visibility into system operation

This foundational module successfully replicates the mechanical behavior of a washing machine drum, providing the essential motor control layer upon which power management and water control systems can be integrated.