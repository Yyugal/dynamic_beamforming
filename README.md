# Project Title
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

# Results
The system successfully demonstrates real-time multi-user beamforming using only a laptop webcam and Python-based computer vision. YOLOv8 consistently detects mobile phones with stable bounding boxes, while the IoU-based tracking module maintains reliable IDs even when users move within the frame. Azimuth and elevation estimation remains smooth due to exponential filtering, and the monocular distance approximation performs effectively within the intended 5-meter range.
