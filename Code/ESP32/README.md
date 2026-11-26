ESP32 – Main Processing and Control Unit

1. Project Overview

The ESP32 acts as the main processing and control unit of the Autonomous Waste Sorting System. It coordinates waste transportation, classification-based sorting, bin weight monitoring, and bin-full notifications.

The system receives waste classification data from the Raspberry Pi 5, which uses a YOLO-based object detection model to identify paper and plastic waste. An inductive proximity sensor is also used to detect metal waste.

Based on the detected waste category, the ESP32 controls a servo motor to redirect the sorting bin to the appropriate waste collection position.

The ESP32 continuously monitors the weight of the collection bin using a load cell. When the weight reaches a predefined limit, the system stops the conveyor belt and sends a push notification to the garbage disposer through the Blynk mobile application.

Once the bin is emptied, the system automatically resumes conveyor operation after a 10-second delay.


2. Main Responsibilities

Control the conveyor belt using N20 geared DC motors.

Receive paper and plastic classification data from the Raspberry Pi 5.

Detect metal waste using an inductive proximity sensor.

Process waste classification information and determine the appropriate sorting operation.

Control the servo motor to redirect the collection bin according to the detected waste category.

Continuously monitor the waste collection bin weight using a load cell.

Stop the conveyor belt when the predefined weight limit is reached.

Send bin-full push notifications to the garbage disposer through the Blynk mobile application.

Resume conveyor operation 10 seconds after the bin has been cleared.


3. Hardware Components

ESP32 Development Board

N20 Geared DC Motors

Motor Driver

Servo Motor

Inductive Proximity Sensor

Load Cell

Load Cell Amplifier Module

Raspberry Pi 5

12v Power Supply

MP1584 Step-Down Buck Converters


4. Software and Technologies

Arduino IDE

C/C++ (Arduino Framework)

ESP32 Board Package

Blynk IoT Platform

USB Serial Communication


5. Communication with Raspberry Pi 5

The ESP32 communicates with the Raspberry Pi 5 through a USB cable using USB Serial Communication.

The Raspberry Pi 5 performs AI-based object detection using a trained YOLO model and identifies paper and plastic waste.

After classification, the Raspberry Pi 5 transmits the detected waste category to the ESP32 through the USB serial connection.

The ESP32 processes the received classification data and coordinates the corresponding sorting operation.

Metal detection is handled separately by the inductive proximity sensor connected to the ESP32.


6. Working Principle

6.1 Conveyor Belt Operation

The ESP32 controls the conveyor belt using N20 geared DC motors. The conveyor transports waste materials through the detection and sorting process.


6.2 Waste Detection and Classification

The system identifies three waste categories:

Paper – Detected by the Raspberry Pi 5 using YOLO object detection.

Plastic – Detected by the Raspberry Pi 5 using YOLO object detection.

Metal – Detected using an inductive proximity sensor.


6.3 Automatic Waste Sorting

After identifying the waste material, the ESP32 determines the appropriate sorting position.

A servo motor is controlled to redirect the collection bin to the corresponding waste category, allowing the detected material to be sorted into its designated collection area.


6.4 Bin Weight Monitoring

A load cell installed beneath the collection bin continuously measures the weight of the collected waste.

The weight readings are transmitted to the ESP32, which compares the measured weight against a predefined weight limit.


6.5 Bin-Full Notification

When the measured weight reaches the predefined limit:

The ESP32 stops the conveyor belt.

A bin-full notification is sent to the garbage disposer through the Blynk mobile application.

The garbage disposer is alerted that the collection bin needs to be emptied.


6.6 Automatic System Restart

After the collection bin has been emptied, the system waits for 10 seconds before restarting the conveyor belt.

The sorting process then resumes automatically.


7. Blynk Mobile Application

The Blynk IoT platform is used to notify the garbage disposer when the waste collection bin reaches its predefined weight limit.

Notification feature:

Push notification when the bin is full.

Alerts the garbage disposer when waste needs to be removed.


6. Source Code

#define BLYNK_TEMPLATE_ID ""

#define BLYNK_TEMPLATE_NAME "Alert notification"

#define BLYNK_AUTH_TOKEN ""


#include <WiFi.h>

#include <WiFiClient.h>

#include <BlynkSimpleEsp32.h>

#include "HX711.h"

#include <ESP32Servo.h>



WIFI 
----

char ssid[] = "";
char pass[] = "";


HX711 
-----

#define HX711_DT 4

