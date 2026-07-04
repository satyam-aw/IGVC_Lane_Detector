# IGVC Lane Detection and Autonomous Navigation

This repository contains the lane detection and autonomous navigation pipeline developed for the Intelligent Ground Vehicle Competition (IGVC) 2018 Auto-Nav Challenge. The system was designed for real-time outdoor navigation on an autonomous ground vehicle using camera-based perception, CUDA/GPU-accelerated SLIC-style superpixel segmentation, HOG + SVM-based obstacle/non-lane region rejection, and ROS-based waypoint publishing.

Our team competed in the 26th edition of IGVC, an international collegiate robotics competition held annually at Oakland University, and achieved Rank 2 in the Auto-Nav Challenge.

## Overview

The pipeline processes live camera frames from the robot, extracts lane-like regions, rejects obstacle or non-lane regions, estimates lane boundaries, and publishes local navigation goals to the ROS stack.

The lane detector transforms the camera view into a bird's-eye/top-view representation, applies CUDA/GPU-accelerated SLIC-style superpixel segmentation to group candidate lane pixels, filters candidate regions, fits lane boundaries using a parabolic lane model, and generates waypoints for downstream navigation.

## Pipeline

```text
Camera / Video Feed
        ↓
ROS Image Subscription + cv_bridge
        ↓
OpenCV Color-Channel Filtering
        ↓
Perspective Transform to Bird's-Eye / Top View
        ↓
CUDA/GPU-Accelerated SLIC-Style Superpixel Segmentation
        ↓
Candidate Lane Region Filtering
        ↓
Contour-Based ROI Extraction
        ↓
HOG Feature Extraction + SVM-Based Obstacle / Non-Lane Region Rejection
        ↓
Parabolic Lane Boundary Fitting
        ↓
Waypoint Generation
        ↓
ROS Navigation Goal + Lane LaserScan Publishing
```

## Key Features

- ROS-integrated lane detection node for autonomous ground vehicle navigation
- Live image subscription from `/camera/image_color`
- OpenCV-based preprocessing and color-channel filtering for lane extraction
- Perspective transformation to generate a bird's-eye/top-view representation
- CUDA/GPU-accelerated SLIC-style superpixel segmentation using `FastImgSeg`
- Contour-based ROI extraction from candidate lane/path regions
- HOG descriptor extraction and SVM classification to reject obstacle or non-lane regions
- Parabolic lane-boundary fitting for estimating left and right lane curves
- Local waypoint generation from detected lane geometry
- ROS publishing of waypoint goals to `/move_base_simple/goal`
- Conversion of detected lane regions into a `LaserScan`-style representation published on `/lanes`

## Technical Details

The lane detection node subscribes to the camera stream and converts incoming ROS image messages into OpenCV images using `cv_bridge`. Each frame is resized and passed through a custom color-channel mixing step that emphasizes lane markings while suppressing grass/background regions.

A calibrated homography is then applied to transform the image into a bird's-eye/top-view perspective. This makes lane geometry easier to estimate because the ground plane is represented more like a local 2D map.

After thresholding lane-like pixels, the pipeline applies CUDA/GPU-accelerated SLIC-style superpixel segmentation through `FastImgSeg`. The resulting segmentation mask groups pixels into superpixel regions. Each superpixel is scored based on the density of high-intensity lane-like pixels, allowing the detector to retain candidate lane/path regions while reducing pixel-level noise.

Candidate regions can then be further filtered using contour-based ROI extraction. Each ROI is resized and represented using HOG descriptors, and an SVM classifier predicts whether the region corresponds to an obstacle or non-lane object. Regions classified as obstacles/non-lane objects are masked out before lane fitting.

The detector samples points from the remaining lane candidates and fits a parabolic lane model to estimate left and right lane boundaries. Depending on whether one or both lanes are visible, the system computes a local waypoint and heading. This waypoint is published as a `geometry_msgs::PoseStamped` goal for the ROS navigation stack, while the lane mask is also converted into a `sensor_msgs::LaserScan` representation for integration with other navigation modules.

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

Generated local waypoint goal for the ROS navigation stack.

```text
/lanes
```

Lane-region representation converted into a `LaserScan` message.

## Notes

- The waypoint output is a local target pose for the ROS navigation stack, not a direct low-level motor command.
- The HOG + SVM stage uses HOG as a feature representation and SVM for classification/rejection of obstacle or non-lane ROIs.
- The CUDA/GPU-based SLIC-style segmentation is invoked through `FastImgSeg`; this repository code calls the segmentation interface rather than showing CUDA kernels directly in the main lane detector file.

## Competition Context

The Intelligent Ground Vehicle Competition evaluates autonomous ground vehicles on perception, navigation, obstacle avoidance, and robustness in outdoor environments. This repository contains the lane perception and waypoint-generation components developed for the IGVC 2018 Auto-Nav problem statement.
