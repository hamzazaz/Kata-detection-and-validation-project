# Kata Pose Detection – Python Environment Setup

This guide provides full instructions for setting up the Python environment needed to run the **Kata Pose Detection System**, including dependency installation, folder structure, and troubleshooting.

---

## Required Python Version

- **Python 3.8 - 3.11** (Recommended: **Python 3.10**)
- **Note**: Avoid Python 3.12+ as some dependencies may not yet be compatible.

---

## Required Libraries & Installation

### Core Dependencies

```bash
# Computer Vision and Image Processing
pip install opencv-python==4.8.1.78
pip install opencv-contrib-python==4.8.1.78

# MediaPipe for Pose Detection
pip install mediapipe==0.10.8

# Scientific Computing
pip install numpy==1.24.3
pip install scipy==1.11.4

# Data Handling and Plotting
pip install pandas==2.1.4
pip install matplotlib==3.8.2
pip install seaborn==0.13.0
```

### Install All at Once (Recommended)

```bash
pip install opencv-python==4.8.1.78 mediapipe==0.10.8 numpy==1.24.3 scipy==1.11.4 pandas==2.1.4 matplotlib==3.8.2 seaborn==0.13.0
```



---

## Environment Setup Options

### Option 1: Virtual Environment (Recommended)

```bash
# Create virtual environment
python -m venv kata_pose_env

# Activate on Windows
kata_pose_env\Scripts\activate

# Activate on macOS/Linux
source kata_pose_env/bin/activate

# Install dependencies
pip install -r requirements.txt
```

### Option 2: Conda Environment

```bash
# Create environment
conda create -n kata_pose python=3.10

# Activate environment
conda activate kata_pose

# Install dependencies
pip install -r requirements.txt
```

---
