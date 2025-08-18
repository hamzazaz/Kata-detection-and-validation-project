# Pose Normalization Method - Technical Explanation

## The Core Problem

When comparing poses between different people or videos, raw MediaPipe coordinates are unusable because they're relative to the camera frame, not the person. We needed a way to make poses comparable regardless of:

- **Person position in frame** (left, center, right)
- **Distance from camera** (close vs far)
- **Body size differences** (tall vs short people)

## Our Normalization Solution

```python
def normalize_landmarks(landmarks):
    # Get anatomical reference points
    left_hip = landmarks[mp_pose.PoseLandmark.LEFT_HIP.value]
    right_hip = landmarks[mp_pose.PoseLandmark.RIGHT_HIP.value]
    left_shoulder = landmarks[mp_pose.PoseLandmark.LEFT_SHOULDER.value]
    right_shoulder = landmarks[mp_pose.PoseLandmark.RIGHT_SHOULDER.value]
    
    # Calculate body center
    hip_center_x = (left_hip.x + right_hip.x) / 2
    hip_center_y = (left_hip.y + right_hip.y) / 2
    shoulder_center_y = (left_shoulder.y + right_shoulder.y) / 2
    
    # Calculate scaling factor
    torso_height = abs(shoulder_center_y - hip_center_y)
    if torso_height < 0.01:
        torso_height = 0.1
    
    # Transform all joints
    normalized = []
    for lm in landmarks:
        normalized.append([
            round((lm.x - hip_center_x) / torso_height, 3),
            round((lm.y - hip_center_y) / torso_height, 3),
            round(lm.z / torso_height, 3)
        ])
    
    return normalized
```

## Why This Specific Approach

### Choice 1: Hip Center as Origin (0,0)

**Why hips instead of shoulders or chest?**
- **Anatomical stability:** Hips move less during upper body techniques
- **Pose independence:** Shoulder position changes dramatically with arm movements
- **Center of mass:** Hips represent the body's natural center point
- **Martial arts relevance:** Stance and hip position are fundamental in karate

**The math:**
```
hip_center_x = (left_hip.x + right_hip.x) / 2
hip_center_y = (left_hip.y + right_hip.y) / 2
```

### Choice 2: Torso Height as Scaling Factor

**Why torso height instead of arm span or total height?**
- **Pose consistency:** Torso height stays constant regardless of arm/leg position
- **Measurement reliability:** Easier to measure accurately than full height
- **Proportion preservation:** Natural body scaling that works for all body types
- **Stability:** Less affected by pose variations than other measurements

**The math:**
```
torso_height = abs(shoulder_center_y - hip_center_y)
```

### Choice 3: Division-Based Scaling

**Why divide by torso height?**
- **Size normalization:** Makes tall and short people directly comparable
- **Unit standardization:** Creates "torso heights" as measurement unit
- **Ratio preservation:** Maintains relative body proportions

**The transformation:**
```
normalized_coordinate = (original_coordinate - reference_point) / scaling_factor
```

## What the Transform Achieves

### Before Normalization (Raw MediaPipe):
```
Tall person (close):     Right wrist at (0.75, 0.35)
Short person (far):      Right wrist at (0.65, 0.55)
Same pose, different coordinates ❌
```

### After Normalization (Our Method):
```
Tall person:    Right wrist at (1.2, -0.8) [1.2 torso-heights right, 0.8 below hip]
Short person:   Right wrist at (1.2, -0.8) [Same relative position]
Same pose, same coordinates ✅
```

## Mathematical Properties

### Translation Invariance
Moving the person anywhere in the frame doesn't change normalized coordinates:

```
Original: wrist_x = 0.75, hip_center_x = 0.50
Normalized: (0.75 - 0.50) / torso_height = 0.25 / torso_height

Person moves right by 0.2:
New: wrist_x = 0.95, hip_center_x = 0.70
Normalized: (0.95 - 0.70) / torso_height = 0.25 / torso_height
Same result! ✅
```

### Scale Invariance
Person distance from camera doesn't affect normalized coordinates:

```
Person moves 2x closer (everything doubles):
New torso_height = 2 × old_torso_height
New distances = 2 × old_distances
Normalized: (2 × distance) / (2 × torso_height) = distance / torso_height
Same result! ✅
```

## Implementation Decisions

### Safety Check for Division by Zero
```python
if torso_height < 0.01:
    torso_height = 0.1
```
**Why needed:** Rare cases where shoulders align with hips could cause torso_height ≈ 0
**Why 0.01 threshold:** Allows for minor measurement errors while catching true edge cases
**Why 0.1 fallback:** Reasonable default that won't distort coordinates severely

### Rounding to 3 Decimal Places
```python
round((lm.x - hip_center_x) / torso_height, 3)
```
**Why round:** Eliminates floating-point precision noise
**Why 3 decimals:** Balance between precision and readability
- 2 decimals: Too coarse, loses important details
- 4+ decimals: Unnecessary precision, harder to debug

### Z-Coordinate Handling
```python
round(lm.z / torso_height, 3)
```
**Why include Z:** Provides depth information for 3D pose analysis
**Why same scaling:** Maintains proportional relationships in all dimensions
**Known limitation:** MediaPipe Z coordinates less reliable than X,Y

## Coordinate Interpretation Guide

### Understanding Normalized Values

After normalization, coordinates represent position relative to the person's body:

```python
left_wrist: {"x": 1.2, "y": -0.8, "z": 0.1}
```

**Meaning:**
- **X: 1.2** = 1.2 torso-heights to the right of hip center
- **Y: -0.8** = 0.8 torso-heights above hip level  
- **Z: 0.1** = 0.1 torso-heights forward from body plane

### Typical Coordinate Ranges

**For martial arts poses:**
- **Arms extended wide:** x ≈ ±2.0 to ±3.0
- **Arms at sides:** x ≈ ±0.5 to ±1.0
- **Head level:** y ≈ -1.5 to -2.0
- **Knee level:** y ≈ +1.0 to +1.5
- **Foot level:** y ≈ +2.0 to +2.5

## Validation and Debugging

### Quick Sanity Check
After normalization, verify:
```python
# Hip center should be approximately at origin
hip_avg_x = (normalized[left_hip_idx]['x'] + normalized[right_hip_idx]['x']) / 2
hip_avg_y = (normalized[left_hip_idx]['y'] + normalized[right_hip_idx]['y']) / 2
assert abs(hip_avg_x) < 0.01 and abs(hip_avg_y) < 0.01
```

### Visual Debugging
The key points preview shows the most important joints in human-readable format:
```
Left wrist:  (1.20, -0.85)   # Easy to spot if arms are positioned correctly
Right wrist: (-1.15, -0.82)  # Should be roughly symmetric for centered poses
```
