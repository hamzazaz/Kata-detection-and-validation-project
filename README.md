# Kata Detection System

This kata project is composed of **four interdependent components** working together to enable automated analysis and evaluation of karate kata performance using computer vision and pose detection.

---

## 1. Screenshot Capture Utility

The first component of the project is a **screenshot taker**. It allows for efficient and consistent capturing of kata pose images and saves them directly into a local directory within the project structure.

### Features:
- Captures pose images by pressing the spacebar.
- Ensures **consistent dimension and orientation** across all screenshots.
- Matches the **input video specifications** used later during detection.
- Saves the screenshots into the project folder for further processing.

This tool is not only helpful because of the speed and simplicity of capturing screenshots, but also because it ensures uniformity with the dimensions and perspective used during the detection phase.

---

## 2. Pose Extractor Pipeline

After obtaining screenshots of the kata poses, the **pose extractor** processes each image to detect and normalize human pose landmarks using MediaPipe.

### 2.1 Input: Karate Pose Images
- **Folder**: `im`
- **Image Files**: `screenshot_0.png` to `screenshot_17.png`
- Each image corresponds to a specific kata step (e.g., “Ichi”, “Ni”, etc.).

### 2.2 Pose Detection (MediaPipe)
- Uses **MediaPipe Pose** in **static image mode**.
- Detects **33 body landmarks** for each image.
- Only processes images where a full pose is successfully detected.

### 2.3 Pose Normalization
- Each set of detected landmarks is:
  - **Centered** on the midpoint between the hips (translation-invariant).
  - **Scaled** by the **torso height** (shoulder-to-hip distance, making it scale-invariant).
  - **Rounded** to **three decimal places** for clarity and consistency.
- Normalization ensures that poses are comparable regardless of scale or position.

### 2.4 Key Points Summary
- Extracts key joints including:
  - Wrists
  - Elbows
  - Knees
  - Hips
- These are used for debugging, visual previews, and quick reference.

### 2.5 Reference Step Data Assembly
- For each step, the following data is stored:
  - Step name and index number.
  - Normalized landmark coordinates.
  - Key points summary.
- All step data is appended to a list of reference poses.

### 2.6 Output: JSON Export
- All processed steps are saved to `heian_shodan_reference.json`.
- The JSON file contains:
  - Kata name
  - Total number of steps
  - Normalization method used
  - Detailed landmark data for each step

### 2.7 Terminal Summary Output
- A readable table is printed in the terminal summarizing:
  - Key point positions (e.g., wrists, knees) for each processed step.

---

## 3. Pose Debug Visualizer

The third component is a **debugging helper tool**. It plots the normalized landmarks in the form of a **human skeleton** that visually resembles the original kata poses from the reference video.

### Purpose:
- To provide a quick visual verification of the correctness of pose extraction and normalization.
- To help identify any anomalies in landmark detection or normalization by visual inspection.

---

## 4. Kata Detection Script

The fourth and final component is the **kata detection system**. This script evaluates a karate kata performance from a video using MediaPipe's pose detection combined with the previously generated JSON reference sequence.

### 4.1 Initialization
- Loads the `heian_shodan_reference.json` file which contains:
  - Step-wise normalized 2D landmarks
  - Key joint summaries
- Opens the input video file.
- Begins detection **after a configurable preparation delay** (default: 10 seconds), allowing the performer to get into position.

### 4.2 Pose Detection and Normalization
- For each frame in the video:
  - Extracts 33 landmarks using MediaPipe Pose.
  - Normalizes landmarks:
    - Centered on hip midpoint.
    - Scaled by torso height (hip-to-shoulder distance).
  - This ensures that pose matching is **independent of performer size and camera position**.

### 4.3 Pose Matching Logic
- Each frame’s pose is compared with the current reference pose using a **hybrid similarity metric**:
  
  **Metric 1: Cosine Similarity**
  - Computed over all joint vectors.
  - Heavily weighted on **wrists and elbows** (e.g., wrists get 4× weight).

  **Metric 2: Distance-Based Similarity**
  - Uses Euclidean distance between **key joints** such as wrists, elbows, and knees.

  **Combination and Penalty**
  - Both metrics are combined using configurable weights (e.g., 60% cosine, 40% distance).
  - A **distance penalty** is applied if the performer's position deviates significantly from the reference.

### 4.4 Matching Thresholds
- If the **hybrid similarity score exceeds a threshold** (e.g., 0.55), and the pose is held for a **minimum time duration** (e.g., 0.7 seconds), the step is marked as a **successful match**.
- If the match is not achieved within the **maximum allowed time per step** (default: 3 seconds), the step is marked as a **timeout**, and the system advances to the next reference pose.

---

## 5. Debug Output and Visualization

### 5.1 Console Output
During the detection process, the script provides detailed debug output in the terminal, including:
- Per-step similarity scores
- Differences in key joints
- Match evaluations
- Breakdown of the hybrid metric components

### 5.2 Visual Feedback (Video Overlay)
Each output video frame is annotated with:
- Step name
- Hybrid similarity score
- Hold time
- Match status: match, pending, or failure
- Current progress through the kata sequence

---

## 6. Output Files

At the end of processing, the following output files are generated:

### 6.1 Annotated Video
- A frame-by-frame video displaying all matched steps, pose overlays, and debug annotations.

### 6.2 Match Statistics CSV
- A `.csv` file containing detailed statistics for each step, including:
  - Step index
  - Step name
  - Timestamp
  - Best similarity score
  - Final status (SUCCESS or TIMEOUT)

---

## 7. Summary

This system offers a **robust, interpretable, and configurable pose evaluation pipeline** for karate kata detection. It combines:
- Real-time joint angle analysis
- Spatial body-position validation
- Time-sensitive pose matching
- Extensive visual and statistical output

It is highly suitable for:
- Karate training
- Automated kata scoring
- Motion analysis research and applications

---
## Important Highlights

1. **Reference-Based Testing**  
   The detector has been tested primarily on the original reference video. It still needs to be validated on additional videos with different lighting, performers, and angles.

2. **Pose Visibility Affects Similarity Scores**  
   Certain steps, such as **Step 11** and **Step 13**, may yield lower similarity scores. This is likely due to **limbs being partially or completely out of the camera view**, which negatively impacts landmark detection accuracy.

3. **Input Video Requirements**  
   For the detection to function reliably, the input video **must have the same dimensions and orientation** as the reference video used for pose extraction. Any mismatch in resolution or rotation can significantly degrade accuracy.

4. **Generalizability to Other Katas**  
   This pipeline is designed to be adaptable. It should work for **any kata**, provided that you update the reference step names accordingly and **tune parameters such as timeouts** and similarity thresholds.

5. **Parameter Tuning is Crucial**  
   Adjusting timeouts and thresholds is essential. Proper tuning ensures that high-quality poses are not accidentally skipped, and false matches are minimized. This is especially important when working with real-world videos where timing may vary.

---

## Next Steps

- **Improve Dimension and Rotation Invariance**  
  Currently, any input video with dimensions or rotation differing from the reference video causes a **significant drop in detection accuracy**. Future improvements should focus on making the system **robust to resolution and orientation changes**.

- **Expand to Multiple Katas**  
  Apply the system to detect and analyze **other kata sequences**, creating a diverse and reusable **kata reference library**. This will enable broader training and evaluation capabilities across different styles and routines.