#define HX711_SCK 5


HX711 scale;

float calibration_factor = 663.5;




SERVO 
-----

Servo gateServo;


#define SERVO_PIN 15


const int NORMAL_POS  = 0;

const int PLASTIC_POS = 60;

const int METAL_POS   = 170;




IR SENSOR
---------

#define IR_PIN 21




METAL SENSOR
------------

#define SENSOR1 18

#define SENSOR2 19


bool lastS1 = false;

bool lastS2 = false;


QUEUE SYSTEM
------------


#define QUEUE_SIZE 10


#define METAL   1

#define PLASTIC 2

#define PAPER   3



int objectQueue[QUEUE_SIZE];


int queueFront = 0;

int queueRear = 0;



bool queueEmpty()
{

  return queueFront == queueRear;

}




bool queueFull()
{

  return ((queueRear + 1) % QUEUE_SIZE) == queueFront;

}






void enqueueObject(int objectType)
{


  if(!queueFull())
  {

    objectQueue[queueRear] = objectType;


    queueRear =
    (queueRear + 1) % QUEUE_SIZE;


    Serial.println("OBJECT ADDED TO QUEUE");

  }

  else
  {

    Serial.println("QUEUE FULL");

  }


}






int dequeueObject()
{


  if(!queueEmpty())
  {


    int object =
    objectQueue[queueFront];



    queueFront =
    (queueFront + 1) % QUEUE_SIZE;



    return object;


  }


  return 0;


}





SERVO CONTROL 
-------------


bool servoBusy = false;


unsigned long servoStartTime = 0;


const unsigned long HOLD_TIME = 3500;





MOTOR 
-----


int pwmA = 14;

int Ain1 = 26;

int Ain2 = 27;



int pwmB = 32;

int Bin1 = 25;

int Bin2 = 33;



int STBY = 12;


int speedVal = 230;



BIN SYSTEM 
----------

bool binFull = false;

bool waitingForRestart = false;


unsigned long emptyTime = 0;



unsigned long lastNotificationTime = 0;


const unsigned long NOTIFICATION_INTERVAL = 60000;







CURRENT OBJECT 
--------------

int currentObject = 0;







METAL PRIORITY
--------------
  
unsigned long metalTime = 0;

const unsigned long METAL_PRIORITY_TIME = 2000;

- SETUP

void setup()
{


  Serial.begin(115200);


 BLYNK App
 ---------

  Blynk.begin(
    BLYNK_AUTH_TOKEN,
    ssid,
    pass
  );




  HX711
  ----- 

  scale.begin(
    HX711_DT,
    HX711_SCK
  );


  scale.set_scale(
    calibration_factor
  );


  scale.tare();






  SERVO
  -----
  
  gateServo.attach(
    SERVO_PIN
  );


  gateServo.write(
    NORMAL_POS
  );






  SENSORS
  -------

  pinMode(
    IR_PIN,
    INPUT
  );


  pinMode(
    SENSOR1,
    INPUT_PULLUP
  );


  pinMode(
    SENSOR2,
    INPUT_PULLUP
  );







 MOTOR
 -----

  pinMode(
    STBY,
    OUTPUT
  );


  digitalWrite(
    STBY,
    HIGH
  );



  pinMode(
    Ain1,
    OUTPUT
  );


  pinMode(
    Ain2,
    OUTPUT
  );


  pinMode(
    Bin1,
    OUTPUT
  );


  pinMode(
    Bin2,
    OUTPUT
  );





  ledcAttach(
    pwmA,
    1000,
    8
  );


  ledcAttach(
    pwmB,
    1000,
    8
  );




  startMotors();



  Serial.println(
    "ESP32 QUEUE SYSTEM READY"
  );


}








LOOP
----



