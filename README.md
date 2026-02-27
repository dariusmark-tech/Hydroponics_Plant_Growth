# 🌱 AI-Powered Hydroponics Plant Growth Monitoring System

An automated plant growth monitoring system that uses an **ESP32-CAM**, **Roboflow AI (YOLOv8)**, and a **Python Flask** web dashboard to track Kangkong growth stages daily — fully hands-free.

---

## 📸 Overview

The system captures a photo every morning at 8:00 AM, sends it to a Flask server, runs it through a trained Roboflow computer vision model, logs the results to a CSV, and displays everything on a live web dashboard with smart harvest alerts.

```
ESP32-CAM → Flask Server → Roboflow AI → CSV Log → Web Dashboard
```

---

## 🛠️ Tech Stack

| Component | Technology |
|-----------|-----------|
| Camera Module | ESP32-CAM |
| AI / Object Detection | Roboflow + YOLOv8n |
| Backend Server | Python Flask |
| Frontend Dashboard | HTML/CSS (Jinja2 templates) |
| Data Storage | CSV flat file |
| Time Sync | NTP (UTC+8, Philippines) |

---

## 🌿 Growth Stages Tracked

| Stage | Estimated Days | Health Status |
|-------|---------------|---------------|
| Germination | Day 1–3 | Healthy |
| Seedling | Day 4–10 | Healthy |
| Vegetative | Day 11–25 | Healthy |
| Mature | Day 26+ | Harvest Ready ✅ |
| Overgrown | Day 30+ | Attention Needed ⚠️ |
| Stressed | Any stage | Stressed 🔴 |

---

## 📁 Project Structure

```
hydroponics-monitor/
├── server.py                  # Flask backend & Roboflow integration
├── plant_growth_log.csv       # Auto-generated growth log
├── images/                    # Saved ESP32-CAM photos
├── templates/
│   └── dashboard.html         # Web dashboard UI
└── esp32/
    └── firmware.ino           # ESP32-CAM Arduino sketch
```

---

## ⚙️ System Architecture (6 Phases)

### Phase 1 — Configuration
Set your constants in both `server.py` and the ESP32 firmware:
- `WIFI_SSID` / `WIFI_PASSWORD`
- `SERVER_IP` / `SERVER_PORT` (default: `5000`)
- `ROBOFLOW_API_KEY`, `ROBOFLOW_MODEL_ID`, `ROBOFLOW_VERSION`
- `CAPTURE_HOUR` / `CAPTURE_MINUTE` (default: `08:00`)
- `GMT_OFFSET_SEC` = `28800` (UTC+8)

### Phase 2 — ESP32-CAM Daily Capture
On boot or deep-sleep wakeup, the ESP32-CAM:
1. Connects to Wi-Fi
2. Syncs time via NTP
3. At 8:00 AM, captures a JPEG (with 5-frame warmup)
4. POSTs the photo to `/upload` on the Flask server
5. Goes back to deep sleep until the next 8:00 AM

### Phase 3 — Roboflow AI Analysis
The server encodes the image as base64 and sends it to the Roboflow detection API. The highest-confidence bounding box prediction determines the current growth stage.

### Phase 4 — Data Logging (Flask)
Each upload triggers a new row appended to `plant_growth_log.csv` with fields: `date`, `time`, `growth_stage`, `health_status`, `confidence`, `image_filename`, and more.

### Phase 5 — Web Dashboard
Visit `http://<SERVER_IP>:5000` to see:
- Summary cards (total photos, current stage, confidence, health)
- Growth progress bar across all stages
- Latest photo preview
- Full sortable log table
- CSV download button
- Smart alert banners (harvest ready, overgrown, stressed)

### Phase 6 — Background Checks
- **Connectivity monitor**: Checks Wi-Fi status every 60 seconds; auto-reconnects and logs offline events
- **Growth alert monitor**: Continuously checks for Mature / Overgrown / Stressed stages
- **Daily CSV backup**: Runs at midnight, logs success or failure

---

## 🚀 Getting Started

### 1. Train Your Roboflow Model

1. Collect 50–100 photos per growth stage, taken from the same angle as your ESP32-CAM mount
2. Upload and annotate images in a new [Roboflow](https://roboflow.com) project
3. Apply preprocessing: `auto-orient`, `resize 640×640`
4. Apply augmentations: horizontal flip, ±15% brightness, blur up to 2px, ±10° rotation
5. Train with **YOLOv8n**
6. Copy your `API Key`, `Model ID`, and `Version` from the Roboflow dashboard

### 2. Set Up the Flask Server

```bash
pip install flask requests roboflow
python server.py
```

The server will start on `0.0.0.0:5000` and auto-create `images/` and `plant_growth_log.csv`.

### 3. Flash the ESP32-CAM

Open `esp32/firmware.ino` in Arduino IDE and fill in:

```cpp
const char* WIFI_SSID     = "YourNetworkName";
const char* WIFI_PASSWORD = "YourPassword";
const char* SERVER_IP     = "192.168.x.x";
const int   SERVER_PORT   = 5000;
```

Flash to your ESP32-CAM board. On first boot it will sync time and sleep until 8:00 AM.

---

## 📊 CSV Log Schema

```
date, time, timestamp, plant_species, device_id, filename,
growth_stage, estimated_day_range, health_status, confidence,
detected_class, all_detections, image_width, image_height,
roboflow_model, notes
```

---

## 🔔 Dashboard Alerts

| Condition | Alert |
|-----------|-------|
| Stage = Mature | 🟢 "Plant is ready to harvest!" |
| Stage = Overgrown | 🟠 "Past harvest point — harvest immediately" |
| Stage = Stressed | 🔴 "Stress detected — check plant conditions" |

---

## 📌 Notes

- **Plant species**: Kangkong *(Ipomoea aquatica)* — easily adaptable to other species by retraining the Roboflow model
- **Timezone**: Configured for UTC+8 (Philippines) — update `GMT_OFFSET_SEC` as needed
- **Camera**: Tested with AI-Thinker ESP32-CAM module
- **Multi-camera support**: Set a unique `DEVICE_ID` per camera (e.g., `cam_01`, `cam_02`)

---

## 📄 License

MIT License — feel free to use, modify, and share.

---

> Built with ❤️ for smart hydroponics farming.
