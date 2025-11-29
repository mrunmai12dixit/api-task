# Sports Pose Estimation — Anomaly Detection API

FastAPI-based API for analyzing time-series pose coordinates, detecting anomalies, and computing a session smoothness score. Includes a generated combined dataset, original detection logic, and tests.

## Features
- Upload pose time-series via CSV or JSON
- Analyze anomalies: angle deviation (z-score), pauses, velocity spikes
- Custom smoothness score: `100 - (std_dev_of_joint_angles + anomalies*2)`
- Synthetic combined dataset mixing MediaPipe-like and UP Fall Detection–like sequences

## Dataset
- Location: `dataset/combined_pose_data.csv`
- Format: columns include `timestamp, nose_x, nose_y, l_shoulder_x, l_shoulder_y, r_shoulder_x, r_shoulder_y, l_elbow_x, l_elbow_y, r_elbow_x, r_elbow_y, l_wrist_x, l_wrist_y, r_wrist_x, r_wrist_y, l_hip_x, l_hip_y, r_hip_x, r_hip_y, l_knee_x, l_knee_y, r_knee_x, r_knee_y, l_ankle_x, l_ankle_y, r_ankle_x, r_ankle_y`
- Generation: on first run, synthetic segments are generated for:
  - Normal motion (MediaPipe-like)
  - Abnormal movement (UP Fall–like falls)
  - Pauses
  - Jerks
  - Wrong form angles

## API

### POST `/upload-pose-data`
Accepts CSV file upload or JSON payload of frames.

JSON body example:
```json
{
  "frames": [
    {
      "timestamp": 0,
      "nose_x": 100, "nose_y": 200,
      "l_shoulder_x": 95, "l_shoulder_y": 210,
      "r_shoulder_x": 105, "r_shoulder_y": 210,
      "l_elbow_x": 90, "l_elbow_y": 230,
      "r_elbow_x": 110, "r_elbow_y": 230,
      "l_wrist_x": 85, "l_wrist_y": 250,
      "r_wrist_x": 115, "r_wrist_y": 250,
      "l_hip_x": 95, "l_hip_y": 280,
      "r_hip_x": 105, "r_hip_y": 280,
      "l_knee_x": 95, "l_knee_y": 320,
      "r_knee_x": 105, "r_knee_y": 320,
      "l_ankle_x": 95, "l_ankle_y": 360,
      "r_ankle_x": 105, "r_ankle_y": 360
    }
  ]
}
```

Response:
```json
{"status": "saved", "path": "uploads/1730000000_pose.json"}
```

### GET `/analyze`
Analyze a dataset. If `source` is omitted, uses `dataset/combined_pose_data.csv`.

Query params:
- `source`: optional path to `.csv` or `.json` saved via upload

Response example:
```json
{
  "anomalies": [
    {
      "timestamp": 150.0,
      "type": "z_score_angle_deviation",
      "joint": "l_elbow",
      "value": 12.3,
      "explanation": "Angle change at l_elbow deviates with z>3.0"
    },
    {
      "timestamp": 200.0,
      "type": "pause",
      "joint": null,
      "value": 0.1,
      "explanation": "Near-zero frame velocity indicates a sudden pause"
    }
  ],
  "smoothness_score": 82.4,
  "summary": {
    "angle_std_dev": 4.7,
    "num_anomalies": 12,
    "z_thresh": 3.0,
    "pause_velocity_threshold": 0.5,
    "velocity_spike_factor": 3.0
  }
}
```

## Detection Logic
- Joint angles computed per frame for shoulders, elbows, knees.
- Angle change z-score: flags outliers above threshold.
- Pause detection: mean per-frame joint velocity below threshold.
- Velocity spikes: median absolute deviation–based threshold.

## Smoothness Score
`score = 100 - (std_dev_of_joint_angles + number_of_anomalies*2)` clamped to `[0, 100]`.

## Running

### Using shell script
```bash
sh run.sh
```

### Manual commands
```bash
pip install -r requirements.txt
uvicorn app.main:app --reload
```

Open: `http://localhost:8000/docs`

## Testing
```bash
pytest -q
```

## GitHub
Repository: `sports-pose-anomaly-detection`
Commits include initialization, dataset, algorithm, routes, README, tests.

