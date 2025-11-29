# Fire-and-smoke-detection-with-alarm-using-gsm-900a
Fire and Smoke Alarm System with GSM SMS Alerts using Arduino  This Arduino project detects fire and smoke using an MQ-2 smoke sensor and a flame sensor. If a dangerous condition is detected, it activates a buzzer and automatically sends emergency SMS alerts via a SIM900 GSM module to up to two configured phone numbers.
Arduino-based fire and smoke alarm system using MQ-2 gas sensor and flame sensor, with local buzzer alarm and SMS alerts via SIM900A GSM module to two emergency contacts.​​

Table of Contents
Overview

Features

System Architecture

Hardware Components

Circuit Connections

Arduino Code

Code Explanation

How to Run

Testing and Calibration

Project Images

Applications

Future Improvements

Repository Structure

License

Overview
This project implements a low-cost fire and smoke detection system using Arduino UNO, an MQ-2 smoke sensor, and a flame sensor. When smoke concentration crosses a threshold or a flame is detected, the system activates a buzzer and sends SMS alerts through a SIM900A (GSM 900A) module to up to two configured phone numbers. The system is designed for homes, labs, and small industrial spaces where early warning and remote notification are critical.​​

Features
Dual sensing: MQ-2 smoke sensor (analog) + flame sensor (digital, active LOW).​

Local audible alarm using a buzzer.

GSM SMS alerts sent automatically to two mobile numbers with live sensor readings.​

SMS cooldown logic (1 minute) to avoid repeated message spamming.​

Serial Monitor output for real-time debugging and calibration (9600 baud).

Simple wiring and easy to reproduce as a mini-project for ECE/embedded students.

System Architecture
High-level flow of the system:​

MQ-2 and flame sensor continuously monitor the environment.

Arduino reads smoke (0–1023) and flame (HIGH/LOW) values in the loop.

If smoke value exceeds threshold or flame is detected:

Buzzer is turned ON.

SMS alerts are sent to the configured phone numbers if cooldown period has passed.

When environment returns to normal:

Buzzer is turned OFF.

Status message is printed to Serial Monitor.

Hardware Components
Component	Quantity	Description / Role
Arduino UNO	1	Main microcontroller board.
MQ-2 Gas Sensor	1	Detects smoke and combustible gases (analog output).​
Flame Sensor	1	Detects presence of flame (digital output, active LOW).​
SIM900A GSM Module	1	Sends SMS alerts via GSM network.​
Buzzer	1	Provides audible alarm.
Breadboard	1	Prototyping platform.
Jumper Wires	–	For connections between components.
5V Power Supply	1	Powers Arduino and sensors (ensure enough current).
Approximate total hardware cost is within a low-budget student project range.​

Circuit Connections
Basic wiring (adjust pin labels if your hardware differs):​​

MQ-2 Smoke Sensor

VCC → 5V

GND → GND

AOUT → A0 (Arduino analog input)

Flame Sensor

VCC → 5V

GND → GND

D0 → D4 (Arduino digital input)

Buzzer

→ D5

– → GND

SIM900A GSM Module

TX → D10 (Arduino RX for SoftwareSerial)

RX → D9 (Arduino TX for SoftwareSerial)

VCC → 5V (or external 5V adapter as per module spec)

GND → GND

See repository images (components connection.jpg, overall design.jpg, wiring design.jpg) for detailed wiring and serial-monitor connection diagrams.​

Arduino Code
The core logic is implemented in the Arduino sketch file (Fire,Smoke with Alarm using gsm.ino.txt or .ino).​
Key configurable parameters inside the code:

const int smokeThreshold = 300; – smoke level threshold (tune during calibration).

const int flameThreshold = LOW; – active level for flame sensor.

String phoneNumber1, phoneNumber2 – recipient mobile numbers (with country code).

const unsigned long smsCooldownPeriod = 60000; – 60,000 ms = 1 minute SMS cooldown.

Code Explanation
1. Library and Global Declarations
#include <SoftwareSerial.h> enables a software UART for the SIM900A so the hardware Serial can remain free for the Serial Monitor.​

Pin constants (flameSensorPin, smokeSensorPin, buzzerPin) define connections clearly.

SoftwareSerial SIM900(9, 10); creates a serial interface on pins 9 (RX) and 10 (TX) for GSM communication.​

Thresholds and phone numbers are defined as global constants/variables for easy modification.

alarmActive, lastSMSTime, and smsCooldownPeriod implement state tracking and SMS rate limiting.

2. setup()
Initializes both Serial and SIM900 at 9600 baud.

Configures sensor and buzzer pins (pinMode), and sets buzzer OFF initially.

Provides a 3-second delay for GSM module to register on the network.​

