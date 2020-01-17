---
layout: post
title: Deep Learning-based Autonomous UAV Landing with Marker Detection
date: 2020-01-15 13:32:20 +0300
description: You’ll find this post in your `_posts` directory. Go ahead and edit it and re-build the site to see your changes. # Add post description (optional)
img: img_drone.jpg # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [Deep learning, Computer vision]
---

As a scientific project leader I was associated with our Innovation Lab in Toulouse to work on this project. I successfully completed this 6 months project and tested the results in an autonmous vehicle simulator. We are currently trying to implement the algorithm I developped in a small drone equipped with a Rasberry pi 3. There are other computational and hardware challenges that I will explain along the way. 

# Objectif of the Project
Autonomous landing of an Unmanned Aerial Vehicle (UAV) is a difficult problem. Although the GPS signal is essential to assist the drone in landing, its error in meters remains significant. In addition, the received signal is often very weak when the drone is in environments with poor GPS connection (urban area, mountains, etc.). We want to remedy this problem by using the standard means available, ie, a monocular vision camera, coupled with the latest advances in Deep Learning for object detection and localization. The object in question in our case is a ground marker. An algorithm based on convolutional neural networks will be proposed in order to be able to help the drone to fly autonomously towards the target and to land there safely and with better precision.


# Context
The autonommous precision landing is part of the AI ​​autopilot project which aims to equip drones with the latest advances in artificial intelligence in order to make the drone autonomous and more stable in the face of environmental variations. The project consists of three main axes: 
* Design of a robust autopilot using reinforcement learning approaches. 
* Develop an AI algorithm to improve the performance of the estimator 
* Develop an algorithm to allow the drone to perform an autonomous landing with a ground marker. 

# Challenges
For the practical implementation development of the precision landing part, we note the following scientific and technical obstacles:
* The reflection of the sun or the shadow on the tag can make detection difficult
* Difficulty detecting the tag if the image from the camera is distorted.
* The lower the resolution of the camera, the more difficult it is to recognize the tag, especially at high heights
* The techniques found in the literature are generally based on the detection of the geometric characteristics of the tag, and this is where the detection problems that have just been mentioned mainly come from.
The proposed solution that relies on the object detection algorithm, named YOLO can overcome most these challenges 

# Introduction
My main interest was in the development of the last, but not least point. In order to carry out the necessary experiments, it was important to choose a drone simulator where the dynamics of the UAV (quadcopter in my case) is well defined. I went through a literature survey to identify numerous autonomous vehicle simulator with the idea to find the one that is most suitable for our type of application. In the survey, it was concluded that AirSim (Aerial Informatics and robotics Simulation). The simulator is developed by Microsoft and 

The drone will thus have the ability to interact with a virtual environment that is closest to reality. This will allow a smoother transition when the time comes to try the algorithms developed on a real drone.

## Main tasks
An algorithm which allows the tracking of a landmark in real time, as well as the calculation of its coordinates using only a single monocular vision camera is proposed. Tiny Yolo has been chosen for the object detection part for its small architecture, which allows it to run faster. Learning the network requires time and significant computing power. It is performed on an Ubuntu 16.04 machine equipped with a GTX 1080 graphics card. The Cuda and cudNN libraries are also installed to allow parallel calculation and thus increase performance and have faster training. AirSim is used for image acquisition (The Figure below shows 4 images taken from a drone's down facing camera from different altitude). ComputerVision mode makes it easier to control the drone, a small Python script is used to acquire the set of images necessary for training the network. To assess the performance of the proposed approach, a quadcopter drone equipped with a camera is simulated with AirSim on the "Landscape Mountain" environment. The results obtained in simulation make it possible to confirm that the drone can detect the marker in real time and, based on the algorithm that will be described later, to calculate the coordinates of the benchmark on the ground and therefore to guide the drone up to a distance of less than a meter in height and with an error of 10 cm from the center of the tag. The drone, when arrived at this position, triggers a secure automatic landing using a feature that is available in the flight controller (autopilot).

![Airsim 01]({{site.baseurl}}/assets/img/airsim_01.jpg){:height="50%" width="50%"}

# Training of Tiny Yolo
Yolo (You Only Look Once) is a fast object detection algorithm developed by Redmon et al. Tiny Yolo is just a compact version of it. Object detection is a task in computer vision which consists of identifying the presence, location and type of one or more objects in a given image. In our case, we are talking about a single object and a single class which represents our ground marker. YOLO makes it possible to predict the coordinates of the detected objects as well as the probability of their associated classes. On a computer equipped with an NVIDIA GTX1080 GPU, it is possible for a YOLOv3, to run at more than 30 fps. YOLO processes the detection of the type of object, and its location inside the image simultaneously. Thanks to its good generalizable representations, it is possible to do transfer learning, that is to say, use weights that are pre-trained and build a new network by changing only the last connected layer.

The network will be trained using the image game from the simulator that we will already have annotated. Annotation consists of associating a text file with each image where the coordinates of the tag in relation to the image will be determined manually using an annotation program easily accessible online. As already mentioned, if you want to embed YOLOv3 on a Rasberry Pi, it will turn very slowly so much that it will be unusable. Tiny YOLO makes it possible to embark YOLOv3, certainly with a small decrease in precision but with a considerable gain in computing time. The structure of the Tiny Yolo network in version 3 is presented in the Table just below

![Yolo Architecture]({{site.baseurl}}/assets/img/yolo_architecture.png){:height="60%" width="60%"}

# Landing System 
The diagram below illustrates the operating procedure for the autonomous landing system. The Tiny YOLO localisation algorithm processes the video stream from the on-board camera image by image, so for each image processed, the network generates a coordinate set composed of the center of the tag detected  (x, y) with its dimension given in width. and height (w, h), as well as the probability "s" of belonging to a class [x, y, w, h, s]. These coordinates are then converted to offset angles relative to the center of the coordinate system. These angles are sent to the autopilot so that the drone moves in a way  the marker is always in the center of the image. It is assumed that the angles of the MAV (yaw, pitch, roll) can be neglected when the offset angles are calculated. 

![diagram]({{site.baseurl}}/assets/img/diagram.png){:height="40%" width="40%"}
![ubuntu simulated drone]({{site.baseurl}}/assets/img/sim_drone.jpg){:height="80%" width="80%"}  

The figure above
