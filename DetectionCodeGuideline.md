# Heian Shodan - Enhanced Video Kata Detection Documentation

## Overview

This system performs real-time kata pose detection using MediaPipe and OpenCV, comparing user poses against reference poses with hybrid similarity matching and fixed time perception.

## Quick Start

### Required Files
* **Input video:** `vid.mp4` (your kata performance)
* **Reference poses:** `heian_shodan_reference.json` (pre-recorded reference poses)

### Output Files
* **Annotated video:** `output_detected_simple.mp4`
* **Results CSV:** `match_stats_simple.csv`



## Configuration Parameters

### Core Detection Settings
```python
SIMILARITY_THRESHOLD = 0.55        # Pose match sensitivity (0.0-1.0)
STEP_TIMEOUT_SECONDS = 3           # Max time to wait per step
START_DELAY_SECONDS = 1            # Initial delay before detection starts  
MIN_HOLD_TIME = 0.01              # Minimum pose hold duration
```

### Hybrid Similarity Parameters
```python
DISTANCE_THRESHOLD = 3.0           # Max allowed distance between joints
COSINE_WEIGHT = 0.6               # Weight for angle-based similarity
DISTANCE_WEIGHT = 0.4             # Weight for position-based similarity
```

## Parameter Adjustment Guide

### SIMILARITY_THRESHOLD (0.0 - 1.0)
**What it does:** Controls how strict pose matching is  
* **Higher values (0.7-0.9):** Very strict matching, fewer false positives  
* **Lower values (0.3-0.5):** More lenient matching, more false positives  
* **Recommended range:** 0.5-0.6

**When to adjust:**  
* Steps completing too easily → Increase threshold  
* Steps timing out frequently → Decrease threshold

### STEP_TIMEOUT_SECONDS  
**What it does:** Maximum time allowed per kata step before moving to next  
* **Higher values (5-10s):** More time to achieve poses  
* **Lower values (1-2s):** Faster progression, may miss poses if previous pose detected early  
* **Recommended range:** 3-5 seconds

### MIN_HOLD_TIME  
**What it does:** How long user must hold correct pose  
* **Higher values (0.5-1.0s):** Ensures stable poses, prevents quick transitions. However may harm very quick poses  
* **Lower values (0.01-0.1s):** Faster detection, may catch transitional poses  
* **Recommended range:** 0.1-0.3 seconds

### DISTANCE_THRESHOLD (1.0 - 10.0)  
**What it does:** Maximum allowed distance between corresponding joints  
* **Lower values (1.0-2.0):** Very precise positioning required  
* **Higher values (5.0-8.0):** More forgiving positioning  
* **Recommended range:** 2.5-4.0

### Weight Parameters (COSINE_WEIGHT + DISTANCE_WEIGHT = 1.0)  
* **COSINE_WEIGHT:** Emphasizes relative joint angles and pose shape  
* **DISTANCE_WEIGHT:** Emphasizes absolute joint positions

**Presets:**  
* **Angle-focused:** COSINE_WEIGHT=0.8, DISTANCE_WEIGHT=0.2 (More forgiving)  
* **Position-focused:** COSINE_WEIGHT=0.3, DISTANCE_WEIGHT=0.7 (not recommended would be very sensitive to dimension differences)  
* **Balanced:** COSINE_WEIGHT=0.6, DISTANCE_WEIGHT=0.4

## Core Methods Reference

### `normalize_landmarks(landmarks)`
**Purpose:** Normalizes pose landmarks relative to hip center and torso height  
**Input:** MediaPipe landmarks list  
**Output:** Normalized pose array (99 elements: 33 joints × 3 coordinates)  

**Key Features:**  
* Centers pose on hip midpoint  
* Scales by torso height for size invariance  
* Returns flattened array for processing

### `get_key_points_preview(normalized_pose)`
**Purpose:** Extracts and formats key joint positions for debugging  
**Input:** Normalized pose array  
**Output:** Formatted string showing key joint coordinates  

**Displays:**  
* Left/Right wrist positions  
* Left/Right elbow positions  
* Left/Right knee positions

### `compare_poses_detailed(user_pose, reference_pose, step_name)`
**Purpose:** Provides detailed side-by-side pose comparison analysis  
**Input:** Two normalized pose arrays and step name  
**Output:** Formatted comparison string with differences  

**Features:**  
* Joint-by-joint coordinate comparison  
* Euclidean distance calculations  
* Color-coded match quality indicators (✅⚠️❌)  
* Average difference scoring  
* Match quality assessment

### `extract_reference_pose(step_data)`
**Purpose:** Extracts reference pose from JSON data structure  
**Input:** JSON step data object  
**Output:** Normalized pose array  

**Handles:**  
* "normalized_landmarks" format  
* "landmarks_2d" format  
* Dict and array data structures

### `pose_similarity(pose1, pose2, distance_threshold, cos_weight, dist_weight)`
**Purpose:** Hybrid pose similarity calculation combining angles and distances  
**Input:** Two poses + similarity parameters  
**Output:** Similarity score (0.0-1.0)

#### Algorithm Components:

##### Part 1: Cosine Similarity (Angle-based)  
* Calculates joint angle similarities  
* Applies weighted importance to arm/wrist joints  
* ARM_WEIGHT_MULTIPLIER: 3.0 (arm joint importance)  
* WRIST_WEIGHT_MULTIPLIER: 4.0 (wrist joint importance)

