## **Phase 3: Integration of Spin Cycle and Power Management**

### **Objective**

The primary objective of Phase 3 was to integrate the two independently developed subsystems—**Spin Cycle Management** (Phase 1) and **Power Management** (Phase 2)—into a unified control system. This integration marked the first complete prototype of my solar-adaptive washing machine, where drum operations now respond dynamically to available power levels communicated via IR remote.

Until this point, the spin cycle code operated independently with fixed timing, and the power management code merely simulated power checks without actually controlling any physical actuators. Phase 3 bridges this gap, creating a system where the washing machine **only executes energy-intensive cycles when sufficient solar power is available**.

---

### **Key Integration Functions**

#### **1. The `checkPower()` Function - Core Integration Logic**

The most critical addition to this phase was the implementation of a **blocking power verification system**. This function halts cycle execution until the user supplies the required power level via IR remote:

```cpp
void checkPower(int requiredPowerLevel, int pinNum) {
  Serial.println("=============================================");
  Serial.print(">>> POWER CHECK REQUIRED <<<");
  Serial.println();
  Serial.print("Required Power Level: ");
  Serial.println(requiredPowerLevel);
  Serial.print("Current Power Level: ");
  Serial.println(currentPowerLevel);
  
  if (currentPowerLevel < requiredPowerLevel) {
    Serial.print(">>> Please press IR button ");
    Serial.print(requiredPowerLevel);
    Serial.println(" to continue <<<");
    
    // Wait silently until power is available
    while (getCurrentPowerLevel() < requiredPowerLevel) {
      delay(100);
    }
  }
  
  Serial.print(">>> SUFFICIENT POWER (Level ");
  Serial.print(currentPowerLevel);
  Serial.println(") - Continuing cycle <<<");
  Serial.println("=============================================");
}
```

**How it works:** The function continuously polls `getCurrentPowerLevel()` in a `while` loop, effectively pausing the entire washing sequence until the user presses the correct IR button. This creates a **user-mediated power verification handshake**—the machine explicitly requests power, and the user must respond by simulating "solar availability" via remote.

---

#### **2. Modified Cycle Functions with Power Requirements**

I modified the existing cycle functions to include power checks before execution:

```cpp
void startWashingMachine() {
  spinningActive = true;
  revolutionCount = 0;
  Serial.println("START BUTTON PRESSED!");
  Serial.println("Beginning washing cycle...");
  
  // WASH CYCLE - Requires Power Level 2
  checkPower(2, LED1_PIN);  // ← Power verification before wash
  wash_rinse();
  
  delay(2000);
  
  // SPIN CYCLE - Requires Power Level 3  
  checkPower(3, LED1_PIN);  // ← Power verification before spin
  spin();
  
  spinningActive = false;
  Serial.println("Ready for next cycle - Press START button");
}
```

**Key Design Decision:** I intentionally assigned **Power Level 2** for wash/rinse cycles and **Power Level 3** for the high-speed spin cycle. This mirrors real-world washing machine behavior where the final spin extraction requires significantly more energy than gentle agitation.

---

#### **3. Continuous IR Monitoring**

Unlike Phase 2 where IR was checked periodically, Phase 3 implements **continuous background monitoring**:

```cpp
void loop() {
  // Continuously update power level - runs every cycle
  getCurrentPowerLevel();
  
  // Check for start button press...
}

int getCurrentPowerLevel() {
  if (IrReceiver.decode()) {
    inputVal = IrReceiver.decodedIRData.decodedRawData;
    IrReceiver.resume();
    
    int irLevel = irPower(inputVal);
    if (irLevel != -1) {
      currentPowerLevel = irLevel;
      Serial.print("IR Signal Received: ");
      displayPowerLevel(currentPowerLevel);
    }
  }
  return currentPowerLevel;
}
```

This ensures that even while the machine is paused waiting for power, the IR receiver remains active and responsive to user input.

---

### **Improvements from Previous Phases**

#### **Improvement 1: Refined Stepper Motor Simulation**

In Phase 1, I used an 8-step LED sequencing pattern. However, during integration testing, I observed that the pattern was **visually cluttered** and difficult for observers to interpret as rotation. I simplified this to a **clean 4-step pattern** with clearer phase transitions:

**Before (Phase 1):**
```cpp
// 8 steps - visually busy, harder to track
// Step 1: LED1 & LED2 ON
// Step 2: LED1 LOW & LED2 HIGH
// Step 3: LED2 HIGH & LED3 HIGH
// Step 4: LED2 LOW & LED3 HIGH
// ... 4 more steps
```

**After (Phase 3):**
```cpp
// 4 steps - clear, distinct phases
// Step 1: LED1 & LED2 ON
// Step 2: LED2 & LED3 ON  
// Step 3: LED3 & LED4 ON
// Step 4: LED4 & LED1 ON
```

This 4-step pattern creates a **visual "chasing" effect** that clearly communicates rotation direction to anyone watching the LEDs.

---

#### **Improvement 2: Speed Optimization**

