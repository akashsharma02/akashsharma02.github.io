---
layout: distill
title: Depth Fusion for Large Scale Environments
description: Implementation of Spatial Hashing in OpenCV RGBD
github: https://github.com/opencv/opencv_contrib/pull/2566
img: assets/img/opencv_kinfu.gif
date: 2020-07-25
authors:
  - name: Akash Sharma
    url: "/"
    affiliations:
      name: The Robotics Insitute, CMU
importance: 2
category: work
---

This project aimed to implement spatial hashing for the TSDF volume data structure in the OpenCV RGBD module, and leverage the same to build a scalable submap based online 3D scene reconstruction system with little to no drift.

TSDF volumes are widely agreed upon in the research community to be locally consistent with minimal drift, therefore a natural mapping model is a PoseGraph[2] of overlapping submaps, each representing a local section of the entire scene. This mapping model allows for periodic global optimization, which corrects accumulating drift retrospectively in the model, as new sensor measurements are incorporated.

The following delineates the pipeline:

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.html path="assets/img/opencv-large-kinfu-pipeline.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Module Pipeline
</div>


This implementation uses the existing Kinect Fusion[1] pipeline in OpenCV.
