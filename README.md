MediPulse is an end-to-end Embedded Systems–based Remote Patient Monitoring (RPM) platform that continuously tracks a patient's heart rate, blood oxygen saturation (SpO2), and body activity/falls, and streams that data in real time to a cloud-connected clinical dashboard. It combines a sub-$5 ESP8266 microcontroller and medical-grade biosensors with a Node.js/Express/MongoDB backend and a live web dashboard, delivering hospital-grade monitoring capability at a fraction of the cost of commercial patient monitors. The system is built with resilience in mind — if the pulse oximeter fails, it keeps transmitting activity and fall-detection data instead of halting entirely.


✨ Features
📡 Continuous Vital Monitoring — Tracks heart rate (BPM) and SpO2 (%) using PPG-based sensing every 3 seconds.
⚡ Real-Time Dashboard Updates — WebSocket (Socket.IO) push delivers sensor data to the browser in under 500ms, no polling required.
🤸 Activity & Fall Detection — Computes acceleration vector magnitude to classify activity level and flag falls (M > 3g).
🖥️ On-Device LCD Feedback — 16x2 I2C LCD shows patient info, sensor status, and live readings at the bedside.
🛡️ Graceful Sensor Degradation — If the MAX30102 pulse oximeter isn't detected, the system keeps transmitting accelerometer data instead of halting.
📉 Signal Smoothing — A 5-sample moving average filter reduces noise and motion artefacts in HR/SpO2 readings.
🚨 Automated Risk Scoring & Alerts — A multi-parameter algorithm computes a 0–100 risk score and classifies patients as Normal, Warning, or Critical.
📧 Guardian Email Reports — Nodemailer sends structured HTML health reports to caregivers on demand or on critical status change.
📄 Downloadable PDF Reports — jsPDF + html2canvas generate multi-page clinical reports including live chart snapshots.
🔐 Authentication System — Sign-in/sign-up flow with role-based access (Doctor, Admin, Nurse) and password-strength validation.
📊 Analytics & Historical Trends — 24-hour history view with average/max/min HR & SpO2 per patient, plus fleet-wide analytics.
👥 Multi-Patient Dashboard — Live patient cards, active alerts panel, and a real-time chronological feed of incoming events.


🧰 Tech Stack
Hardware
ESP8266 NodeMCU v3 (Wi-Fi microcontroller)
MAX30102 Pulse Oximeter & Heart-Rate Sensor
MPU6050 6-Axis Accelerometer/Gyroscope (accelerometer used)
16x2 I2C LCD (PCF8574 backpack)
breadboard, jumper wires


Programming Languages
C++ (ESP8266 Firmware, Arduino framework)
JavaScript (Backend & Frontend)
HTML / CSS

Communication Protocols
I2C (sensor bus)
Wi-Fi / HTTP POST (ESP8266 → Server)
WebSocket via Socket.IO (Server → Dashboard)
SMTP (email alerts via Nodemailer)

Backend
Node.js
Express.js (REST API)
Socket.IO (real-time server)
Mongoose (MongoDB ODM)
Nodemailer (email delivery)
CORS, body-parsing middleware

Frontend
Vanilla JavaScript, HTML, CSS
Chart.js — live waveform & analytics charts
jsPDF + html2canvas — client-side PDF report generation
LocalStorage-based session/auth persistence

Database
MongoDB Community Server (NoSQL)
Libraries (Firmware)
MAX30105 Arduino Library (pulse oximeter driver)
MPU6050 Arduino Library
LiquidCrystal_I2C
Wire.h (I2C)
ESP8266HTTPClient
ESP8266WiFi


Tools
Arduino IDE 2.x
MongoDB Compass
Postman (API testing)
Git/GitHub

🔩 Hardware Components
ComponentPurposeESP8266 NodeMCU v3Main microcontroller and Wi-Fi gatewayMAX30102 Sensor ModuleHeart rate (HR) and SpO2 sensing via PPGMPU6050 ModuleActivity classification and fall detection16x2 I2C LCD (PCF8574)Local, on-device patient feedback4.7kΩ Resistors (×2)I2C bus pull-ups (SDA, SCL)Breadboard + Jumper WiresPrototyping and interconnectsMicro-USB Power Supply (5V/1A)Power for the NodeMCUPC / LaptopProgramming and running the backend server


