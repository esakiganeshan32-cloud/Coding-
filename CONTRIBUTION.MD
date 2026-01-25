Team Name : Electroff
Team Members : Bharanidharan R Esakiganeshan R

Domains : Electrical Power systems with IoT

Idea : 
Problem:
Power distribution networks, especially in urban and semi-urban areas, face frequent hazards such as wire breakages, fallen poles, and abnormal current conditions. These faults pose serious risks including electrocution, fire hazards, prolonged power outages, and delays in maintenance response. Traditional monitoring methods rely heavily on manual inspection or delayed reporting, leading to increased downtime, safety hazards, and reduced reliability of the electricity supply.
There is a critical need for an automated, real-time monitoring and auto power cut off power line safety system that can detect faults immediately, isolate the affected line safely, and notify maintenance teams promptly. Such a system must also ensure continuous monitoring even during power outages and provide actionable information to electricity board staff to improve response time and reduce hazards.

Proposed Solution:
It finds a break in the wire between the poles and sometimes the pole will fall due to disaster. Once detected, it automatically shutdown the power in that section and immediately sends an SMS to the EB using Edge level processing for quick action.

Working:

Assume the transformer as the Receiver Pole (Rx) and the street pole as the Transmitter pole (Tx).
If the wire is cut, the Current sensor at Tx detects no current and sends this to the microcontroller.
The microcontroller sends signals through LoRa to the Rx pole(Transformer), which then gives outputs.
Outputs are: SMS alert, display on monitor, and MCCB to switch off the faulty line/Servo mech to cut the power line.
Additionally, A tilt sensor detects pole fall, the microcontroller cuts current, and power backup is given by a rechargeable battery from the Output transformer line via AC to DC.

Tech Stack Used : 
Hardware:
CT sensors
Voltage sensors
Tilt sensor
Microcontroller (ESP32)
LoRa modules (Tx & Rx)
Relay driver circuit
MCCB
Servomotor mechanism
GSM
Buzzer
Display (16×2)
Reset button (push)
Rechargeable Battery

Software:
Embedded C/C++
SMS/IoT dashboard (Blynk) integration for EB alerts
Edge processing for faster alert response

Implementation Process:
At the TX pole, voltage, current, and tilt sensors are connected to the power line and pole.
The sensor outputs are connected to the TX side microcontroller for continuous monitoring.
The microcontroller processes the sensor data and sends it to the LoRa module.
LoRa communication transmits the signal from the TX pole to the RX unit.
The RX-side microcontroller receives and analyzes the transmitted signal.
If a wire break, pole fall, or abnormal current condition is detected, the Servo mechanism or MCCB trips automatically to cut the power supply.
A buzzer and emergency warning light are activated to indicate the fault condition locally.
System status and fault information are displayed on the LCD screen.
Continuous monitoring is carried out at the EB office using the loT based Blynk dashboard.
The GSM module sends SMS alerts to EB officials for immediate notification.
A manual reset button is provided to restore the system after fault clearance.
The microcontrollers are powered using rechargeable batteries.
The batteries are charged through a regulated power supply connected to the local transformer.

Drive link Project Related Images and Videos : https://drive.google.com/drive/folders/1_kfd6snUCCmo6WxGQ4AtU_htMoNNkEUu

Project Details ( PPT ) :  https://drive.google.com/file/d/1PHhTZYn_yFlHoR15EbnlWWKF1FdXx4dV/view?usp=drivesdk

Code : Using IDE SOFTWARE TO EXECUTE THE CODE 
TX CODING
#include <Arduino.h>

// ---- Pin Setup ----
#define VOLTAGE_PIN 32
#define CURRENT_PIN 35   // ACS712 analog pin
#define TILT_PIN 34

// LoRa / HC-12 (SI4463)
#define LORA_TX 17
#define LORA_RX 16
HardwareSerial LORA(2);

// ---- ACS712 Calibration ----
int ACS_OFFSET = 2048;

void setup() {
  Serial.begin(115200);
  LORA.begin(9600, SERIAL_8N1, LORA_RX, LORA_TX);
  Serial.println("TX Node Ready...");
}

void loop() {

  int voltageVal = analogRead(VOLTAGE_PIN);

  int rawCurrent = analogRead(CURRENT_PIN);
  int currentStatus = (rawCurrent <= ACS_OFFSET + 50);

  int tiltRaw = analogRead(TILT_PIN);
  int tiltVal = (tiltRaw > 4000);

  String data = String(voltageVal) + "," + String(tiltVal) + "," + String(currentStatus);
  LORA.println(data);

  Serial.print("Voltage=");
  Serial.print(voltageVal);
  Serial.print(" | TiltRaw=");
  Serial.print(tiltRaw);
  Serial.print(" | CurrentRaw=");
  Serial.print(rawCurrent);
  Serial.print(" | TX Sent → ");
  Serial.println(data);

  delay(2000);
}





