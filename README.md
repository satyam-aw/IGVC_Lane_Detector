# IGVC Lane Detection and Autonomous Navigation

This repository contains the lane detection and autonomous navigation pipeline developed for the Intelligent Ground Vehicle Competition (IGVC) 2018 Auto-Nav Challenge. The system was designed for real-time outdoor navigation on an autonomous ground vehicle using camera-based lane perception, CUDA-accelerated segmentation, and ROS-based actuation.

Our team competed in the 26th edition of IGVC, an international collegiate robotics competition held annually at Oakland University, and achieved Rank 2 in the Auto-Nav Challenge.

## Overview

The pipeline processes live video from the robot’s camera, extracts lane-like regions, estimates the navigable path, and publishes navigation goals to the ROS stack. The lane detector transforms the camera view into a bird’s-eye/top-view representation, performs superpixel-based region grouping, fits lane boundaries using a parabolic lane model, and generates waypoints for downstream navigation.

## Pipeline

```text
Camera / Video Feed
        ↓
ROS Image Subscription
        ↓
OpenCV + cv_bridge Frame Conversion
        ↓
Color-Channel Filtering for Lane/Grass Separation
        ↓
Perspective Transform to Top View
        ↓
CUDA-Accelerated SLIC Superpixel Segmentation
        ↓
Lane Boundary Estimation and Parabolic Curve Fitting
        ↓
Waypoint Generation
        ↓
ROS Publishing for Navigation and Actuation
```

## Key Features

- ROS-integrated lane detection node for autonomous ground vehicle navigation
- Live image subscription from `/camera/image_color`
- OpenCV-based preprocessing and color-channel filtering for lane extraction
- Perspective transformation to generate a top-view representation of the scene
- CUDA-accelerated superpixel segmentation for grouping lane-region candidates
- Parabolic lane-boundary fitting for estimating left and right lane curves
- Waypoint generation from detected lane geometry
- ROS publishing of navigation goals to `/move_base_simple/goal`
- Conversion of detected lane regions into a `LaserScan`-style representation published on `/lanes`

## Technical Details

The lane detection node subscribes to the camera stream, converts incoming ROS image messages into OpenCV images, and resizes each frame for processing. A custom channel-mixing step emphasizes lane markings while suppressing grass/background regions. The image is then warped into a top-view perspective using a calibrated homography matrix.

After thresholding lane-like regions, the pipeline applies CUDA-accelerated SLIC-style superpixel segmentation to group candidate lane pixels. These superpixels are filtered based on the density of high-intensity lane pixels, producing a sparse set of candidate points for lane estimation.

The detector samples candidate points and fits a parabolic lane model to estimate lane boundaries. Depending on whether one or both lanes are visible, the system computes a suitable waypoint and heading. The resulting waypoint is published as a `geometry_msgs::PoseStamped` message for ROS navigation, while the lane mask is also converted into a `sensor_msgs::LaserScan` representation for integration with other navigation modules.

## ROS Topics

### Subscribed

```text
/camera/image_color
```

Input camera stream used for lane detection.

### Published

```text
/move_base_simple/goal
```

Generated waypoint for navigation.

```text
/lanes
```

Lane-region representation converted into a `LaserScan` message.

## Competition Context

The Intelligent Ground Vehicle Competition evaluates autonomous ground vehicles on perception, navigation, obstacle avoidance, and robustness in outdoor environments. This repository contains the lane perception and waypoint-generation components developed for the IGVC 2018 Auto-Nav problem statement.