🏗️ Software Architecture
MediPulse follows a three-tier architecture:
Perception Layer (ESP8266 + Sensors) — The ESP8266 reads raw data from the MAX30102 and MPU6050 over I2C, applies moving-average smoothing and the Maxim SpO2 algorithm, and shows status on the LCD. It then packages the readings into a JSON payload and sends it via HTTP POST to the backend every 3 seconds.
Processing & Storage Layer (Node.js + MongoDB) — Express.js receives and validates each POST request, persists it to MongoDB via Mongoose, computes a risk score/status/insight, and immediately emits a sensor_update event to all connected dashboard clients over Socket.IO. It also exposes REST endpoints for patient data, history, notes, stats, and email reporting via Nodemailer.
Presentation Layer (Browser Dashboard) — A single-page dashboard authenticates the clinician, subscribes to WebSocket updates, renders live Chart.js waveforms, computes/display risk alerts, and lets users generate PDF reports or email guardians.
The system is designed for graceful degradation: if the MAX30102 fails to initialize, the firmware sets a flag and continues transmitting accelerometer-only data (HR/SpO2 = 0) rather than halting, and the dashboard renders – for missing vitals.


🔄 System Workflow
ESP8266 boots, initializes the I2C bus, LCD, and connects to Wi-Fi; time is synced via NTP.
Clinician registers a patient (name, age, condition, guardian email) via the Serial Monitor.
Firmware attempts to initialize the MAX30102 and MPU6050, setting sensorMAX_ok / sensorMPU_ok flags — no sensor failure halts execution.
In the main loop, the firmware checks for finger presence (IR threshold), collects 50 red/IR samples, and runs the Maxim SpO2 algorithm to compute HR & SpO2 (or reads accelerometer only if MAX30102 is absent).
Readings are pushed through a 5-sample moving-average filter to smooth out noise.
The firmware builds a JSON payload and sends it via HTTP POST to the Node.js server's /sensor endpoint every 3 seconds.
The server validates the payload, saves it to MongoDB, computes a risk score/status, and emits a sensor_update WebSocket event.
The browser dashboard receives the event and updates patient cards, live charts, and alert panels in real time — no page refresh required.
Clinicians can view 24-hour trends/analytics, write doctor notes, generate a downloadable PDF report, or trigger an automated HTML email report to the guardian.
The LCD continuously reflects current status (readings, connection state, alerts) at the bedside.


📁 Folder Structure

MediPulse/
├── firmware/
│   └── ESP8266_MediPulse/
│       └── ESP8266_MediPulse.ino        # ESP8266 C++ firmware
├── server/
│   ├── server.js                        # Express + Socket.IO + Mongoose backend
│   ├── package.json
│   └── public/                          # Static files served by Express
│       └── index.html                   # Dashboard (HTML/CSS/JS)
├── docs/
│   ├── Patient_monitor_report.pdf       # Full project report
│   └── images/
│       ├── circuit_diagram.png
│       └── demonstration.png
├── .env.example                         # Environment variable template
├── .gitignore
└── README.md


Note: The dashboard frontend, server.js, and firmware sketch are currently maintained as separate source files/documents; organize them per the structure above when setting up the repository.



⚙️ Installation

1. Clone the repository

bashgit clone https://github.com/<your-username>/MediPulse.git
cd MediPulse

2. Install backend dependencies

bashcd server
npm install express mongoose cors socket.io nodemailer

3. Configure environment variables

Create a .env (or edit the config section in server.js) with your own values — do not commit real credentials:

MONGODB_URI=mongodb://127.0.0.1:27017/patient_monitoring
EMAIL_USER=your-email@gmail.com
EMAIL_APP_PASSWORD=your-16-char-gmail-app-password
PORT=3000

4. Start MongoDB

Ensure MongoDB Community Server is running locally on the default port (27017), or point MONGODB_URI to your own instance.

5. Run the backend server

bashnode server.js

The server starts on http://localhost:3000 and serves the dashboard from the public/ folder.

6. Flash the ESP8266 firmware


Open firmware/ESP8266_MediPulse/ESP8266_MediPulse.ino in the Arduino IDE.
Install the ESP8266 board package and the following libraries via Library Manager: MAX30105lib, MPU6050, LiquidCrystal_I2C.
Update the following in the sketch:

