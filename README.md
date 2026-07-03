# 🫀 MediPulse

### End-to-End Embedded Remote Patient Monitoring (RPM) Platform


MediPulse continuously tracks a patient's **heart rate**, **blood oxygen saturation (SpO2)**, and **activity/fall status**, streaming that data in real time to a cloud-connected clinical dashboard — delivering a fraction of the cost of commercial patient monitors, without sacrificing reliability.

---

## 📖 Overview

**MediPulse** is an end-to-end Embedded Systems–based Remote Patient Monitoring platform. It combines a low-cost **ESP8266 microcontroller** with **medical-grade biosensors**, a **Node.js/Express/MongoDB** backend, and a **live web dashboard** — bringing real-time, hospital-grade monitoring capability to a fraction of the cost of commercial devices.

The system is engineered for **resilience**: if the pulse oximeter fails, it keeps transmitting activity and fall-detection data instead of halting entirely.

---

## ✨ Features

| | |
|---|---|
| 📡 **Continuous Vital Monitoring** | Tracks heart rate (BPM) and SpO2 (%) via PPG-based sensing every 3 seconds |
| ⚡ **Real-Time Dashboard Updates** | WebSocket (Socket.IO) push delivers sensor data in **under 500ms** — no polling |
| 🤸 **Activity & Fall Detection** | Computes acceleration vector magnitude to classify activity and flag falls (M > 3g) |
| 🖥️ **On-Device LCD Feedback** | 16×2 I2C LCD shows patient info, sensor status, and live readings at the bedside |
| 🛡️ **Graceful Sensor Degradation** | Keeps transmitting accelerometer data if the MAX30102 isn't detected, instead of halting |
| 📉 **Signal Smoothing** | 5-sample moving average filter reduces noise and motion artefacts |
| 🚨 **Automated Risk Scoring & Alerts** | Multi-parameter algorithm computes a 0–100 risk score → Normal / Warning / Critical |
| 📧 **Guardian Email Reports** | Nodemailer sends structured HTML health reports on demand or on critical status change |
| 📄 **Downloadable PDF Reports** | jsPDF + html2canvas generate multi-page clinical reports with live chart snapshots |
| 🔐 **Authentication System** | Sign-in/sign-up with role-based access (Doctor, Admin, Nurse) + password strength validation |
| 📊 **Analytics & Historical Trends** | 24-hour history view with avg/max/min HR & SpO2 per patient, plus fleet-wide analytics |
| 👥 **Multi-Patient Dashboard** | Live patient cards, active alerts panel, and a real-time chronological event feed |

---

## 🧰 Tech Stack

### Hardware
- **ESP8266 NodeMCU v3** — Wi-Fi microcontroller
- **MAX30102** — Pulse oximeter & heart-rate sensor
- **MPU6050** — 6-axis accelerometer/gyroscope (accelerometer used)
- **16×2 I2C LCD** (PCF8574 backpack)
- Breadboard, jumper wires

### Programming Languages
- **C++** — ESP8266 firmware (Arduino framework)
- **JavaScript** — Backend & frontend
- **HTML / CSS**

### Communication Protocols
- **I2C** — Sensor bus
- **Wi-Fi / HTTP POST** — ESP8266 → Server
- **WebSocket (Socket.IO)** — Server → Dashboard
- **SMTP** — Email alerts via Nodemailer

### Backend
- Node.js · Express.js (REST API) · Socket.IO · Mongoose (MongoDB ODM) · Nodemailer · CORS / body-parsing middleware

### Frontend
- Vanilla JavaScript, HTML, CSS
- **Chart.js** — live waveform & analytics charts
- **jsPDF + html2canvas** — client-side PDF report generation
- LocalStorage-based session/auth persistence

### Database
- **MongoDB Community Server** (NoSQL)

### Firmware Libraries
- MAX30105 Arduino Library (pulse oximeter driver)
- MPU6050 Arduino Library
- LiquidCrystal_I2C
- Wire.h (I2C)
- ESP8266HTTPClient
- ESP8266WiFi

### Tools
- Arduino IDE 2.x · MongoDB Compass · Postman · Git/GitHub

---

## 🔩 Hardware Components

| Component | Purpose |
|---|---|
| **ESP8266 NodeMCU v3** | Main microcontroller and Wi-Fi gateway |
| **MAX30102 Sensor Module** | Heart rate (HR) and SpO2 sensing via PPG |
| **MPU6050 Module** | Activity classification and fall detection |
| **16×2 I2C LCD (PCF8574)** | Local, on-device patient feedback |
| **4.7kΩ Resistors (×2)** | I2C bus pull-ups (SDA, SCL) |
| **Breadboard + Jumper Wires** | Prototyping and interconnects |
| **Micro-USB Power Supply (5V/1A)** | Power for the NodeMCU |
| **PC / Laptop** | Programming and running the backend server |

