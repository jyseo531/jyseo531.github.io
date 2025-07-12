---
title: LiDAR SLAM tutorial
author: jiyoung
date: 2025-07-13 07:00:00 +0800
categories: [Study, SLAM]
tags: [SLAM, 3D Perception, Autonomous Driving]
---

<script type="text/javascript">
  MathJax = {
    tex: {
      inlineMath: [['$', '$'], ['\\(', '\\)']]
    }
  };
</script>
<script type="text/javascript" src="https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js"></script>


## Paper List (TODO)
- 87 PAMI Least-squares fitting of two 3D point sets

    *tldr: SVD-based closed form of registration; a basic of basic of the scan matching* 
    
- 92 PAMI A method for registration of 3-D shapes
    
    *tldr: a.k.a ICP; a basic of the scan matching* 
    
- 97 AR Globally Consistent Range Scan Alignment for Environment Mapping
    
    *tldr: mostly called as "Lu and Milios"; considered as the first work of a scan matching and pose graph optimization-based SLAM.*
    
- 03 IROS The Normal Distributions Transform: A New Approach to Laser Scan Matching
    
    *tldr: NDT registration* 
    
- 06 IJRR Square Root SAM: Simultaneous Localization and Mapping via Square Root Information Smoothing
    
    *tldr: from probabilistic nature to least square formulation of SLAM for smoothing (i.e., modification of past poses)* 
    
- 07 JFR Scan registration for autonomous mining vehicles using 3D-NDT
    
    *tldr: 3D version of NDT registration*
    
- 08 TRO iSAM: Incremental Smoothing and Mapping
    
    *tldr: incremental SAM and an open source library* 
    
- 09 ICRA Real-Time Correlative Scan Matching
    
    *tldr: prof. Olson; later affects  to Cartographer, etc.*
    
- 09 ICRA Fast Point Feature Histograms (FPFH) for 3D Registration
    
    *tldr: FPFH (the most famous 3D local descriptor) registration*
    
- 09 RSS Generalized-ICP
    
    *tldr: uncertainty-embedded ICP (probabilistic perspective)* 
    
- 10 ITSM A Tutorial on Graph-Based SLAM
    
    *tldr: Grisetti's must-read tutorial*
    
- 11 IV Velodyne SLAM
    
    *tldr: an early work of modern 3D scanning LiDAR-based motion estimation* 
    
- 12 TRO Zebedee: Design of a Spring-Mounted 3-D Range Sensor with Application to Mobile Mapping
    
    *tldr: mobile mapping system and IMU fusion*
    
- 12 RAM Tutorial: Point Cloud Library: Three-Dimensional Object Recognition and 6 DOF Pose Estimation
    
    *tldr: PCL tutorial, but not much delved into the SLAM perspective.*
    
- 12 IJRR iSAM2: Incremental smoothing and mapping using the Bayes tree
    
    *tldr: in GTSAM 4.0, iSAM2 (not iSAM1) is currently a de-facto default factor graph optimizer.*
    
- 13 ICRA Robust Odometry Estimation for RGB-D Cameras
    
    *tldr: a.k.a DVO; this is not an actually LiDAR thing, but to understand the effectiveness of direct alignment rather ICP* 
    
- 13 IROS Dense Visual SLAM for RGB-D Cameras
    
    *tldr: a SLAM version (i.e., including loop closures) of the DVO; studying RGB-D SLAMs is also worthy for LiDAR guys because they frequently considers the both a photometric error and a geometric error.*   
    
- 13 AR Challenging data sets for point cloud registration algorithms
    
    *tldr: a.k.a the open library: Libpointmatcher*
    
- 14 RSS LOAM: Lidar Odometry and Mapping in Real-time
    
    *tldr: THE LOAM; (surface and corner) feature matching for frame-to-frame registration and frame-to-map refinement*
    
- 15 ICRA Visual-lidar Odometry and Mapping: Low-drift, Robust, and Fast
    
    *tldr: Visual + LOAM*
    
- 15 ICRA Initialization Techniques for 3D SLAM: a Survey on Rotation Estimation and its Use in Pose Graph Optimization
    
    *tldr: LiDAR SLAM usually have more opportunity to think of the better pose graph optimization because it directly measures the depth (rather easier front-end than visual domain).*
    