void loop()
{


  Blynk.run();






 
  RECEIVE FROM RASPBERRY PI
  -------------------------
  


  if(Serial.available())
  {


    String command =
    Serial.readStringUntil('\n');


    command.trim();





    if(command == "PLASTIC")
    {


      if(
      millis() - metalTime > METAL_PRIORITY_TIME
      )
      {


        Serial.println(
        "PLASTIC RECEIVED"
        );


        enqueueObject(PLASTIC);


      }

      else
      {


        Serial.println(
        "PLASTIC IGNORED - METAL PRIORITY"
        );


      }


    }






    else if(command == "PAPER")
    {


      enqueueObject(PAPER);


      Serial.println(
      "PAPER RECEIVED"
      );


    }


  }







  WEIGHT MONITORING
  -----------------
 

  float weight =
  scale.get_units(5);



  if(weight < 0)

  weight = 0;




  Blynk.virtualWrite(
  V0,
  weight
  );




  BIN FULL
  --------


  if(weight >= 3.0 && !binFull)
  {


    stopMotors();


    Blynk.logEvent(
    "bin_full",
    "Clear the bin"
    );



    Serial.println(
    "BIN FULL"
    );



    binFull = true;


    lastNotificationTime =
    millis();


  }







  REMINDER
  --------

  if(binFull)
  {


    if(
    millis() - lastNotificationTime
    >= NOTIFICATION_INTERVAL
    )
    {


      Blynk.logEvent(
      "bin_full",
      "Clear the bin"
      );


      Serial.println(
      "BIN REMINDER SENT"
      );


      lastNotificationTime =
      millis();


    }


  }







  BIN EMPTY
  ---------


  if(binFull && weight < 3.0)
  {


    if(!waitingForRestart)
    {


      emptyTime =
      millis();


      waitingForRestart = true;


    }




    if(
    millis() - emptyTime > 10000
    )
    {


      startMotors();


      binFull = false;


      waitingForRestart = false;


      Serial.println(
      "BIN EMPTY - SYSTEM STARTED"
      );


    }


  }


 METAL DETECTION
 --------------- 


  int s1 =
  digitalRead(SENSOR1);


  int s2 =
  digitalRead(SENSOR2);




  bool currentS1 =
  (s1 == LOW);



  bool currentS2 =
  (s2 == LOW);







  if(currentS1 && !lastS1)
  {


    Serial.println(
    "METAL SENSOR 1 DETECTED"
    );


    enqueueObject(METAL);


    metalTime =
    millis();


  }






  if(currentS2 && !lastS2)
  {


    Serial.println(
    "METAL SENSOR 2 DETECTED"
    );


    enqueueObject(METAL);


    metalTime =
    millis();


  }






  lastS1 =
  currentS1;


  lastS2 =
  currentS2;









  
  SORTING PROCESS
  ---------------
 

  if(!servoBusy)
  {


    if(!queueEmpty())
    {


      currentObject =
      dequeueObject();



      startSorting(
      currentObject
      );


    }


  }









 
  SERVO TIMER
  -----------

  if(servoBusy)
  {


    if(
    millis() - servoStartTime
    >= HOLD_TIME
    )
    {


      gateServo.write(
      NORMAL_POS
      );



      servoBusy = false;



      Serial.println(
      "SERVO RETURNED"
      );


    }


  }







  delay(20);


}   // END LOOP


START SORTING FUNCTION
----------------------


void startSorting(int objectType)
{


  bool objectAtIR =
  (digitalRead(IR_PIN) == LOW);





  if(objectAtIR)
  {


    if(objectType == METAL)
    {


      Serial.println(
      "SORTING METAL"
      );


      gateServo.write(
      METAL_POS
      );


    }






    else if(objectType == PLASTIC)
    {


      Serial.println(
      "SORTING PLASTIC"
      );


      gateServo.write(
      PLASTIC_POS
      );


    }






    else if(objectType == PAPER)
    {


      Serial.println(
      "SORTING PAPER"
      );


      gateServo.write(
      NORMAL_POS
      );


    }






    servoBusy = true;


    servoStartTime =
    millis();


  }






  else
  {


    Serial.println(
    "OBJECT WAITING FOR IR"
    );


    // return object to queue

    enqueueObject(objectType);


  }


}












MOTOR START FUNCTION
--------------------


void startMotors()
{


  digitalWrite(
  Ain1,
  HIGH
  );


  digitalWrite(
  Ain2,
  LOW
  );



  digitalWrite(
  Bin1,
  HIGH
  );


  digitalWrite(
  Bin2,
  LOW
  );




  ledcWrite(
  pwmA,
  speedVal
  );


  ledcWrite(
  pwmB,
  speedVal
  );


}












MOTOR STOP FUNCTION
-------------------


void stopMotors()
{


  ledcWrite(
  pwmA,
  0
  );


  ledcWrite(
  pwmB,
  0
  );




  digitalWrite(
  Ain1,
  LOW
  );


  digitalWrite(
  Ain2,
  LOW
  );



  digitalWrite(
  Bin1,
  LOW
  );


  digitalWrite(
  Bin2,
  LOW
  );


}


