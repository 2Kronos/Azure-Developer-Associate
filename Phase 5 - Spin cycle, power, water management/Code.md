[Circuit](https://www.tinkercad.com/things/01tvD1EJwjG-phase-5-cycles-and-power-and-water-management)

```c++
#include <IRremote.hpp>
//Chrinovic Raya Tshiwaya
/* -------------------- PIN / GLOBAL DEFINITIONS -------------------- */
// IR & Start
const int IR_PIN = 3;
const int START_BUTTON_PIN = 10; // Push Button - Start

// Drum motor LEDs (from cycle code)
const int LED1_PIN = 4;    // Controls LED 1
const int LED3_PIN = 5;    // Controls LED 3  
const int LED4_PIN = 6;    // Controls LED 4
const int LED2_PIN = 7;    // Controls LED 2

// Water management (from water code)
const int VALVE_PIN = 8;         // Blue LED - Water Inlet Valve
const int PUMP_PIN = 9;          // DC Motor - Water Pump
const int WATER_SENSOR_PIN = A5; // Potentiometer - Water Level Sensor

/* -------------------- WATER THRESHOLDS -------------------- */
const int WATER_FULL_THRESHOLD  = 80; // "drum full more than 80"
const int WATER_EMPTY_THRESHOLD = 10; // "drum empty less than 10"

/* -------------------- IR / POWER -------------------- */
// IR Variables
unsigned long inputVal;
int currentPowerLevel = 0;

/* -------------------- Washing Machine Variables -------------------- */
// Washing Machine Variables
bool spinningActive = false;
int revolutionCount = 0;

/* -------------------- BUTTON DEBOUNCING VARIABLES -------------------- */
bool lastButtonState = LOW;
unsigned long lastDebounceTime = 0;
const unsigned long debounceDelay = 50;

/* -------------------- FORWARD DECLARATIONS -------------------- */
void startWashingMachine();
void checkPower(int requiredPowerLevel, int pinNum);
int getCurrentPowerLevel();
int irPower(unsigned long irValue);
void displayPowerLevel(int powerLevel);

// Drum motor & cycles
void allLEDsOff();
void wash();
void rinse();
void spin();
void clockwiseRevolution(int stepDelay);
void counterClockwiseRevolution(int stepDelay);

// Water functions
int readWaterLevel();
// Helper functions
void waitForFillDetailed();
void waitForDrainDetailed();
void ensureIRServiced();

/* -------------------- SETUP -------------------- */
void setup() {
  // IR Setup
  IrReceiver.begin(IR_PIN, ENABLE_LED_FEEDBACK);
  
  // Pin setup
  pinMode(LED1_PIN, OUTPUT);
  pinMode(LED2_PIN, OUTPUT);
  pinMode(LED3_PIN, OUTPUT);
  pinMode(LED4_PIN, OUTPUT);
  pinMode(START_BUTTON_PIN, INPUT); // INPUT (not INPUT_PULLUP)

  pinMode(VALVE_PIN, OUTPUT);
  pinMode(PUMP_PIN, OUTPUT);
  pinMode(WATER_SENSOR_PIN, INPUT);
  
  // Turn all LEDs / actuators off initially
  allLEDsOff();
  digitalWrite(VALVE_PIN, LOW);
  digitalWrite(PUMP_PIN, LOW);
  
  Serial.begin(9600);
  Serial.println("SOLAR POWERED WASHING MACHINE");
  Serial.println("Press START button to begin washing cycle");
  Serial.println("You will need to provide power via IR remote");
  Serial.println("---------------------------------------------");
}

/* -------------------- MAIN LOOP -------------------- */
void loop() {
  // Continuously update power level so IR is responsive at all times
  getCurrentPowerLevel();
  
  // Read current button state
  int buttonReading = digitalRead(START_BUTTON_PIN);
  
  // Check if button state has changed
  if (buttonReading != lastButtonState) {
    lastDebounceTime = millis();
  }
  
  // If enough time has passed since last state change, accept the reading
  if ((millis() - lastDebounceTime) > debounceDelay) {
    // If button is pressed (HIGH) and we're not already running
    if (buttonReading == HIGH && !spinningActive) {
      startWashingMachine();
    }
  }
  
  lastButtonState = buttonReading;
}

/* -------------------- MERGED SEQUENCE (startWashingMachine) -------------------- */

void startWashingMachine() {
  spinningActive = true;
  revolutionCount = 0;
  Serial.println("START BUTTON PRESSED!");
  Serial.println("Beginning washing cycle...");
  Serial.println("=============================================");
  
  // ---------- MANUAL FILL FOR WASH ----------
  Serial.println("STEP: Fill drum for WASH.");
  Serial.println("Please TURN the POT to INCREASE water level until it is MORE THAN 80%.");
  digitalWrite(VALVE_PIN, HIGH); // Valve OPEN - user fills manually
  waitForFillDetailed();
  digitalWrite(VALVE_PIN, LOW);  // Valve CLOSED
  Serial.println("DRUM FULL (>80%) - Valve closed");
  delay(500);

  // ---------- REQUEST POWER FOR WASH (IR button 2) ----------
  checkPower(2, LED1_PIN);
  wash();
  delay(2000);

  // ---------- DRAIN AFTER WASH ----------
  Serial.println("STEP: Drain dirty water after WASH.");
  Serial.println("Please TURN the POT in the OPPOSITE DIRECTION to DECREASE water level until it is LESS THAN 10%.");
  digitalWrite(PUMP_PIN, HIGH); // Pump ON
  waitForDrainDetailed();
  digitalWrite(PUMP_PIN, LOW);  // Pump OFF
  Serial.println("DRUM EMPTY (<10%) - Pump stopped");
  delay(500);
  
  // ---------- MANUAL FILL FOR RINSE ----------
  Serial.println("STEP: Fill drum for RINSE.");
  Serial.println("Please TURN the POT to INCREASE water level until it is MORE THAN 80%.");
  digitalWrite(VALVE_PIN, HIGH); // Valve OPEN
  waitForFillDetailed();
  digitalWrite(VALVE_PIN, LOW);  // Valve CLOSED
  Serial.println("DRUM FULL (>80%) - Valve closed");
  delay(500);

  // ---------- REQUEST POWER FOR RINSE (IR button 2) ----------
  checkPower(2, LED1_PIN);
  rinse();
  delay(2000);

  // ---------- DRAIN AFTER RINSE ----------
  Serial.println("STEP: Drain dirty water after RINSE.");
  Serial.println("Please TURN the POT in the OPPOSITE DIRECTION to DECREASE water level until it is LESS THAN 10%.");
  digitalWrite(PUMP_PIN, HIGH); // Pump ON
  waitForDrainDetailed();
  digitalWrite(PUMP_PIN, LOW);  // Pump OFF
  Serial.println("DRUM EMPTY (<10%) - Pump stopped");
  delay(500);

  // ---------- REQUEST POWER FOR SPIN (IR button 3) ----------
  checkPower(3, LED1_PIN);
  spin();
  
  // Reset for next button press
  spinningActive = false;
  Serial.println("=============================================");
  Serial.println("Finished");
  Serial.println("Ready for next cycle - Press START button");
}

/* -------------------- ORIGINAL CYCLE & POWER CODE FUNCTIONS -------------------- */

// REQUIRED FUNCTION: checkPower(int requiredPowerLevel, int pinNum)
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

// Get the current power level from IR remote only
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

// Convert IR codes to power levels (0-3)
int irPower(unsigned long irValue) {
  switch(irValue) {
    case 4077698816: return 0; // Button 0
    case 4010852096: return 1; // Button 1
    case 3994140416: return 2; // Button 2
    case 3977428736: return 3; // Button 3
    default: return -1;
  }
}

// Display power level description
void displayPowerLevel(int powerLevel) {
  switch(powerLevel) {
    case 0: Serial.println("Power Level 0: Battery charging"); break;
    case 1: Serial.println("Power Level 1: Low power"); break;
    case 2: Serial.println("Power Level 2: Moderate Power"); break;
    case 3: Serial.println("Power Level 3: Maximum Power"); break;
  }
}

/* -------------------- ORIGINAL DRUM / CYCLE FUNCTIONS -------------------- */

// Turn all LEDs off
void allLEDsOff() {
  digitalWrite(LED1_PIN, LOW);
  digitalWrite(LED2_PIN, LOW);
  digitalWrite(LED3_PIN, LOW);
  digitalWrite(LED4_PIN, LOW);
}

// WASH cycle: 3 clockwise + 3 counter-clockwise (repeat 3x)
void wash() {
  Serial.println("Starting WASH cycle...");
  Serial.println("3 clockwise + 3 counter-clockwise revolutions (repeat 3x)");
  
  for (int cycle = 0; cycle < 3; cycle++) {
    Serial.print("Cycle ");
    Serial.print(cycle + 1);
    Serial.println(": Clockwise 3 revolutions");
    
    // Clockwise - 3 revolutions
    for (int rev = 0; rev < 3; rev++) {
      clockwiseRevolution(400); // Moderate speed
    }
    
    Serial.print("Cycle ");
    Serial.print(cycle + 1);
    Serial.println(": Counter-clockwise 3 revolutions");
    delay(1000); // 1-second pause
    
    // Counter-clockwise - 3 revolutions
    for (int rev = 0; rev < 3; rev++) {
      counterClockwiseRevolution(400); // Moderate speed
    }
    
    if (cycle < 2) delay(1000); // 1-second pause between cycles
  }
  
  Serial.println("WASH cycle completed!");
  allLEDsOff();
}

void rinse() {
  Serial.println("Starting RINSE cycle...");
  Serial.println("3 clockwise + 3 counter-clockwise revolutions (repeat 3x)");
  
  for (int cycle = 0; cycle < 3; cycle++) {
    Serial.print("Cycle ");
    Serial.print(cycle + 1);
    Serial.println(": Clockwise 3 revolutions");
    
    // Clockwise - 3 revolutions
    for (int rev = 0; rev < 3; rev++) {
      clockwiseRevolution(400); // Moderate speed
    }
    
    Serial.print("Cycle ");
    Serial.print(cycle + 1);
    Serial.println(": Counter-clockwise 3 revolutions");
    delay(1000); // 1-second pause
    
    // Counter-clockwise - 3 revolutions
    for (int rev = 0; rev < 3; rev++) {
      counterClockwiseRevolution(400); // Moderate speed
    }
    
    if (cycle < 2) delay(1000); // 1-second pause between cycles
  }
  
  Serial.println("RINSE cycle completed!");
  allLEDsOff();
}

// SPIN cycle: 20 revolutions at double speed (one direction)
void spin() {
  Serial.println("Starting SPIN cycle - 20 revolutions at high speed");
  
  // 20 revolutions clockwise at double speed
  for (int rev = 0; rev < 20; rev++) {
    clockwiseRevolution(100); // Double speed
  }
  
  Serial.println("SPIN cycle completed!");
  allLEDsOff();
}

// Single clockwise revolution (8 steps)
void clockwiseRevolution(int stepDelay) {
  // Step 1: LED1 & LED2 ON
  digitalWrite(LED1_PIN, HIGH);
  digitalWrite(LED2_PIN, HIGH);
  digitalWrite(LED3_PIN, LOW);
  digitalWrite(LED4_PIN, LOW);
  delay(stepDelay);
  
  // Step 2: LED1 LOW & LED2 HIGH
  digitalWrite(LED1_PIN, LOW);
  digitalWrite(LED2_PIN, HIGH);
  delay(stepDelay);
  
  // Step 3: LED2 HIGH & LED3 HIGH
  digitalWrite(LED2_PIN, HIGH);
  digitalWrite(LED3_PIN, HIGH);
  delay(stepDelay);
  
  // Step 4: LED2 LOW & LED3 HIGH
  digitalWrite(LED2_PIN, LOW);
  digitalWrite(LED3_PIN, HIGH);
  delay(stepDelay);
  
  // Step 5: LED3 HIGH LED4 HIGH
  digitalWrite(LED3_PIN, HIGH);
  digitalWrite(LED4_PIN, HIGH);
  delay(stepDelay);
  
  // Step 6: LED3 LOW LED4 HIGH
  digitalWrite(LED3_PIN, LOW);
  digitalWrite(LED4_PIN, HIGH);
  delay(stepDelay);
  
  // Step 7: LED1 HIGH LED4 HIGH
  digitalWrite(LED1_PIN, HIGH);
  digitalWrite(LED4_PIN, HIGH);
  delay(stepDelay);
  
  // Step 8: LED1 HIGH LED4 LOW
  digitalWrite(LED1_PIN, HIGH);
  digitalWrite(LED4_PIN, LOW);
  delay(stepDelay);
  
  revolutionCount++;
  Serial.print("Revolution: ");
  Serial.println(revolutionCount);
}

// Single counter-clockwise revolution (8 steps)
void counterClockwiseRevolution(int stepDelay) {
  // Step 1: LED1 & LED4 ON
  digitalWrite(LED1_PIN, HIGH);
  digitalWrite(LED4_PIN, HIGH);
  digitalWrite(LED2_PIN, LOW);
  digitalWrite(LED3_PIN, LOW);
  delay(stepDelay);
  
  // Step 2: LED1 LOW & LED4 HIGH
  digitalWrite(LED1_PIN, LOW);
  digitalWrite(LED4_PIN, HIGH);
  delay(stepDelay);
  
  // Step 3: LED4 HIGH & LED3 HIGH
  digitalWrite(LED4_PIN, HIGH);
  digitalWrite(LED3_PIN, HIGH);
  delay(stepDelay);
  
  // Step 4: LED4 LOW & LED3 HIGH
  digitalWrite(LED4_PIN, LOW);
  digitalWrite(LED3_PIN, HIGH);
  delay(stepDelay);
  
  // Step 5: LED3 HIGH & LED2 HIGH
  digitalWrite(LED3_PIN, HIGH);
  digitalWrite(LED2_PIN, HIGH);
  delay(stepDelay);
  
  // Step 6: LED3 LOW & LED2 HIGH
  digitalWrite(LED3_PIN, LOW);
  digitalWrite(LED2_PIN, HIGH);
  delay(stepDelay);
  
  // Step 7: LED2 HIGH & LED1 HIGH
  digitalWrite(LED2_PIN, HIGH);
  digitalWrite(LED1_PIN, HIGH);
  delay(stepDelay);
  
  // Step 8: LED1 HIGH & LED2 LOW
  digitalWrite(LED1_PIN, HIGH);
  digitalWrite(LED2_PIN, LOW);
  delay(stepDelay);
  
  revolutionCount++;
  Serial.print("Revolution: ");
  Serial.println(revolutionCount);
}

/* -------------------- ORIGINAL WATER MANAGEMENT FUNCTION -------------------- */

// Read water level from potentiometer (0-100%)
int readWaterLevel() {
  int sensorValue = analogRead(WATER_SENSOR_PIN);
  return map(sensorValue, 0, 1023, 0, 100);
}

/* -------------------- NEW HELPERS (for manual filling/draining loops) -------------------- */

// Wait until user increases POT above full threshold (>80%)
void waitForFillDetailed() {
  int lastPrinted = -1;
  while (true) {
    int wl = readWaterLevel();
    // Only print when value changes to reduce serial spam
    if (wl != lastPrinted) {
      Serial.print("Water Level: ");
      Serial.print(wl);
      Serial.println("%");
      lastPrinted = wl;
    }
    // If water is now full, break
    if (wl > WATER_FULL_THRESHOLD) break;
    // Service IR so user can press IR buttons while filling
    ensureIRServiced();
    delay(200);
  }
}

// Wait until user decreases POT below empty threshold (<10%)
void waitForDrainDetailed() {
  int lastPrinted = -1;
  while (true) {
    int wl = readWaterLevel();
    // Only print when value changes to reduce serial spam
    if (wl != lastPrinted) {
      Serial.print("Water Level: ");
      Serial.print(wl);
      Serial.println("%");
      lastPrinted = wl;
    }
    // If water is now empty, break
    if (wl < WATER_EMPTY_THRESHOLD) break;
    // Service IR so user can press IR buttons while draining
    ensureIRServiced();
    delay(200);
  }
}

// Ensure IR input is processed inside waiting loops
void ensureIRServiced() {
  if (IrReceiver.decode()) {
    inputVal = IrReceiver.decodedIRData.decodedRawData;
    IrReceiver.resume();
    int irLevel = irPower(inputVal);
    if (irLevel != -1) {
      currentPowerLevel = irLevel;
      Serial.print("IR Signal Received while waiting: ");
      displayPowerLevel(currentPowerLevel);
    }
  }
}
```