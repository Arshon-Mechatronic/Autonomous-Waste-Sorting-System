# Autonomous-Waste-Sorting-System

1. Project Overview
   
This project focuses on developing an automated waste detection and sorting system using computer vision and artificial intelligence.

The system aims to identify and classify different types of waste materials, including:

Paper

Plastic Cap

Metal

A Raspberry Pi 5 and camera are used to detect waste materials in real time.

2. Hardware Components

Raspberry Pi 5

Raspberry Pi Camera Module V2.1 (Sony IMX219)

ESP32 

Inductive Proximity Sensors

Conveyor belt 

N20 Gear Motors

IR Sensor 

MG995 Servo Motor

Plastic Bin

3. Software and Technologies

Python

YOLOv8n

OpenCV

Ultralytics

Roboflow

Google Colab

4. Waste Detection Categories

Current YOLO model classes:

Paper

Plastic Cap

5. How It Works

The camera captures images of waste materials.

The YOLO object detection model processes the images.

The system identifies the waste category.

The sorting mechanism can separate waste according to its category.

6. Dataset and Model Training

The initial object detection dataset was prepared using Roboflow.

The YOLOv8n model was trained using Google Colab.

Model: YOLOv8n

Image size: 640 × 640

Current number of classes: 2

Planned number of classes: 3




