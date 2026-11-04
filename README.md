#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// LCD configuration
LiquidCrystal_I2C lcd(0x27, 16, 2);

// ESP32 pin connections
const int trigPin = 5;
const int echoPin = 18;
const int buzzerPin = 23;

// Tank configuration
const float tankHeightCm = 30.0;
const float alarmLevel = 90.0;

// Variables
float distanceCm = 0;
float waterLevelCm = 0;
float waterPercentage = 0;

void setup() {

  Serial.begin(115200);

  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(buzzerPin, OUTPUT);

  digitalWrite(buzzerPin, LOW);

  // LCD initialization
  lcd.init();
  lcd.backlight();

  lcd.setCursor(0, 0);
  lcd.print("WATER LEVEL");
  lcd.setCursor(0, 1);
  lcd.print("MONITORING");
  delay(2000);
  lcd.clear();
}

void loop() {

  // Send ultrasonic signal
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);

  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);

  digitalWrite(trigPin, LOW);

  // Read echo signal
  long duration = pulseIn(echoPin, HIGH, 30000);

  // Check sensor error
  if (duration == 0) {

    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("SENSOR ERROR");

    digitalWrite(buzzerPin, LOW);

    delay(1000);
    return;
  }

  // Calculate distance in cm
  distanceCm = duration * 0.0343 / 2.0;

  // Calculate water level
  waterLevelCm = tankHeightCm - distanceCm;

  // Limit water level
  waterLevelCm = constrain(waterLevelCm, 0, tankHeightCm);

  // Calculate water percentage
  waterPercentage = (waterLevelCm / tankHeightCm) * 100.0;

  // Display distance
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Dist:");
  lcd.print(distanceCm, 1);
  lcd.print(" cm");

  // Display water percentage
  lcd.setCursor(0, 1);
  lcd.print("Level:");
  lcd.print(waterPercentage, 1);
  lcd.print("%");

  // Buzzer alert at 90%
  if (waterPercentage >= alarmLevel) {

    digitalWrite(buzzerPin, HIGH);

  } else {

    digitalWrite(buzzerPin, LOW);
  }

  // Serial monitor output
  Serial.print("Distance: ");
  Serial.print(distanceCm);
  Serial.print(" cm");

  Serial.print(" | Water Level: ");
  Serial.print(waterPercentage);
  Serial.println("%");

  delay(1000);
}
