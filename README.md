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


void loop() {
  checkDustSensor(lcd);
  checkGasSensor();
  checkCurrentSensor();
  delay(1000);
}