cpp   const char* ssid     = "YOUR_WIFI_SSID";
   const char* password = "YOUR_WIFI_PASSWORD";
   const char* SERVER_IP = "YOUR_SERVER_LOCAL_IP";


Select NodeMCU 1.0 (ESP-12E Module) as the board and the correct COM port.
Upload the sketch.


7. Open the dashboard
Navigate to http://localhost:3000 in your browser and sign in (demo accounts: doctor / doctor123, admin / admin123).


▶️ Usage
Power on the ESP8266 with sensors wired per the circuit connections below.
Register a patient via the Arduino Serial Monitor (name, age, condition, guardian email).
Once Wi-Fi connects, the device begins transmitting readings every 3 seconds; the LCD shows live HR/SpO2.
Open the dashboard and sign in — the new patient appears automatically as data arrives.
Click into a patient card to view live waveforms, activity/fall status, risk score, and analytics.
Use Generate Report to download a PDF, or Email Guardian to send an automated HTML report.
Use the Alerts, Live Feed, and Analytics side panels to monitor multiple patients at once.



🔌 Circuit Connections
ESP8266 PinI2C SignalConnected ToNotesD2 (GPIO4)SDAMAX30102, MPU6050, LCD (shared bus)4.7kΩ pull-up to 3.3V requiredD1 (GPIO5)SCLMAX30102, MPU6050, LCD (shared bus)4.7kΩ pull-up to 3.3V required3.3VVCCMAX30102, MPU6050MAX30102 must not use 5V5V (VIN)VCCLCDLCD module is 5V-tolerantGNDGNDAll modulesCommon ground for all components


🌐 API Endpoints
MethodEndpointDescriptionKey LogicPOST/sensorReceives ESP8266 sensor dataSaves to DB, computes risk score, emits WebSocket eventGET/api/patientsGets all unique patients (latest readings)MongoDB $group aggregate, sorted by risk scoreGET/api/patients/:nameGets patient details + 24h analyticsLatest record + avg/max/min computationGET/api/patients/:name/historyGets 24-hour history (up to 500 records)Sorted by createdAt, used to seed chartsGET / POST/api/patients/:name/notesGets/saves doctor notesStored in doctornotes collectionGET/api/statsAggregate stats: total, critical, warning, normalUsed for dashboard stat cardsPOST/api/patients/:name/sendreportSends HTML email report to guardianNodemailer + HTML template + risk data



🚀 Future Improvements
Miniaturize the system into a wearable form factor.
Add more physiological sensors (ECG, body temperature).
Integrate machine learning for early anomaly/deterioration prediction.
Add HTTPS and end-to-end encryption for secure data transmission.
Integrate with hospital Electronic Medical Record (EMR) systems.
Scale for multi-hospital, cloud-based deployment (Docker + Nginx reverse proxy).
Add voice alerts and telemedicine features such as video consultations.



🏆 Project Outcomes
Delivered a fully functional, low-cost (~₹800–1,200) remote patient monitoring prototype — 100–500× cheaper than commercial monitors.
Achieved SpO2 accuracy within ±1.5% and heart rate accuracy within ±3 BPM against a clinical reference oximeter.
Achieved end-to-end sensor-to-dashboard latency of ~380–400ms, well under the 1-second target.
Verified reliable fall detection (M > 3g) across all controlled drop tests with no false positives during normal activity.
Validated graceful degradation: the system continued operating and transmitting data when the pulse oximeter was disconnected.
Delivered a production-style software stack: authentication, real-time WebSocket updates, automated email alerts, and PDF reporting.



🧠 Skills Demonstrated
Embedded firmware development in C++ for ESP8266 (Arduino framework)
I2C sensor interfacing (multi-device shared bus) and signal processing (PPG-based SpO2/HR algorithms)
Real-time systems design with resilience/fault-tolerance (graceful degradation)
RESTful API design with Express.js and MongoDB/Mongoose schema modeling
Real-time bidirectional communication using WebSockets (Socket.IO)
Frontend engineering: live data visualization (Chart.js), authentication flows, and PDF generation (jsPDF/html2canvas)
End-to-end IoT system integration across hardware, backend, and frontend layers
Automated notification systems (SMTP email integration via Nodemailer)
Testing and validation methodology (unit, integration, latency, and accuracy testing)



👨‍💻 Author
Name:Mohanish Gunda