Sends "AT" to check module presence and "AT+CMGF=1" to set SMS in text mode, relaying responses to Serial through updateSerial() for debugging.

3. loop()
Continuously reads:

smokeValue = analogRead(smokeSensorPin);

flameValue = digitalRead(flameSensorPin);

Prints both values to the Serial Monitor so the user can see real-time sensor data and use it for threshold calibration.

If smokeValue > smokeThreshold OR flameValue == flameThreshold, fireDetected() is called; otherwise noFireDetected() is executed.​

4. fireDetected(int smokeValue, int flameValue)
Executes only when alarmActive is false to avoid repeating actions for the same event.

Sets alarmActive = true and turns the buzzer ON.

Uses millis() to check if the current time minus lastSMSTime is greater than the cooldown period (1 minute).

If allowed, sends SMS to phoneNumber1 and phoneNumber2 by calling sendSMS() with the current sensor values, then updates lastSMSTime.

5. noFireDetected()
If the system was previously in alarm state (alarmActive == true), prints a “normal condition restored” message and resets alarmActive to false.

Ensures the buzzer is OFF when no fire or smoke is present.

6. sendSMS(String phoneNumber, int smokeValue, int flameValue)
Issues AT+CMGS="phoneNumber" command to initiate an SMS.

Sends a formatted message including:

A text alert line.

Current smoke value.

Interpreted flame status (“FLAME DETECTED” or “NO FLAME”).

Ends the SMS by sending ASCII 26 (SIM900.write(26)), which corresponds to CTRL+Z in AT command mode and instructs the GSM module to send the SMS.​

7. updateSerial()
Bridges the hardware Serial (USB) and the GSM software serial port.

Any data typed in the Serial Monitor is forwarded to the GSM module, and any GSM replies are printed back to the monitor.

Helpful for manual AT command testing and troubleshooting connection issues.​

How to Run
Install the Arduino IDE and drivers for Arduino UNO.

Connect the hardware according to the Circuit Connections section.

Insert an active SIM card (with SMS capability) into the SIM900A module.

Open the .ino file from this repository in Arduino IDE.

Set the correct Board (Arduino UNO) and COM Port.

Replace phoneNumber1 and phoneNumber2 with real phone numbers in international format (e.g., +91XXXXXXXXXX).

Upload the sketch to Arduino.

Open Serial Monitor at 9600 baud to view sensor data and status messages.​​

Testing and Calibration
Baseline Measurement

Power the system in clean air with no smoke or flame.

Note the average smokeValue from the Serial Monitor (typically in the range of 100–200 depending on sensor and environment).​

Set Threshold

Choose smokeThreshold approximately 100–150 counts above the clean-air baseline to reduce false alarms.

Smoke Test

Use incense or safe smoke source near the MQ-2 sensor.

Verify that smokeValue rises above the threshold and that:

Buzzer turns ON.

SMS messages are received on the configured phones.

Flame Test

Use a lighter or small flame in front of the flame sensor (do not touch the sensor).

Confirm that flameValue goes LOW, buzzer activates, and SMS is sent.

Cooldown Verification

Trigger multiple times within 1 minute and check that additional SMS messages are suppressed during the cooldown period.​

Project Images
You can find supporting images in this repository:​

components connection.jpg – Basic component wiring.

components connections with serial monitor.jpg – Setup with USB/Serial monitor.

overall design.jpg – Full assembled prototype.

serial monitor output.jpg – Example of serial readings during testing.

wiring design.jpg – Clean wiring layout diagram.

These images help understand the physical setup and are suitable for reports or presentations.

Applications
Home and apartment fire/smoke early warning system.

Small offices, shops, and labs where remote SMS alerts are useful.

Educational demonstration for embedded systems, IoT, and sensor interfacing.

Can be extended for warehouses or remote monitoring locations with GSM coverage.​

Future Improvements
Potential extensions to this project:​

Add an LCD or OLED display to show live smoke level and flame status.

Integrate Wi‑Fi (ESP8266/ESP32) for cloud logging and mobile app notifications.

Include additional sensors such as temperature (DHT11/DHT22) for better risk assessment.

Add battery backup and enclosure for real deployment.

Repository Structure
Example structure of this repository:​

Fire and Smoke Detection System with GSM AI... – Project report document.

Fire,Smoke with Alarm using gsm.ino.txt – Arduino source code (rename to .ino if required).

components connection.jpg – Hardware connections.

components connections with serial monitor.jpg – Setup image with Serial Monitor.

overall design.jpg – Assembled prototype.

serial monitor output.jpg – Screenshot of serial readings.

wiring design.jpg – Wiring diagram.

LICENSE – MIT License file.

README.md – This documentation.

License
This project is released under the MIT License. You are free to use, modify, and distribute this code and documentation with appropriate attribution.​
