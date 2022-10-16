---
layout: post
title:  "Automated Camera Stabilization and Calibration for Intelligent Transportation Systems"
date:   2021-02-12 13:52:00 +0100
categories: Studies
tags: coding guided-research computer-vision autonomous-driving
pin: false
mermaid: true
---

Within the [PROVIDENTIA++](https://www.bmvi.de/SharedDocs/DE/Artikel/DG/AVF-projekte/providentia-plusplus.html) project, a section of the highway A9 between Munich and Nuremberg was converted to a testing site for autonomous driving. As part of this, a large sensor network system has been set up along the highway to allow monitoring and steering of traffic as well as to improve the coordination between autonomous and traditional cars. The primary task of the intelligent system is to create a digital traffic twin that accurately represents the physical road situation in real-time. Based on this digital twin, the smart infrastructure can provide a far-reaching and comprehensive view to the drivers and autonomous cars in order to improve their situational awareness within the current traffic environment. A video about the PROVIDENTIA project is available on [YouTube](https://youtu.be/4oCnQlGFuc4).

# Description
A key challenge lies in the reliable and accurate calibration of the different
sensors. The calibration is especially challenging when the sensor is subject to
real-life disturbances like vibration of its mounting pole caused by wind or
displacements due to temperature expansion. The aim of this [Guided Research](https://www.in.tum.de/en/current-students/masters-programs/informatics/guided-research/) is to
investigate the feasibility to automatically stabilize and calibrate a shaking
camera using an additional IMU sensor that delivers measurements on the
disturbances.

# Tasks
- Familiarization with stabilization and calibration methods via literature research
- Development of an experimental setup consisting of an oscillating platform equipped with a camera and an IMU
- Development of an approach to automatically stabilize and calibrate the camera
- Evaluation of the concept using real-life data

## [Implementation and Resources](https://github.com/Brucknem/GuidedResearch)

***

# Timeline

```mermaid
gantt
Familiarisation                 : done,     des1,   2020-11-01, 2020-11-30
Dynamic Stabilization           : done,     des2,   after des1, 2021-01-15
Static Calibration              : done,     des3,   after des2, 2021-02-28
Evaluation                      : done,          des4,   after des3, 2021-03-31
Report                          : active,          des5,   after des4, 2021-04-18
```

***

# Examples

Will follow soon.