---

## 🏗️ Software Architecture

MediPulse follows a **three-tier architecture**:

1. **Perception Layer** *(ESP8266 + Sensors)*
   Reads raw data from the MAX30102 and MPU6050 over I2C, applies moving-average smoothing and the Maxim SpO2 algorithm, and shows status on the LCD. Packages readings into JSON and sends them via HTTP POST to the backend every 3 seconds.

2. **Processing & Storage Layer** *(Node.js + MongoDB)*
   Express.js receives and validates each POST request, persists it to MongoDB via Mongoose, computes a risk score/status/insight, and immediately emits a `sensor_update` event to all connected dashboard clients over Socket.IO. Also exposes REST endpoints for patient data, history, notes, stats, and email reporting.

3. **Presentation Layer** *(Browser Dashboard)*
   A single-page dashboard authenticates the clinician, subscribes to WebSocket updates, renders live Chart.js waveforms, computes/displays risk alerts, and lets users generate PDF reports or email guardians.

> **Graceful degradation by design:** if the MAX30102 fails to initialize, the firmware sets a flag and continues transmitting accelerometer-only data (HR/SpO2 = 0) rather than halting — the dashboard renders `–` for missing vitals.

---

## 🔄 System Workflow

1. ESP8266 boots, initializes the I2C bus, LCD, and connects to Wi-Fi; time is synced via NTP.
2. Clinician registers a patient (name, age, condition, guardian email) via the Serial Monitor.
3. Firmware attempts to initialize the MAX30102 and MPU6050, setting `sensorMAX_ok` / `sensorMPU_ok` flags — no sensor failure halts execution.
4. In the main loop, the firmware checks for finger presence (IR threshold), collects 50 red/IR samples, and runs the Maxim SpO2 algorithm to compute HR & SpO2 (or reads accelerometer only if MAX30102 is absent).
5. Readings pass through a 5-sample moving-average filter to smooth out noise.
6. The firmware builds a JSON payload and POSTs it to the server's `/sensor` endpoint every 3 seconds.
7. The server validates the payload, saves it to MongoDB, computes a risk score/status, and emits a `sensor_update` WebSocket event.
8. The browser dashboard receives the event and updates patient cards, live charts, and alert panels in real time — no page refresh required.
9. Clinicians can view 24-hour trends/analytics, write doctor notes, generate a downloadable PDF report, or trigger an automated HTML email report to the guardian.
10. The LCD continuously reflects current status (readings, connection state, alerts) at the bedside.

---

## 📁 Folder Structure

```
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
```

> **Note:** The dashboard frontend, `server.js`, and firmware sketch are currently maintained as separate source files/documents — organize them per the structure above when setting up the repository.

---

## ⚙️ Installation

### 1. Clone the repository
```bash
git clone https://github.com/<your-username>/MediPulse.git
cd MediPulse
```

### 2. Install backend dependencies
```bash
cd server
npm install express mongoose cors socket.io nodemailer
```

### 3. Configure environment variables
Create a `.env` (or edit the config section in `server.js`) with your own values — **do not commit real credentials**:

```env
MONGODB_URI=mongodb://127.0.0.1:27017/patient_monitoring
EMAIL_USER=your-email@gmail.com
EMAIL_APP_PASSWORD=your-16-char-gmail-app-password
PORT=3000
```

### 4. Start MongoDB
Ensure MongoDB Community Server is running locally on the default port (`27017`), or point `MONGODB_URI` to your own instance.

### 5. Run the backend server
```bash
node server.js
```
The server starts on **http://localhost:3000** and serves the dashboard from the `public/` folder.

### 6. Flash the ESP8266 firmware
1. Open `firmware/ESP8266_MediPulse/ESP8266_MediPulse.ino` in the Arduino IDE.
2. Install the ESP8266 board package and the following libraries via Library Manager: `MAX30105lib`, `MPU6050`, `LiquidCrystal_I2C`.
3. Update the following in the sketch:
   ```cpp
   const char* ssid     = "YOUR_WIFI_SSID";
   const char* password = "YOUR_WIFI_PASSWORD";
   const char* SERVER_IP = "YOUR_SERVER_LOCAL_IP";
   ```
4. Select **NodeMCU 1.0 (ESP-12E Module)** as the board and the correct COM port.
5. Upload the sketch.

### 7. Open the dashboard
Navigate to **http://localhost:3000** in your browser and sign in.

**Demo accounts:** `doctor / doctor123` · `admin / admin123`