- 15 RAM Registration with the Point Cloud Library: A Modular Framework for Aligning in 3-D
    
    *tldr: PCL tutorial for registration*
    
- 15 IROS NICP: Dense normal based point cloud registration
    
    *tldr: as already in the title, dense normal*
    
- 16 ICRA Real-time loop closure in 2D LIDAR SLAM
    
    *tldr: the Google Cartographer's paper*
    
- 16 SSRR ICP-based pose-graph SLAM
    
    *tldr: an almost standard framework of scan matching- and pose-graph-based LiDAR SLAM*  
    
- 16 IROS M2DP: A novel 3D point cloud descriptor and its application in loop closure detection
    
    *tldr: place descriptor using a single LiDAR scan*
    
- 16 BookChapter [World modeling](https://link.springer.com/referenceworkentry/10.1007%2F978-3-540-30301-5_37)
    
    *tldr: various representations of a surrounding environment; selecting a proper representation of the environment is important and it determines the state estimation ways.*
    
- 18 RSS Efficient Surfel-Based SLAM using 3D Laser Range Data in Urban Environments
    
    *tldr: a.k.a SuMa; projective view rendering, and* *open source*
    
- 18 ICRA IMLS-SLAM: scan-to-model matching based on 3D data
    
    *tldr: sophisticated feature selections, not real-time but for the accuracy, this is considered as a SOTA*
    
- 18 ICRA Elastic LiDAR Fusion: Dense Map-Centric Continuous-Time SLAM
    
    *tldr: (continuous-time) non-rigid map deformation (see 15 RSS ElasticFusion also)*
    
- 18 ICRA Efficient Continuous-time SLAM for 3D Lidar-based Online Mapping
    
    *tldr: a hierarchical continuous-time LiDAR SLAM*
    
- 18 IROS LeGO-LOAM: Lightweight and Ground-Optimized Lidar Odometry and Mapping on Variable Terrain
    
    *tldr: range image-based fast feature selection for LOAM; and open source*
    
- 18 IROS LIPS: LiDAR-Inertial 3D Plane SLAM
    
    *tldr: leveraging plane for LINS system* 
    
- 18 IROS Scan Context: Egocentric Spatial Descriptor for Place Recognition Within 3D Point Cloud Map
    
    *tldr: A visibility-based place descriptor for fast and robust place recognition, and open source*
    
- 19 A-LOAM (code only)
    
    *tldr: a well-implemented LOAM algorithm (the original LOAM author closed the official code), and open source* 
    
- 19 ICRA Tightly Coupled 3D Lidar Inertial Odometry and Mapping
    
    *tldr: a.k.a lio-mapping; imu tight fusion but practically slow, and* *open source*
    
- 19 IV DeLiO: Decoupled LiDAR Odometry
    
    *tldr: rotation and translation are decoupled*
    
- 19 IJRR SegMap: Segment-based mapping and localization using data-driven descriptors
    
    *tldr: deep segment feature learning for LiDAR place recognition* 
    
- 19 IROS SuMa++: Efficient LiDAR-based Semantic SLAM
    
    *tldr: merging semantic information into SuMa*
    
- 20 AR DVL-SLAM: sparse depth enhanced direct visual-LiDAR SLAM
    
    *tldr: enhanced visual SLAM by LiDAR data* 
    
- 20 RSS OverlapNet: Loop Closing for LiDAR-based SLAM
    
    *tldr: learning two scan's overlap and integrated it into the modern probabilistic SLAM system.*
    
- 20 IROS LIO-SAM: Tightly-coupled Lidar Inertial Odometry via Smoothing and Mapping
    
    *tldr: IMU fusion (tightly) of LOAM, and open source.* 
    
- 20 IROS SpoxelNet: Spherical Voxel-based Deep Place Recognition for 3D Point Clouds of Crowded Indoor Spaces
    
    *tldr: Deep LiDAR feature learning for place recognition and robust to occlusions*
    
- 20 IROS Semantic Graph Based Place Recognition for 3D Point Clouds
    
    *tldr: Summarizing a place with a single semantic graph. The matching part is also deep (SegMap didn't).* 
    
- 20 IROS A Fast and Robust Place Recognition Approach for Stereo Visual Odometry Using LiDAR Descriptors
    
    *tldr: LiDAR descriptors are also good for stereo-camera-based place recognition*
