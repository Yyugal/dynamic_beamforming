# Dynamic Beamforming
Vision Aided Beamforming for Multi-Phone Detection & Distance-Based Beam Steering


# Short Description
This project uses a predefined 8×8 beam codebook that spans a full range of azimuth and elevation angles, enabling smooth direction mapping for multi-user beamforming. Distance estimation is performed using the inverse of the detected phone’s bounding-box height, combined with a simple calibration constant. Camera intrinsics are approximated using standard values for a laptop webcam, which provides sufficiently accurate azimuth and elevation estimation for real-time demonstrations.


# Features
Real-Time Multi-Phone Detection: Uses YOLOv8 to accurately detect multiple mobile phones simultaneously in a live webcam feed.

Azimuth & Elevation Estimation: Computes 3D direction of each phone using camera geometry and bounding-box centers.

Monocular Distance Estimation: Approximates user distance based on bounding-box height, enabling distance-aware beam steering.

Multi-User Beamforming: Uses a 64-beam (8×8) codebook with Gaussian weighting to select the dominant beam for one or more users.

Stable Tracking System: IoU-based multi-object tracking with smoothing filters ensures consistent IDs and reduced jitter.

Dynamic Visualization: Live display of detection boxes, beam arrows, polar user map, and time-series graph of beam power.

Range Filtering: Automatically ignores users beyond 5 meters to maintain stable and realistic beam alignment.


# System Architecture / Project Overview
<img width="1168" height="427" alt="image" src="https://github.com/user-attachments/assets/9c105cdd-62d5-481c-a240-0df8b65816de" />

The system follows a streamlined computer-vision pipeline built using Python, OpenCV, YOLOv8, and Matplotlib. The process begins with continuous frame capture from a standard laptop webcam, which supplies real-time visual data to the system. Each frame is passed to the YOLOv8 phone detector, where all visible mobile phones are identified with high confidence. The detected bounding boxes are then forwarded to the tracking, angle estimation, and distance estimation module, where IoU-based multi-object tracking assigns stable IDs and camera geometry is used to compute azimuth, elevation, and approximate user distance.


# Installation & Running the Project Locally
1. Install Dependencies

Make sure you have Python 3.8+ installed, then install the required libraries:

pip install ultralytics
pip install opencv-python
pip install numpy
pip install matplotlib


If you want GPU acceleration, install the CUDA-compatible PyTorch version first:

pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118

2. Download or Clone the Project
git clone https://github.com/your-repo/vision-aided-beamforming.git
cd vision-aided-beamforming


Or simply place the provided Python script in a folder.

3. Run the Script

Run the main Python file:

python beamforming_demo.py


The system will automatically:

Open your webcam

Detect mobile phones

Estimate angles and distance

Select dominant beams

Display the camera feed, polar plot, and beam timeline

Press ‘q’ anytime to exit.


# System Architecture / Project Overview
The system follows a streamlined computer-vision pipeline built using Python, OpenCV, YOLOv8, and Matplotlib. The process begins with continuous frame capture from a standard laptop webcam, which supplies real-time visual data to the system. Each frame is passed to the YOLOv8 phone detector, where all visible mobile phones are identified with high confidence. The detected bounding boxes are then forwarded to the tracking, angle estimation, and distance estimation module, where IoU-based multi-object tracking assigns stable IDs and camera geometry is used to compute azimuth, elevation, and approximate user distance.

<img width="570" height="296" alt="image" src="https://github.com/user-attachments/assets/7b4c5179-62c9-417e-9960-04d2fcb2356d" />

These spatial estimates are fed into the beamforming codebook and power allocation module, which compares the detected phone directions against a predefined 8×8 (64-beam) codebook. Using Gaussian weighting, the system computes per-beam power contributions for one or multiple users and selects the dominant beam dynamically. Finally, all outputs are rendered through the visualization module, which overlays bounding boxes 
and a beam arrow on the camera feed while simultaneously updating the polar plot that shows the user direction and beam distribution.

1. Phone Detection (YOLOv8)

The webcam frame is passed to YOLOv8, which identifies mobile phones using its pretrained COCO model. Only detections labeled as "cell phone" and having a confidence score above a threshold are kept. In the code, this is handled by:

results = model(frame)
for box in r.boxes:
    if cls_name in PHONE_LABELS and score >= SCORE_THRESH:
        detections.append(...)


Detection runs every N frames (controlled by DETECT_EVERY_N) to improve FPS while still maintaining accuracy.

2. Tracking Using IoU Matching

YOLO detects objects per frame but does not know which detection belongs to which user. To solve this, the system uses IoU-based greedy matching to associate new detections with existing tracks. Each track has an ID, color, and counters for visible/invisible frames. If a track is not detected for several frames, it is removed.

The key logic appears in functions:

matches = greedy_match(tracks, detections, IOU_THRESH)


and track updates such as:

tr["invisible"] += 1
if tr["invisible"] > MAX_INVISIBLE:
    delete track


This ensures consistent tracking even when phones move or briefly disappear.

3. Angle Estimation (Azimuth & Elevation)

For each tracked phone, the center of its bounding box is converted to a 3D ray using the pinhole camera model. The system approximates camera intrinsics (fx, fy, cx, cy) and computes:

Azimuth (horizontal angle)

Elevation (vertical angle)

Using:

az = arctan2(x, z)
el = arctan2(-y, sqrt(x*x + z*z))


To reduce jitter, an Exponential Moving Average (EMA) is applied:

tr["az_s"] = EMA_ALPHA * old + (1 - EMA_ALPHA) * new


This stabilizes the beam direction even when detections fluctuate slightly.

4. Distance Estimation (Monocular Depth)

Since there is no depth sensor, distance is estimated from the height of the phone’s bounding box. A larger bounding box means the phone is closer to the camera. The code uses:

h_norm = (y2 - y1) / H
dist_m = DIST_CALIB / h_norm


The value is clamped between 0.5 m and 10 m to avoid unstable calculations.
Phones beyond 5 meters are excluded from beamforming to maintain reliability.

5. Beamforming Logic (64-Beam Codebook)

The system uses an 8×8 beam codebook representing 64 possible azimuth–elevation beam directions. For each track, the angular difference between the phone’s azimuth/elevation and every beam in the codebook is computed. A Gaussian weighting is applied to calculate how strongly each beam aligns with each user:

w = exp(-0.5 * ((d_az/σ_az)^2 + (d_el/σ_el)^2))


When multiple phones are active, their weights are combined using their detection confidence scores. The beam with the highest total power becomes the dominant beam:

dom_idx = argmax(beam_power)


This index corresponds to the beam direction used for visualization.

# Results
The system successfully demonstrates real-time multi-user beamforming using only a laptop webcam and Python-based computer vision. YOLOv8 consistently detects mobile phones with stable bounding boxes, while the IoU-based tracking module maintains reliable IDs even when users move within the frame. Azimuth and elevation estimation remains smooth due to exponential filtering, and the monocular distance approximation performs effectively within the intended 5-meter range.

