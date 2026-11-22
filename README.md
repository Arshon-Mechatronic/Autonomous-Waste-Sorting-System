# Autonomous-Waste-Sorting-System

1. Project Overview
   
This project focuses on developing an automated waste detection and sorting system using computer vision and artificial intelligence.

The system aims to identify and classify different types of waste materials, including:

Paper
Plastic Caps
Metal

A Raspberry Pi 5 and camera are used to detect waste materials in real time.

2. Project Objectives
Develop an AI-based waste detection and sorting system.
Identify paper, plastic caps, and metal waste.
Implement an object detection model on a Raspberry Pi 5.
Test the detection system using a conveyor belt setup.
Develop an automated waste separation mechanism.

3. Hardware Components

Raspberry Pi 5
Raspberry Pi Camera Module V2.1 (Sony IMX219)
Inductive Proximity Sensors
Conveyor belt test setup
Power supply

4. Software and Technologies

Python
YOLOv8n
OpenCV
Ultralytics
Roboflow
Google Colab

5. Waste Detection Categories

Current YOLO model classes:
Paper
Plastic Cap

6. How It Works

The camera captures images of waste materials.
The YOLO object detection model processes the images.
The system identifies the waste category.
The sorting mechanism can separate waste according to its category.

7. Dataset and Model Training

The initial object detection dataset was prepared using Roboflow.
The YOLOv8n model was trained using Google Colab.
Model: YOLOv8n
Image size: 640 × 640
Current number of classes: 2
Planned number of classes: 3




