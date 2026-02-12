
[Circuit Link](https://www.tinkercad.com/things/iZLTJ3jjSmA-project-spin-cycle)



```c++
const int LED1_PIN = 4;    // Controls LED 1
const int LED3_PIN = 5;    // Controls LED 3  
const int LED4_PIN = 6;    // Controls LED 4
const int LED2_PIN = 7;    // Controls LED 2
const int START_BUTTON_PIN = 10; // CORRECTED: Push Button - Start

bool spinningActive = false;
int revolutionCount = 0;
bool lastButtonState = LOW;
unsigned long lastDebounceTime = 0;
const unsigned long debounceDelay = 50;

void setup() {
  pinMode(LED1_PIN, OUTPUT);
  pinMode(LED2_PIN, OUTPUT);
  pinMode(LED3_PIN, OUTPUT);
  pinMode(LED4_PIN, OUTPUT);
  pinMode(START_BUTTON_PIN, INPUT);
  
  // Turn all LEDs off initially
  allLEDsOff();
  
  Serial.begin(9600);
  Serial.println("DRUM MOTOR SIMULATION - CORRECT MAPPING");
  Serial.println("Press START button to begin");
  Serial.println("---------------------------------------------");
}

void loop() {
  // Read current button state
  int buttonReading = digitalRead(START_BUTTON_PIN);
  Serial.println(buttonReading);
  
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

void startWashingMachine() {
  spinningActive = true;
  revolutionCount = 0;
  Serial.println("START BUTTON PRESSED!");
  
  // Test wash cycle
  wash_rinse();
  delay(2000);
  
  // Test spin cycle
  spin();
  
  // Reset for next button press
  spinningActive = false;
  Serial.println("Ready for next cycle - Press START button");
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

// SPIN cycle: 20 revolutions at double speed (one direction)
void spin() {
  Serial.println("Starting SPIN cycle - 20 revolutions at high speed");
  
  // 20 revolutions clockwise at double speed
  for (int rev = 0; rev < 20; rev++) {
    clockwiseRevolution(100); // Double speed
  }
  
  Serial.println("SPIN COMPLETE - Finished!");
  allLEDsOff();
}

// Single clockwise revolution (4 steps) - YOUR CODE IS CORRECT
void clockwiseRevolution(int stepDelay) {
  // Step 1: LED1 & LED2 ON
  digitalWrite(LED1_PIN, HIGH);
  digitalWrite(LED2_PIN, HIGH);
  digitalWrite(LED3_PIN, LOW);
  digitalWrite(LED4_PIN, LOW);
  delay(stepDelay);
  
  // Step 2: LED1 LOW & LED2  HIGH
  digitalWrite(LED1_PIN, LOW);
  digitalWrite(LED2_PIN, HIGH);
   delay(stepDelay);
  
  // Step 3: LED2 HIGH & LED3  HIGH
 
  digitalWrite(LED2_PIN, HIGH);
   digitalWrite(LED3_PIN, HIGH);
  delay(stepDelay);
  
  // Step 4: LED2 LOW & LED3  HIGH
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
  
  
  revolutionCount++;
  Serial.print("Revolution: ");
  Serial.println(revolutionCount);
}

// Single counter-clockwise revolution (4 steps) - YOUR CODE IS CORRECT
void counterClockwiseRevolution(int stepDelay) {
  // Reverse the clockwise sequence
  // Step 1: LED1 & LED4 ON (reverse of clockwise step 1)
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
```
