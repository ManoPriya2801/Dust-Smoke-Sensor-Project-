# Dust & Smoke Sensor Project

This Arduino project monitors dust density, smoke levels, and current consumption. It uses the GP2Y1014AU0F dust sensor, MQ-6 gas sensor, and a current sensor.

## Components Used:
- Arduino Uno
- GP2Y1014AU0F dust sensor
- MQ-6 gas sensor
- LCD Display
- Buzzer
- LEDs

## How to Use:
1. Clone the repository.
2. Upload the `main.ino` file to your Arduino.
3. Ensure all the components are connected as per the code.

#include "dust_sensor.h"
#include "gas_sensor.h"
#include "current_sensor.h"
#include <LiquidCrystal.h>

LiquidCrystal lcd(2,3,4,5,6,7);

void setup() {
  lcd.begin(16, 2);
  Serial.begin(9600);
  setupDustSensor();
  setupGasSensor();
  setupCurrentSensor();
}

// dust_sensor.h
#ifndef DUST_SENSOR_H
#define DUST_SENSOR_H

#include <LiquidCrystal.h>

void setupDustSensor();
void checkDustSensor(LiquidCrystal &lcd);

#endif

// dust_sensor.cpp
#include "dust_sensor.h"

int measurePin = A0;
int ledPower = 8;
float dustDensity = 0;

void setupDustSensor() {
  pinMode(measurePin, INPUT);
  pinMode(ledPower, OUTPUT);
}

void checkDustSensor(LiquidCrystal &lcd) {
  digitalWrite(ledPower, LOW);
  delayMicroseconds(280);
  float voMeasured = analogRead(measurePin);
  digitalWrite(ledPower, HIGH);
  delayMicroseconds(9680);

  float calcVoltage = voMeasured * (5.0 / 1024);
  dustDensity = 0.17 * calcVoltage - 0.1;
  if (dustDensity < 0) dustDensity = 0.00;

  lcd.clear();
  if (dustDensity > 0.5) {
    lcd.print("Dust detected");
  } else if (dustDensity > 0.3) {
    lcd.print("Moderate dust");
  } else {
    lcd.print("No dust");
  }

  Serial.print("Dust density: ");
  Serial.print(dustDensity);
  Serial.println(" mg/m^3");
}

// gas_sensor.h
#ifndef GAS_SENSOR_H
#define GAS_SENSOR_H

void setupGasSensor();
void checkGasSensor();

#endif

// gas_sensor.cpp
#include "gas_sensor.h"
int smoke = A4;
int buzzer = 11;

void setupGasSensor() {
  pinMode(smoke, INPUT);
  pinMode(buzzer, OUTPUT);
  Serial.println("Gas sensor warming up!");
  delay(20000);  // Warm up time
}

void checkGasSensor() {
  int sensorValue = analogRead(smoke);
  Serial.print("Smoke sensor value: ");
  Serial.print(sensorValue);

  if (sensorValue > 460) {
    Serial.println(" | Smoke detected!");
    tone(buzzer, 1000, 200);
  } else {
    Serial.println(" | Smoke not detected!");
    noTone(buzzer);
  }
}

// current_sensor.h
#ifndef CURRENT_SENSOR_H
#define CURRENT_SENSOR_H

void setupCurrentSensor();
void checkCurrentSensor();

#endif

// current_sensor.cpp
#include "current_sensor.h"

double Voltage = 0;
double Current = 0;

void setupCurrentSensor() {
  // Nothing to set up specifically
}

void checkCurrentSensor() {
  for (int i = 0; i < 1000; i++) {
    Voltage = (Voltage + (.0049 * analogRead(A5)));
    delay(1);
  }
  Voltage = Voltage / 1000;
  Current = (Voltage - 2.5) / 0.185;

  Serial.print("Voltage (V) = ");
  Serial.print(Voltage, 2);
  Serial.print(" | Current (A) = ");
  Serial.print(Current, 2);

  if (Voltage > 0.97) {
    Serial.println(" | Current overconsumed");
    tone(buzzer, 1000, 300);
  } else {
    Serial.println(" | Current is normal");
    noTone(buzzer);
  }
}


void loop() {
  checkDustSensor(lcd);
  checkGasSensor();
  checkCurrentSensor();
  delay(1000);
}