---

## ▶️ Usage

1. Power on the ESP8266 with sensors wired per the [circuit connections](#-circuit-connections) below.
2. Register a patient via the Arduino Serial Monitor (name, age, condition, guardian email).
3. Once Wi-Fi connects, the device begins transmitting readings every 3 seconds; the LCD shows live HR/SpO2.
4. Open the dashboard and sign in — the new patient appears automatically as data arrives.
5. Click into a patient card to view live waveforms, activity/fall status, risk score, and analytics.
6. Use **Generate Report** to download a PDF, or **Email Guardian** to send an automated HTML report.
7. Use the **Alerts**, **Live Feed**, and **Analytics** side panels to monitor multiple patients at once.

---

## 🔌 Circuit Connections

| ESP8266 Pin | I2C Signal | Connected To | Notes |
|---|---|---|---|
| **D2 (GPIO4)** | SDA | MAX30102, MPU6050, LCD (shared bus) | 4.7kΩ pull-up to 3.3V required |
| **D1 (GPIO5)** | SCL | MAX30102, MPU6050, LCD (shared bus) | 4.7kΩ pull-up to 3.3V required |
| **3.3V** | VCC | MAX30102, MPU6050 | MAX30102 **must not** use 5V |
| **5V (VIN)** | VCC | LCD | LCD module is 5V-tolerant |
| **GND** | GND | All modules | Common ground for all components |

---

## 🌐 API Endpoints

| Method | Endpoint | Description | Key Logic |
|---|---|---|---|
| `POST` | `/sensor` | Receives ESP8266 sensor data | Saves to DB, computes risk score, emits WebSocket event |
| `GET` | `/api/patients` | Gets all unique patients (latest readings) | MongoDB `$group` aggregate, sorted by risk score |
| `GET` | `/api/patients/:name` | Gets patient details + 24h analytics | Latest record + avg/max/min computation |
| `GET` | `/api/patients/:name/history` | Gets 24-hour history (up to 500 records) | Sorted by `createdAt`, used to seed charts |
| `GET` / `POST` | `/api/patients/:name/notes` | Gets/saves doctor notes | Stored in `doctornotes` collection |
| `GET` | `/api/stats` | Aggregate stats: total, critical, warning, normal | Used for dashboard stat cards |
| `POST` | `/api/patients/:name/sendreport` | Sends HTML email report to guardian | Nodemailer + HTML template + risk data |

---

## 🏆 Project Outcomes

- ✅ Delivered a fully functional, low-cost (**~₹800–1,200**) remote patient monitoring prototype — **100–500× cheaper** than commercial monitors.
- ✅ Achieved **SpO2 accuracy within ±1.5%** and **heart rate accuracy within ±3 BPM** against a clinical reference oximeter.
- ✅ Achieved end-to-end sensor-to-dashboard latency of **~380–400ms**, well under the 1-second target.
- ✅ Verified reliable fall detection (M > 3g) across all controlled drop tests with **no false positives** during normal activity.
- ✅ Validated graceful degradation: the system continued operating and transmitting data when the pulse oximeter was disconnected.
- ✅ Delivered a production-style software stack: authentication, real-time WebSocket updates, automated email alerts, and PDF reporting.

---

## 🚀 Future Improvements

- [ ] Miniaturize the system into a wearable form factor
- [ ] Add more physiological sensors (ECG, body temperature)
- [ ] Integrate machine learning for early anomaly/deterioration prediction
- [ ] Add HTTPS and end-to-end encryption for secure data transmission
- [ ] Integrate with hospital Electronic Medical Record (EMR) systems
- [ ] Scale for multi-hospital, cloud-based deployment (Docker + Nginx reverse proxy)
- [ ] Add voice alerts and telemedicine features such as video consultations

---

## 🧠 Skills Demonstrated

- Embedded firmware development in **C++** for ESP8266 (Arduino framework)
- I2C sensor interfacing (multi-device shared bus) and signal processing (PPG-based SpO2/HR algorithms)
- Real-time systems design with resilience/fault-tolerance (graceful degradation)
- RESTful API design with **Express.js** and MongoDB/Mongoose schema modeling
- Real-time bidirectional communication using **WebSockets** (Socket.IO)
- Frontend engineering: live data visualization (Chart.js), authentication flows, and PDF generation (jsPDF/html2canvas)
- End-to-end IoT system integration across hardware, backend, and frontend layers
- Automated notification systems (SMTP email integration via Nodemailer)
- Testing and validation methodology (unit, integration, latency, and accuracy testing)

---

## 👨‍💻 Author

**Mohanish Gunda**

⭐ *If you found this project useful, consider giving it a star on GitHub!*
