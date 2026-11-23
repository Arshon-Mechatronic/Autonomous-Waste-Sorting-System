Raspberry Pi 5 – Waste Detection System

1. Overview

The Raspberry Pi 5 is responsible for detecting and classifying waste materials using computer vision and artificial intelligence.

It uses a camera module and a trained YOLO object detection model to identify different types of waste in real time.

After identifying the waste material, the Raspberry Pi 5 sends the classification result to the ESP32 for further processing and sorting control.

2. Main Responsibilities

Capture images of waste materials using the Raspberry Pi Camera Module.

Perform real-time object detection using a trained YOLO model.

Identify and classify paper and plastic caps.

Process the detection results.

Send the detected waste category to the ESP32.

3. Hardware

Raspberry Pi 5

Raspberry Pi Camera Module V2.1 (Sony IMX219)

4. Software and Technologies

Python

YOLOv8n

OpenCV

Ultralytics

Picamera2

Roboflow

Google Colab

5. Object Detection Model

The system uses a custom-trained YOLOv8n model.

Current detection classes:

Paper

Plastic Cap

Model specifications:

Model: YOLOv8n

Input image size: 640 × 640

Number of classes: 2

6. Communication with ESP32

After detecting and classifying a waste material, the Raspberry Pi 5 sends the corresponding classification information to the ESP32.

The ESP32 uses this information to make sorting decisions and control the overall system.

7. Source Code

from picamera2 import Picamera2
from ultralytics import YOLO
import serial
import time



# =====================================
# ESP32 SERIAL CONNECTION
# =====================================

while True:

    try:

        esp32 = serial.Serial(
            '/dev/ttyUSB0',
            115200,
            timeout=1
        )


        print(
            "ESP32 connected",
            flush=True
        )


        break


    except serial.SerialException:


        print(
            "Waiting for ESP32...",
            flush=True
        )


        time.sleep(5)



time.sleep(2)





# =====================================
# LOAD YOLO MODEL
# =====================================

model = YOLO(
    "/home/pi5/YOLO_Project/best.pt"
)


print(
    "Model loaded",
    flush=True
)


print(
    model.names,
    flush=True
)






# =====================================
# CAMERA SETUP
# =====================================

picam2 = Picamera2()



picam2.configure(

    picam2.create_preview_configuration(

        main={

            "size":(640,480),

            "format":"RGB888"

        }

    )

)



picam2.start()



print(
    "Camera started",
    flush=True
)







# =====================================
# OBJECT MEMORY
# =====================================


last_object = ""







# =====================================
# MAIN LOOP
# =====================================


while True:



    frame = picam2.capture_array()





    results = model(

        frame,

        conf=0.4,

        verbose=False

    )





    object_found = False





    for box in results[0].boxes:



        class_id = int(box.cls[0])



        class_name = model.names[class_id]



        detected = class_name.lower()



        print(

            "Detected:",

            class_name,

            flush=True

        )




        object_found = True





        # =============================
        # PLASTIC
        # =============================


        if "plastic" in detected:



            if last_object != "PLASTIC":



                esp32.write(

                    b"PLASTIC\n"

                )



                print(

                    "Sent PLASTIC",

                    flush=True

                )



                last_object = "PLASTIC"








        # =============================
        # PAPER
        # =============================


        elif "paper" in detected:



            if last_object != "PAPER":



                esp32.write(

                    b"PAPER\n"

                )



                print(

                    "Sent PAPER",

                    flush=True

                )



                last_object = "PAPER"









        # =============================
        # METAL
        # =============================


        elif "metal" in detected:



            print(

                "Metal detected - ESP32 sensor handles it",

                flush=True

            )



            last_object = "METAL"









    # =================================
    # OBJECT LEFT CAMERA
    # RESET MEMORY
    # =================================


    if not object_found:


        last_object = ""






    time.sleep(0.05)