RX CODING 

#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <ESP32Servo.h>

// ---- Pin Setup ----
#define VOLTAGE_PIN 32
#define CURRENT_PIN 35
#define TILT_PIN 34

#define LORA_TX 25
#define LORA_RX 26
#define GSM_TX 17
#define GSM_RX 16

#define SERVO_PIN 33
#define BUZZER_PIN 27
#define BUTTON_PIN 13

// ---- Thresholds ----
#define VOLTAGE_THRESHOLD 10
#define TILT_THRESHOLD 2000
#define ACS_OFFSET 2048
#define CURRENT_MARGIN 50   // near-zero current range

LiquidCrystal_I2C lcd(0x27, 16, 2);
HardwareSerial LORA(1);
HardwareSerial GSM(2);
Servo gateServo;

bool emergencyActive = false;

void setup() {
  Serial.begin(115200);

  LORA.begin(9600, SERIAL_8N1, LORA_RX, LORA_TX);
  GSM.begin(9600, SERIAL_8N1, GSM_RX, GSM_TX);

  lcd.init();
  lcd.backlight();
  lcd.clear();
  lcd.print("RX Node Ready");
  delay(1500);
  lcd.clear();

  pinMode(BUTTON_PIN, INPUT_PULLUP);
  pinMode(BUZZER_PIN, OUTPUT);

  gateServo.attach(SERVO_PIN);
  gateServo.write(0);
}

void loop() {

  if (digitalRead(BUTTON_PIN) == LOW) {
    emergencyActive = false;
    gateServo.write(0);
    digitalWrite(BUZZER_PIN, LOW);
    lcd.clear();
    lcd.print("System Reset");
    delay(1500);
    lcd.clear();
  }

  int localVoltage = analogRead(VOLTAGE_PIN);

  int tiltRaw = analogRead(TILT_PIN);
  int localTilt = (tiltRaw > TILT_THRESHOLD);

  int currentRaw = analogRead(CURRENT_PIN);
  int localCurrent = (currentRaw <= ACS_OFFSET + CURRENT_MARGIN); // 1 = NO CURRENT

  int remoteVoltage = localVoltage;
  int remoteTilt = localTilt;
  int remoteCurrent = localCurrent;

  if (LORA.available()) {
    String received = LORA.readStringUntil('\n');
    received.trim();

    int c1 = received.indexOf(',');
    int c2 = received.lastIndexOf(',');

    if (c1 > 0 && c2 > c1) {
      remoteVoltage = received.substring(0, c1).toInt();
      remoteTilt = received.substring(c1 + 1, c2).toInt();
      remoteCurrent = received.substring(c2 + 1).toInt();
    }
  }

  lcd.setCursor(0, 0);
  lcd.print("V:");
  lcd.print(remoteVoltage);
  lcd.print(" C:");
  lcd.print(remoteCurrent);
  lcd.print("   ");

  lcd.setCursor(0, 1);
  lcd.print("Tilt:");
  lcd.print(remoteTilt);
  lcd.print("   ");

  if (!emergencyActive) {

    if ((remoteTilt == 1 || localTilt == 1) &&
        (remoteVoltage <= VOLTAGE_THRESHOLD || localVoltage <= VOLTAGE_THRESHOLD) &&
        (remoteCurrent == 1 || localCurrent == 1)) {
      triggerEmergency("WIRE+POLE+CUR");
    }

    else if (remoteCurrent == 1 || localCurrent == 1) {
      triggerEmergency("NO CURRENT");
    }

    else if (remoteVoltage <= VOLTAGE_THRESHOLD || localVoltage <= VOLTAGE_THRESHOLD) {
      triggerEmergency("WIRE BREAK");
    }

    else if (remoteTilt == 1 || localTilt == 1) {
      triggerEmergency("POLE FALL");
    }
  }

  delay(500);
}

void triggerEmergency(String msg) {
  emergencyActive = true;

  lcd.clear();
  lcd.print("EMERGENCY!");
  lcd.setCursor(0, 1);
  lcd.print(msg);

  gateServo.write(60);
  digitalWrite(BUZZER_PIN, HIGH);

  Serial.println("Emergency: " + msg);
  sendSMS(msg);
}

void sendSMS(String text) {
  GSM.println("AT+CMGF=1");
  delay(500);
  GSM.println("AT+CMGS=\"+919345385535\"");
  delay(500);
  GSM.print(text);
  delay(500);
  GSM.write(26);
  delay(2000);
}
