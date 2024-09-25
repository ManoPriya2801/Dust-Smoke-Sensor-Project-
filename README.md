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

void loop() {
  checkDustSensor(lcd);
  checkGasSensor();
  checkCurrentSensor();
  delay(1000);
}
