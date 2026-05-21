# Retail AI Agent

**Real-time Edge AI retail intelligence on NVIDIA Jetson AGX Xavier**

> Converts a live camera feed into actionable store metrics — fully local, no cloud required, no face recognition.

---

## What This Is

A production-style Edge AI pipeline for smart retail analytics. The system runs entirely on a Jetson AGX Xavier at the edge. It detects customers, tracks them anonymously, counts entries and exits, measures dwell time per zone, monitors queue formation, logs every event, and streams a live dashboard to any browser on the local network.

This is not a notebook demo. It runs on real hardware with a live camera.

---

## System Architecture

```
Logitech Brio 4K Camera (1280x720 @ 30 FPS)
        │
        ▼
YOLOv8n Person Detection (class 0 only, conf 0.35)
        │
        ▼
ByteTrack — Persistent Anonymous Customer IDs
        │
        ▼
Retail Analytics Engine
  ├── Entry / Exit Line Crossing
  ├── Zone Dwell Time (Promo Shelf, Checkout)
  ├── Queue Zone Monitoring
  └── JSONL Event Logging
        │
        ▼
FastAPI + Uvicorn Dashboard
  ├── /         → Live browser dashboard
  ├── /video    → MJPEG stream
  ├── /metrics  → Live JSON metrics
  ├── /events   → Recent event log
  └── /health   → Service health
        │
        ▼
Browser — accessible over LAN at http://<jetson-ip>:8000
```

---

## Hardware & Software Stack

| Layer | Detail |
|---|---|
| Edge Device | NVIDIA Jetson AGX Xavier 32GB |
| Camera | Logitech Brio 4K — used at 1280x720 MJPG |
| OS | Ubuntu 20.04.6 LTS, L4T R35.6.4 |
| CUDA | 11.4 |
| TensorRT | 8.5.2 (optimization: next phase) |
| Python | 3.8.10 |
| PyTorch | 2.1.0 (NVIDIA build) |
| OpenCV | 4.13.0 |
| Ultralytics | 8.4.48 |
| Tracker | ByteTrack via Ultralytics |
| API | FastAPI + Uvicorn |

---

## Project Structure

```
retail-ai-agent/
├── configs/
│   └── retail_agent.yaml       # All zones, lines, thresholds configured here
├── src/
│   ├── retail_agent.py         # Core pipeline: detection, tracking, analytics
│   └── retail_web.py           # FastAPI dashboard + MJPEG stream
├── scripts/
│   ├── test_camera.py          # Camera validation + calibration frame capture
│   └── benchmark_phase1.sh     # Repeatable benchmark runner
├── models/
│   └── yolov8n.pt
├── runs/
│   └── <run_dir>/
│       ├── final_metrics.json
│       ├── events.jsonl
│       └── retail_overlay.mp4
└── logs/
    └── benchmark_phase1.txt
```

---

## Metrics Output (Real-time)

| Metric | Description |
|---|---|
| Entry Count | Customers who crossed the entry line inbound |
| Exit Count | Customers who crossed the entry line outbound |
| Occupancy | Current people inside (Entry - Exit) |
| Active Tracks | Live ByteTrack IDs in frame |
| Queue Count | People detected in the checkout zone |
| Promo Dwell | Average seconds spent near promo shelf |
| Checkout Dwell | Average seconds spent near checkout |
| FPS | Live inference frame rate |

---

## Calibration

All zones and the entry/exit line are defined in `configs/retail_agent.yaml`. No code changes needed to recalibrate for a different camera angle or store layout.

**Key principle:** Analytics uses the **bottom-center (foot point)** of each bounding box. Zone polygons must cover the floor area where a person stands, not the shelf or wall.

```yaml
analytics:
  entry_line: [[180, 640], [1120, 640]]
  entry_transition: "positive_to_negative"
  exit_transition: "negative_to_positive"
  min_track_age_frames: 8
  crossing_cooldown_sec: 5.0

zones:
  promo_shelf:
    polygon: [[0, 575], [270, 560], [390, 720], [0, 720]]
    dwell_threshold_sec: 3.0
  checkout:
    polygon: [[1110, 585], [1280, 560], [1280, 720], [1115, 720]]
    dwell_threshold_sec: 3.0

queue:
  zone_name: "checkout"
  threshold_count: 2
  alert_after_sec: 5.0
```

---

## Running the Dashboard

```bash
cd /home/jetson/retail-ai-agent
source /home/jetson/vision-safety-officer/.venv/bin/activate

# Kill any existing session
pkill -f uvicorn || true

# Start
uvicorn src.retail_web:app --host 0.0.0.0 --port 8000
```

Open `http://<jetson-ip>:8000` from any browser on the same network.

---

## Baseline Performance

| Parameter | Value |
|---|---|
| Resolution | 1280 x 720 |
| Inference Size | 640px |
| Model | YOLOv8n |
| Tracker | ByteTrack |
| Runtime | PyTorch (CUDA) |
| Device | Jetson AGX Xavier |
| FPS | ~15 FPS (stable) |

---

## Roadmap

- [ ] TensorRT FP16 optimization
- [ ] INT8 calibration for higher throughput
- [ ] Heatmap generation from foot-point trails
- [ ] GenAI insight layer — auto-generate store summaries from JSONL metrics
- [ ] DeepStream pipeline integration
- [ ] Multi-camera support
- [ ] Cloud sync for aggregated historical metrics
- [ ] Docker / systemd production deployment

---

## Privacy Design

- No face recognition
- No identity storage
- Anonymous track IDs only
- No raw video leaves the device
- All processing is local on the Jetson

---

## Author

**Vishal Bhapkar**
Senior AI/ML Engineer — Edge AI | Computer Vision | Generative AI

[LinkedIn](https://www.linkedin.com/in/your-profile) | [GitHub](https://github.com/bhapkarvishal-eic)
