# Dual Axis Solar Tracker 🌞⚙️

This project is a **Dual Axis Solar Tracker** system designed to automatically align a solar panel with the direction of maximum sunlight using light sensors (LDRs) and servo motors. It also integrates temperature, humidity, and rain detection with real-time data display on an I2C LCD.

## 📌 Features

- ☀️ **Dual-axis solar tracking** for maximizing energy capture
- 🌡️ **Temperature and humidity sensing** using DHT11
- 🌧️ **Rain detection** to protect the panel
- 📟 **16x2 LCD display** for real-time environmental data
- 🎛️ Easy-to-build on an FR4 PCB board using KiCad
- 🔌 Powered via 12V external supply

## ⚙️ Components Used

- Arduino Nano
- 2x Servo Motors (for X and Y axis)
- 4x LDRs (Light Dependent Resistors)
- DHT11 Sensor
- Rain Sensor (Analog + Digital)
- 16x2 I2C LCD Display
- FR4 PCB (custom designed in KiCad)
- 12V DC power supply
- Capacitors: 0.1µF, 47µF, 100µF
- Regulator diode

## 🛠️ Working Principle

The system uses four LDR sensors arranged in a cross configuration to compare light intensity from different directions. Based on the differences, the servos adjust the solar panel both horizontally and vertically to face the direction of highest intensity. The system also continuously monitors temperature, humidity, and rain presence, displaying the data on the LCD. If rain is detected, the panel can be redirected or halted to prevent damage.

## 📸 Schematic & PCB

- Designed using **KiCad**
- Organized and compact layout for ease of fabrication and debugging
- Supports easy component placement and routing

## 👩🏻‍💻 Code 

#include <Servo.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <DHT.h>

#define DHTPIN 0        // DHT11 sensor connected to D0 (not recommended, but as per your setup)
#define DHTTYPE DHT11
#define RAIN_ANALOG A6
#define RAIN_DIGITAL 1  // Also avoid D1 during uploading, disconnect rain sensor while uploading

DHT dht(DHTPIN, DHTTYPE);
Servo horizontalServo;
Servo verticalServo;
LiquidCrystal_I2C lcd(0x27, 16, 2);

// LDR pins
const int ldrTL = A0;
const int ldrTR = A1;
const int ldrBL = A2;
const int ldrBR = A3;

int horizontalServoPin = 9;
int verticalServoPin = 10;

int horizontalAngle = 90;
int verticalAngle = 90;
const int horizontalLimitHigh = 180;
const int horizontalLimitLow = 10;
const int verticalLimitHigh = 180;
const int verticalLimitLow = 10;

const int threshold = 20; // Light difference threshold
const int stepSize = 5;   // Servo movement step

void setup() {
    dht.begin();
    horizontalServo.attach(horizontalServoPin);
    verticalServo.attach(verticalServoPin);

    lcd.init();
    lcd.backlight();

    pinMode(RAIN_ANALOG, INPUT);
    pinMode(RAIN_DIGITAL, INPUT);

    horizontalServo.write(horizontalAngle);
    verticalServo.write(verticalAngle);
    delay(150);
}

void loop() {
    // Read LDR values
    int TL = analogRead(ldrTL);
    int TR = analogRead(ldrTR);
    int BL = analogRead(ldrBL);
    int BR = analogRead(ldrBR);

    // Read Rain Sensor values
    int rainValueAnalog = analogRead(RAIN_ANALOG);
    int rainValueDigital = digitalRead(RAIN_DIGITAL);

    // Read Temperature and Humidity
    float temperature = dht.readTemperature();
    float humidity = dht.readHumidity();

    // Compute average LDR values
    int avgTop = (TL + TR) / 2;
    int avgBottom = (BL + BR) / 2;
    int avgLeft = (TL + BL) / 2;
    int avgRight = (TR + BR) / 2;

    // Vertical Movement
    if (abs(avgTop - avgBottom) > threshold) {
        if (avgTop < avgBottom) {
            verticalAngle -= stepSize;
            if (verticalAngle < verticalLimitLow) verticalAngle = verticalLimitLow;
        } else {
            verticalAngle += stepSize;
            if (verticalAngle > verticalLimitHigh) verticalAngle = verticalLimitHigh;
        }
        verticalServo.write(verticalAngle);
    }

    // Horizontal Movement
    if (abs(avgLeft - avgRight) > threshold) {
        if (avgLeft > avgRight) {
            horizontalAngle -= stepSize;
            if (horizontalAngle < horizontalLimitLow) horizontalAngle = horizontalLimitLow;
        } else {
            horizontalAngle += stepSize;
            if (horizontalAngle > horizontalLimitHigh) horizontalAngle = horizontalLimitHigh;
        }
        horizontalServo.write(horizontalAngle);
    }

    // Display on LCD
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Temp: ");
    lcd.print(temperature);
    lcd.print("C");

    lcd.setCursor(0, 1);
    if (rainValueAnalog < 500 || rainValueDigital == 0) {
        lcd.print("Rain Detected!");
    } else {
        lcd.print("Humidity: ");
        lcd.print(humidity);
        lcd.print("%");
    }

    delay(100); // Wait before next reading
}


https://github.com/user-attachments/assets/accd74fc-cd20-4451-8b1d-cb7651f4eb0f

