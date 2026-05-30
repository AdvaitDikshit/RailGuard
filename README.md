#  RailGuard — Railway Track Crack Detection System

AI-powered railway crack detection using **YOLOv8**, with automatic severity classification and advisory report generation.

---

##  Project Structure

```
railway_crack_detection/
├── app.py                          ← Flask web application (main entry point)
├── detector.py                     ← Core detection & advisory engine
├── requirements.txt
├── templates/
│   └── index.html                  ← Web UI
├── static/
│   ├── uploads/                    ← Uploaded images (auto-created)
│   └── results/                    ← Annotated output images (auto-created)
├── scripts/
│   ├── download_dataset.py         ← Download datasets from Roboflow
│   ├── train_model.py              ← Train YOLOv8 model
│   ├── predict.py                  ← CLI inference tool
│   └── evaluate_model.py           ← Model evaluation
├── dataset/
│   └── merged/                     ← Merged training data (auto-created)
├── models/
│   └── best_crack_detector.pt      ← Trained model weights (after training)
└── configs/
```

---

##  Quick Start (Step by Step)

### Step 1 — Install Python & Dependencies

```bash
# Python 3.10+ required
pip install -r requirements.txt
```

> **GPU (recommended):** If you have an NVIDIA GPU, install PyTorch with CUDA:
> ```bash
> pip install torch torchvision --index-url https://download.pytorch.org/whl/cu118
> ```

---

### Step 2 — Get a Roboflow API Key

1. Go to [roboflow.com](https://roboflow.com) and create a free account
2. Click your avatar → **Settings** → **Roboflow API** → copy your key

---

### Step 3 — Configure Datasets

Open `scripts/download_dataset.py` and:

1. Paste your API key: `ROBOFLOW_API_KEY = "your_key_here"`
2. Add/edit the datasets list. To find dataset coordinates:
   - Browse [universe.roboflow.com](https://universe.roboflow.com)
   - Search for "railway crack", "rail defect", "track crack"
   - Click a dataset → **Download** → **YOLOv8** → note the workspace/project/version

**Recommended datasets to search for:**
| Search Term | Notes |
|---|---|
| `railway track defects` | General track defects |
| `rail crack detection` | Surface cracks |
| `concrete crack` | Transferable crack patterns |
| `pavement crack` | Transferable surface cracks |

---

### Step 4 — Download & Merge Datasets

```bash
python scripts/download_dataset.py
```

This downloads all configured datasets and merges them into `dataset/merged/`.

---

### Step 5 — Train the Model

```bash
python scripts/train_model.py
```

Training runs for 100 epochs by default (~1-3 hours on GPU, longer on CPU).
Best model is saved to `models/best_crack_detector.pt`.

**To change training settings**, edit `CONFIG` in `scripts/train_model.py`:
- `"epochs": 150` — more epochs = better accuracy
- `"model": "yolov8m.pt"` — larger model = more accurate but slower
- `"batch": 8` — reduce if GPU runs out of memory

---

### Step 6 — Launch the Web App

```bash
python app.py
```

Open **http://localhost:5000** in your browser.

---

### (Optional) CLI Inference

```bash
# Analyse a single image
python scripts/predict.py --image path/to/track_image.jpg

# With custom model and show result
python scripts/predict.py --image my_image.jpg --show

# Output JSON
python scripts/predict.py --image my_image.jpg --json
```

---

### (Optional) Evaluate Model Performance

```bash
python scripts/evaluate_model.py
```

---

##  Severity Levels & Advisory System

| Severity | Trigger | Action |
|---|---|---|
|  **CRITICAL** | conf ≥ 80%, large area | Immediate line closure, emergency repair |
|  **HIGH** | conf ≥ 65%, moderate area | Speed restriction 10 km/h, repair in 48h |
|  **MODERATE** | conf ≥ 50% | Speed restriction 30 km/h, repair in 7 days |
|  **LOW** | conf < 50% | Routine monitoring |
|  **NO CRACK** | No detections | No action required |

---

##  Configuration Reference

### Severity Thresholds (`detector.py`)
```python
SEVERITY_RULES = [
    # (min_confidence, min_area_fraction, severity, color)
    (0.80, 0.05, "CRITICAL",  ...),
    (0.65, 0.02, "HIGH",      ...),
    (0.50, 0.00, "MODERATE",  ...),
    (0.00, 0.00, "LOW",       ...),
]
```

### Confidence Threshold (`app.py`)
```python
detector = CrackDetector(conf_threshold=0.35)  # lower = more sensitive
```

---

##  Tips for Best Results

1. **Use high-quality images** — overhead/side view of track, good lighting
2. **Multiple datasets** — adding more crack datasets improves accuracy
3. **Fine-tune confidence** — lower threshold for more detections, higher for precision
4. **GPU strongly recommended** — training on CPU takes 10-20x longer
5. **Augmentation** — the training script applies automatic augmentation for small datasets

---

##  Troubleshooting

| Issue | Fix |
|---|---|
| `No trained model found` | Run `train_model.py` first; app works in demo mode until then |
| `CUDA out of memory` | Set `"batch": 8` or `"batch": 4` in training config |
| Roboflow download fails | Check API key and dataset workspace/project names |
| Flask not found | Run `pip install -r requirements.txt` |
| Poor accuracy | Add more datasets, increase epochs, try `yolov8m.pt` |