I adjusted the timing parameters to create more noticeable distinction between wash and spin cycles:

```cpp
// Phase 3 timing adjustments:
clockwiseRevolution(700);  // Wash cycle - slower, more deliberate (was 400)
clockwiseRevolution(400);  // Spin cycle - faster, urgent (was 100)
```

The **700ms step delay** during wash cycles makes the agitation visibly slower and more methodical, while the **400ms delay** during spin creates a clearly faster rhythm. This 75% speed increase between cycles better communicates the machine's state to users.

---

#### **Improvement 3: Simplified Counter-Clockwise Implementation**

I completely rewrote the counter-clockwise revolution function to be more intuitive and maintainable:

```cpp
// Phase 3: Clean, readable reverse sequence
void counterClockwiseRevolution(int stepDelay) {
  // Reverse the clockwise sequence
  digitalWrite(LED1_PIN, HIGH);
  digitalWrite(LED2_PIN, LOW);
  digitalWrite(LED3_PIN, LOW);
  digitalWrite(LED4_PIN, HIGH);
  delay(stepDelay);
  
  digitalWrite(LED1_PIN, LOW);
  digitalWrite(LED2_PIN, LOW);
  digitalWrite(LED3_PIN, HIGH);
  digitalWrite(LED4_PIN, HIGH);
  delay(stepDelay);
  
  digitalWrite(LED1_PIN, LOW);
  digitalWrite(LED2_PIN, HIGH);
  digitalWrite(LED3_PIN, HIGH);
  digitalWrite(LED4_PIN, LOW);
  delay(stepDelay);
  
  digitalWrite(LED1_PIN, HIGH);
  digitalWrite(LED2_PIN, HIGH);
  digitalWrite(LED3_PIN, LOW);
  digitalWrite(LED4_PIN, LOW);
  delay(stepDelay);
  
  revolutionCount++;
  Serial.print("Revolution: ");
  Serial.println(revolutionCount);
}
```

This implementation is **exactly the reverse** of the clockwise sequence, making the code self-documenting and reducing cognitive load when debugging.

---

#### **Improvement 4: Professional Button Handling**

I upgraded from the debounced button code in Phase 1 to a more robust implementation:

```cpp
// Phase 3: Professional button handling with release detection
if (digitalRead(START_BUTTON_PIN) == LOW) {
  delay(50); // Initial debounce
  if (digitalRead(START_BUTTON_PIN) == LOW && !spinningActive) {
    startWashingMachine();
  }
  // Wait for button release - prevents multiple triggers
  while(digitalRead(START_BUTTON_PIN) == LOW) {
    delay(10);
  }
}
```

The addition of the **button release waiting loop** eliminates the possibility of a single press being registered multiple times, a common issue in embedded systems.

---

#### **Improvement 5: Simplified Power Mapping**

I removed the potentiometer-based power simulation used in Phase 2, focusing exclusively on **IR-based power communication**. This better reflects the final system requirements where power availability is communicated via remote (simulating a solar charge controller), not manually adjusted with a potentiometer.

```cpp
// Phase 3: Clean, single-source power monitoring
int getCurrentPowerLevel() {
  if (IrReceiver.decode()) {
    // Process IR signal only - no potentiometer fallback
    // ...
  }
  return currentPowerLevel;
}
```

---

### **How the Integrated System Works**

When a user presses the start button, the following sequence executes:

1. **System checks power for wash cycle** (`checkPower(2, LED1_PIN)`)
2. **If power < Level 2:** Machine displays:
   ```
   >>> POWER CHECK REQUIRED <<<
   Required Power Level: 2
   Current Power Level: 1
   >>> Please press IR button 2 to continue <<<
   ```
3. **User presses IR Button 2** (simulating sufficient solar power)
4. **System detects power increase and resumes:**
   ```
   IR Signal Received: Power Level 2: Moderate Power
   >>> SUFFICIENT POWER (Level 2) - Continuing cycle <<<
   ```
5. **Wash cycle executes** (3×[3CW + 3CCW] at moderate speed)
6. **System checks power for spin cycle** (`checkPower(3, LED1_PIN)`)
7. **User provides Level 3 power** via IR remote
8. **Spin cycle executes** (20×CW at high speed)

---

### **Summary of Phase 3 Contributions**

| Component | Phase 1/2 Status | Phase 3 Integration |
|-----------|------------------|---------------------|
| **Motor Control** | Independent LED sequencing | Power-aware execution |
| **Power Monitoring** | Standalone simulation | Active cycle interruption |
| **User Interaction** | Button-only control | Button + IR handshake |
| **Cycle Flow** | Fixed timing | Power-dependent progression |
| **Code Architecture** | Separate systems | Unified control flow |

This integration phase transformed two parallel prototypes into a **single, cohesive system** that demonstrates the core value proposition of my project: a washing machine that intelligently adapts its operation to available solar power. The machine now **waits for sufficient energy** before proceeding with energy-intensive operations, exactly as required in the project specifications.