##### Part 2: Distance Constraint (Position-based)  
* Measures euclidean distances between key joints  
* Key joints monitored: Wrists, elbows, knees  
* Converts distances to similarity scores  
* Applies distance penalty for poses too far apart

##### Part 3: Hybrid Combination  
* Combines weighted cosine + distance similarities  
* Applies distance penalty multiplier  
* Returns final hybrid score

## Detection Loop Flow

### Frame Processing Sequence  
1. **Start Delay:** Skip initial frames (START_DELAY_SECONDS)  
2. **Pose Detection:** Extract MediaPipe pose landmarks  
3. **Pose Normalization:** Apply normalize_landmarks()  
4. **Similarity Calculation:** Use pose_similarity() with reference  
5. **Hold Timer Logic:** Track consecutive good frames  
6. **Step Completion Check:** Verify hold time + frame requirements  
7. **Visual Feedback:** Draw annotations on frame  
8. **State Management:** Reset for next step or handle timeout

### Timing System (Fixed Time Perception)  
* **Frame-based timing:** Uses video FPS for consistent timing  
* **Video timestamp:** frame_idx / fps  
* **Elapsed time:** (current_frame - step_start_frame) / fps  
* **Hold time:** (current_frame - pose_match_start_frame) / fps

### Step Completion Conditions  
Both conditions required:  
1. `hold_time >= MIN_HOLD_TIME`  
2. `consecutive_good_frames >= required_good_frames`  

**Required good frames:** `int(fps * MIN_HOLD_TIME)`

### Visual Feedback Elements  
* **Step name and number:** Current kata step being detected  
* **Similarity scores:** Current and best similarity for step  
* **Hold timer:** Progress toward minimum hold time  
* **Frame counter:** Consecutive good frames vs required  
* **Timeout timer:** Elapsed time vs step timeout  
* **Status indicators:** Match quality and completion status

## Troubleshooting Common Issues

### Steps Completing Too Quickly  
* Increase SIMILARITY_THRESHOLD (try 0.65-0.75)  
* Increase MIN_HOLD_TIME (try 0.2-0.5s)  
* Decrease DISTANCE_THRESHOLD (try 2.0-2.5)

### Steps Timing Out Frequently  
* Decrease SIMILARITY_THRESHOLD (try 0.4-0.5)  
* Increase STEP_TIMEOUT_SECONDS (try 5-8s)  
* Increase DISTANCE_THRESHOLD (try 4.0-6.0)  
* Adjust weight toward cosine: COSINE_WEIGHT=0.8, DISTANCE_WEIGHT=0.2

### False Positive Matches  
* Increase DISTANCE_THRESHOLD precision (try 2.0-2.5)  
* Adjust weight toward distance: COSINE_WEIGHT=0.4, DISTANCE_WEIGHT=0.6  
* Increase MIN_HOLD_TIME (try 0.3-0.5s)

### Pose Not Detected  
* Check MediaPipe detection confidence in pose initialization  
* Ensure good lighting and full body visibility  
* Verify reference JSON file format and data integrity

## File Format Requirements

### Reference JSON Structure
```json
{
  "steps": [
    {
      "step_name": "Ichi: Left Lower Block",
      "normalized_landmarks": [
        {"x": 0.123, "y": -0.456, "z": 0.789}
        // ... 33 landmarks total
      ]
    }
  ]
}
```

**Alternative formats supported:**  
* Direct array format (no "steps" wrapper)  
* "landmarks_2d" field name  
* Array format: `[[x, y, z], [x, y, z], ...]`

### Output CSV Format
```csv
Step Index,Step Name,Timestamp (s),Best Similarity,Status
0,Ichi: Left Lower Block,5.23,0.672,SUCCESS
1,Ni: Right Lunge Punch,8.45,0.456,TIMEOUT
```

## Performance Optimization

### Joint Weighting Strategy  
The system applies higher weights to critical joints:  
* **Wrists (4x weight):** Most important for technique accuracy  
* **Arms (3x weight):** Secondary importance for pose structure  
* **Other joints (1x weight):** Standard importance

### Distance Calculation Optimization  
Only key joints used for distance calculations:  
* Left/Right wrists  
* Left/Right elbows  
* Left/Right knees  

This reduces computational overhead while maintaining accuracy.

### Memory Management  
* Pose arrays are flattened for efficient processing  
* Only essential joint data stored in memory  
* Frame-by-frame processing prevents memory buildup

## Advanced Configuration

### Custom Joint Weights  
Modify joint_weights array in pose_similarity() method:
```python
joint_weights = np.ones(33)  # Default weights
joint_weights[15] = 5.0      # LEFT_WRIST extra importance
joint_weights[16] = 5.0      # RIGHT_WRIST extra importance
```

### Custom Distance Joints  
Modify key_joints_for_distance list:
```python
key_joints_for_distance = [
    mp_pose.PoseLandmark.LEFT_WRIST.value,   # 15
    mp_pose.PoseLandmark.RIGHT_WRIST.value,  # 16
    # Add other joints as needed
]
```

### Debug Output Control  
Control debug information by modifying the distance threshold check in pose_similarity():
```python
if avg_distance > distance_threshold * 1.5:  # Only log problematic cases
    print(f" HYBRID DEBUG: ...")  # Debug output
```

## Summary Statistics

The system provides comprehensive analysis:  
* **Success rate:** Percentage of steps completed successfully  
* **Average similarity:** Mean similarity scores for successful steps  
* **Similarity range:** Min/max similarity values achieved  
* **Timeout analysis:** Performance of steps that timed out  

All statistics are automatically calculated and displayed after processing completion.
