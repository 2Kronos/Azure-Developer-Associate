[Circuit ](https://www.tinkercad.com/things/kNIxDl4IsVj-phase-3-spin-cycles-and-power-management-)

```c++
#include <IRremote.hpp>

// Pin Definitions
const int IR_PIN = 3;
const int LED1_PIN = 4;    // Controls LED 1
const int LED3_PIN = 5;    // Controls LED 3  
const int LED4_PIN = 6;    // Controls LED 4
const int LED2_PIN = 7;    // Controls LED 2
const int START_BUTTON_PIN = 10; // Push Button - Start

// IR Variables
unsigned long inputVal;
int currentPowerLevel = 0;

// Washing Machine Variables
bool spinningActive = false;
int revolutionCount = 0;

void setup() {
  // IR Setup
  IrReceiver.begin(IR_PIN, ENABLE_LED_FEEDBACK);
  
  // Pin setup
  pinMode(LED1_PIN, OUTPUT);
  pinMode(LED2_PIN, OUTPUT);
  pinMode(LED3_PIN, OUTPUT);
  pinMode(LED4_PIN, OUTPUT);
  pinMode(START_BUTTON_PIN, INPUT_PULLUP);
  
  // Turn all LEDs off initially
  allLEDsOff();
  
  Serial.begin(9600);
  Serial.println("SOLAR POWERED WASHING MACHINE");
  Serial.println("Press START button to begin washing cycle");
  Serial.println("You will need to provide power via IR remote");
  Serial.println("---------------------------------------------");
}

void loop() {
  // Continuously update power level
  getCurrentPowerLevel();
  
  // Check for start button press with proper debouncing
  if (digitalRead(START_BUTTON_PIN) == LOW) {
    delay(50); // Initial debounce
    if (digitalRead(START_BUTTON_PIN) == LOW && !spinningActive) {
      startWashingMachine();
    }
    // Wait for button release
    while(digitalRead(START_BUTTON_PIN) == LOW) {
      delay(10);
    }
  }
}

void startWashingMachine() {
  spinningActive = true;
  revolutionCount = 0;
  Serial.println("START BUTTON PRESSED!");
  Serial.println("Beginning washing cycle...");
  Serial.println("=============================================");
  
  // WASH CYCLE - Requires Power Level 2
  checkPower(2, LED1_PIN);
  wash_rinse();
  
  delay(2000);
  
  // SPIN CYCLE - Requires Power Level 3
  checkPower(3, LED1_PIN);
  spin();
  
  // Reset for next button press
  spinningActive = false;
  Serial.println("=============================================");
  Serial.println("Finished");
  Serial.println("Ready for next cycle - Press START button");
}

// REQUIRED FUNCTION: checkPower(int requiredPowerLevel, int pinNum)
// Pauses execution until sufficient power is available
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

// Turn all LEDs off
void allLEDsOff() {
  digitalWrite(LED1_PIN, LOW);
  digitalWrite(LED2_PIN, LOW);
  digitalWrite(LED3_PIN, LOW);
  digitalWrite(LED4_PIN, LOW);
}

// WASH/RINSE cycle: 3 clockwise + 3 counter-clockwise (repeat 3x)
void wash_rinse() {
  Serial.println("Starting WASH cycle...");
  Serial.println("3 clockwise + 3 counter-clockwise revolutions (repeat 3x)");
  
  for (int cycle = 0; cycle < 3; cycle++) {
    Serial.print("Cycle ");
    Serial.print(cycle + 1);
    Serial.println(": Clockwise 3 revolutions");
    
    // Clockwise - 3 revolutions
    for (int rev = 0; rev < 3; rev++) {
      clockwiseRevolution(700); // Moderate speed
    }
    
    Serial.print("Cycle ");
    Serial.print(cycle + 1);
    Serial.println(": Counter-clockwise 3 revolutions");
    delay(1000); // 1-second pause
    
    // Counter-clockwise - 3 revolutions
    for (int rev = 0; rev < 3; rev++) {
      counterClockwiseRevolution(700); // Moderate speed
    }
    
    if (cycle < 2) delay(1000); // 1-second pause between cycles
  }
  
  Serial.println("WASH cycle completed!");
  allLEDsOff();
}

// SPIN cycle: 20 revolutions at double speed (one direction)
void spin() {
  Serial.println("Starting SPIN cycle - 20 revolutions at high speed");
  
  // 20 revolutions clockwise at double speed
  for (int rev = 0; rev < 20; rev++) {
    clockwiseRevolution(400); // Double speed
  }
  
  Serial.println("SPIN cycle completed!");
  allLEDsOff();
}

// Single clockwise revolution (4 steps)
void clockwiseRevolution(int stepDelay) {
  // Step 1: LED1 & LED2 ON
  digitalWrite(LED1_PIN, HIGH);
  digitalWrite(LED2_PIN, HIGH);
  digitalWrite(LED3_PIN, LOW);
  digitalWrite(LED4_PIN, LOW);
  delay(stepDelay);
  
  // Step 2: LED2 & LED3 ON
  digitalWrite(LED1_PIN, LOW);
  digitalWrite(LED2_PIN, HIGH);
  digitalWrite(LED3_PIN, HIGH);
  digitalWrite(LED4_PIN, LOW);
  delay(stepDelay);
  
  // Step 3: LED3 & LED4 ON
  digitalWrite(LED1_PIN, LOW);
  digitalWrite(LED2_PIN, LOW);
  digitalWrite(LED3_PIN, HIGH);
  digitalWrite(LED4_PIN, HIGH);
  delay(stepDelay);
  
  // Step 4: LED4 & LED1 ON
  digitalWrite(LED1_PIN, HIGH);
  digitalWrite(LED2_PIN, LOW);
  digitalWrite(LED3_PIN, LOW);
  digitalWrite(LED4_PIN, HIGH);
  delay(stepDelay);
  
  revolutionCount++;
  Serial.print("Revolution: ");
  Serial.println(revolutionCount);
}

// Single counter-clockwise revolution (4 steps)
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