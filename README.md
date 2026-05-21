# retail-ai-agent
Real-time Edge AI retail intelligence on NVIDIA Jetson AGX Xavier

Converts a live camera feed into actionable store metrics — fully local, no cloud required, no face recognition.


What This Is
A production-style Edge AI pipeline for smart retail analytics. The system runs entirely on a Jetson AGX Xavier at the edge. It detects customers, tracks them anonymously, counts entries and exits, measures dwell time per zone, monitors queue formation, logs every event, and streams a live dashboard to any browser on the local network.
This is not a notebook demo. It runs on real hardware with a live camera.

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
