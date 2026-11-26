Autonomous Waste Sorting System

1. Project Overview

This project focuses on developing an automated waste detection and sorting system using computer vision, artificial intelligence, and embedded systems.

The system is designed to identify and classify three types of waste materials:

Paper

Plastic Cap

Metal

The Raspberry Pi 5 performs AI-based waste detection using a camera and a trained YOLOv8n object detection model. An inductive proximity sensor is used to detect metal waste.

The ESP32 acts as the main processing and control unit, receiving waste classification data from the Raspberry Pi 5 through USB serial communication and coordinating the automatic sorting process.

The system also monitors the weight of collected waste and sends a push notification through the Blynk mobile application when a collection bin reaches its predefined weight limit.


2. Hardware Components

Processing and Detection

Raspberry Pi 5

Raspberry Pi Camera Module V2.1 (Sony IMX219)

ESP32 Development Board

Inductive Proximity Sensor

IR Sensor

Mechanical and Sorting Components

Conveyor Belt

N20 Geared DC Motors

MG995 Servo Motor

Plastic Waste Collection Bins

Monitoring Components

Load Cell

Load Cell Amplifier Module


3. Software and Technologies

Python

C/C++ (Arduino Framework)

YOLOv8n

OpenCV

Ultralytics

Roboflow

Google Colab

Arduino IDE

Blynk IoT Platform


4. Waste Detection Categories

Paper

Plastic Cap

Metal


5.Current YOLO model classes:

Paper

Plastic Cap

Metal detection is performed separately using an inductive proximity sensor.


6. Working Principle

6.1 Waste Transportation

The conveyor belt, driven by N20 geared DC motors, transports waste materials through the detection and sorting process. The ESP32 controls conveyor operation.


6.2 Waste Detection and Classification

The Raspberry Pi 5 uses the Raspberry Pi Camera Module to capture images of waste materials.

The trained YOLOv8n model processes the images and identifies paper and plastic caps.

An inductive proximity sensor connected to the ESP32 detects metal waste.


6.3 Communication Between Controllers

The Raspberry Pi 5 communicates with the ESP32 through a USB cable using USB serial communication.

After detecting and classifying paper or plastic waste, the Raspberry Pi 5 sends the corresponding classification information to the ESP32.

The ESP32 also processes the metal detection signal from the inductive proximity sensor.


6.4 Automatic Waste Sorting

The ESP32 processes the waste classification information and controls the MG995 servo motor to redirect the collection bin to the appropriate waste collection position.

This allows the system to automatically separate paper, plastic caps, and metal waste.


6.5 Waste Weight Monitoring

A load cell installed beneath the collection bin continuously measures the weight of the collected waste.

The ESP32 monitors the weight readings and compares them with a predefined weight limit.


6.6 Bin-Full Notification

When the collected waste reaches the predefined weight limit:

The ESP32 stops the conveyor belt.

A push notification is sent to the garbage disposer through the Blynk mobile application.

The garbage disposer is alerted that the collection bin needs to be emptied.


6.7 Automatic Restart

Once the collection bin has been cleared, the system waits for 10 seconds before automatically restarting the conveyor belt.

The waste detection and sorting process then resumes.


7. Dataset and Model Training

The object detection dataset was prepared using Roboflow.

The YOLOv8n object detection model was trained using Google Colab.

Model Specifications:

Model: YOLOv8n

Image Size: 640 × 640

Current Number of YOLO Classes: 2

Current Classes: Paper and Plastic Cap


8. Communication 

The Raspberry Pi 5 and ESP32 communicate through USB serial communication.

Raspberry Pi 5: Performs AI-based detection and sends paper and plastic classification results.

ESP32: Acts as the main processing and control unit, handles metal detection, controls the conveyor and servo motor, monitors waste weight, and manages Blynk notifications.
