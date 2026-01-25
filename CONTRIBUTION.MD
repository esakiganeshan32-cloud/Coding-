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

Code : 
Tx code
