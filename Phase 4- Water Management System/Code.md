[Circuit link](https://www.tinkercad.com/things/iZLTJ3jjSmA-project-spin-cycle)

```C++
const int VALVE_PIN = 8;         // Blue LED - Water Inlet Valve
const int PUMP_PIN = 9;          // DC Motor - Water Pump
const int WATER_SENSOR_PIN = A5; // Potentiometer - Water Level Sensor
const int START_BUTTON_PIN = 10; // Push Button - Start/Pause

void setup() {
  pinMode(VALVE_PIN, OUTPUT);
  pinMode(PUMP_PIN, OUTPUT);
  pinMode(WATER_SENSOR_PIN, INPUT);
  pinMode(START_BUTTON_PIN, INPUT_PULLUP); // Use internal pull-up resistor
  
  Serial.begin(9600);
  Serial.println("Water Management System - Simulation 2 Step 1");
  Serial.println("---------------------------------------------");
  Serial.println("Turn potentiometer to set water level");
  Serial.println("---------------------------------------------");
}

void loop() {
  int waterLevel = readWaterLevel();
  
  // Display current water level
  Serial.print("Water Level: ");
  Serial.print(waterLevel);
  Serial.println("%");
  
  // Manual valve control test
  if (waterLevel < 80) {
    digitalWrite(VALVE_PIN, HIGH); // Valve OPEN
    Serial.println("Valve: OPEN - Filling drum");
  } else {
    digitalWrite(VALVE_PIN, LOW); // Valve CLOSED
    Serial.println("Valve: CLOSED - Drum full");
  }
  
  // Manual pump control test
  if (waterLevel > 75) {
    digitalWrite(PUMP_PIN, HIGH); // Pump ON
    Serial.println("Pump: ON - Draining water");
  } else {
    digitalWrite(PUMP_PIN, LOW); // Pump OFF
    Serial.println("Pump: OFF - Drum empty");
  }
  
  // Check start button
  if (digitalRead(START_BUTTON_PIN) == LOW) {
    Serial.println("START BUTTON PRESSED!");
    delay(300); // Simple debounce
  }
  
  Serial.println("---------------------------------------------");
  delay(1000);
}

// Read water level from potentiometer (0-100%)
int readWaterLevel() {
  int sensorValue = analogRead(WATER_SENSOR_PIN);
  return map(sensorValue, 0, 1023, 0, 100);
}
```

