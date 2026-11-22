
 SMART WASTE SORTING SYSTEM ESP32 Code

 FEATURES:
 - Conveyor continuous
 - Raspberry Pi sends Plastic/Paper
 - Proximity sensors detect Metal
 - FIFO object queue
 - IR triggers servo sorting
 - Non-blocking servo
 - Blynk bin reminder
 - HX711 weight monitoring

 PART 1/4
*********************************************************/


#define BLYNK_TEMPLATE_ID "TMPL6Kx1VEpYC"
#define BLYNK_TEMPLATE_NAME "Alert notification"
#define BLYNK_AUTH_TOKEN "vgvQj6SZL05Wi3XK5N_eMT_I2BkE5IEs"


#include <WiFi.h>
#include <WiFiClient.h>
#include <BlynkSimpleEsp32.h>
#include "HX711.h"
#include <ESP32Servo.h>



// ================= WIFI =================

char ssid[] = "A06";
char pass[] = "xlcu3689";




// ================= HX711 =================

#define HX711_DT 4
#define HX711_SCK 5


HX711 scale;

float calibration_factor = 663.5;





// ================= SERVO =================

Servo gateServo;


#define SERVO_PIN 15


const int NORMAL_POS  = 0;
const int PLASTIC_POS = 60;
const int METAL_POS   = 170;






// ================= IR SENSOR =================

#define IR_PIN 21






// ================= METAL SENSOR =================

#define SENSOR1 18
#define SENSOR2 19


bool lastS1 = false;
bool lastS2 = false;








// =================================================
// QUEUE SYSTEM
// =================================================


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







// ================= SERVO CONTROL =================


bool servoBusy = false;


unsigned long servoStartTime = 0;


const unsigned long HOLD_TIME = 3500;







// ================= MOTOR =================


int pwmA = 14;

int Ain1 = 26;

int Ain2 = 27;



int pwmB = 32;

int Bin1 = 25;

int Bin2 = 33;



int STBY = 12;



int speedVal = 230;








// ================= BIN SYSTEM =================


bool binFull = false;

bool waitingForRestart = false;


unsigned long emptyTime = 0;



unsigned long lastNotificationTime = 0;


const unsigned long NOTIFICATION_INTERVAL = 60000;







// ================= CURRENT OBJECT =================


int currentObject = 0;







// ================= METAL PRIORITY =================

unsigned long metalTime = 0;

const unsigned long METAL_PRIORITY_TIME = 2000;

// =================================================
// SETUP
// =================================================

void setup()
{


  Serial.begin(115200);



  // ================= BLYNK =================

  Blynk.begin(
    BLYNK_AUTH_TOKEN,
    ssid,
    pass
  );




  // ================= HX711 =================

  scale.begin(
    HX711_DT,
    HX711_SCK
  );


  scale.set_scale(
    calibration_factor
  );


  scale.tare();






  // ================= SERVO =================

  gateServo.attach(
    SERVO_PIN
  );


  gateServo.write(
    NORMAL_POS
  );






  // ================= SENSORS =================

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







  // ================= MOTOR =================


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







// =================================================
// LOOP
// =================================================


void loop()
{


  Blynk.run();






  // =================================================
  // RECEIVE FROM RASPBERRY PI
  // =================================================


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







  // =================================================
  // WEIGHT MONITORING
  // =================================================


  float weight =
  scale.get_units(5);



  if(weight < 0)

  weight = 0;




  Blynk.virtualWrite(
  V0,
  weight
  );






  // ================= BIN FULL =================


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







  // ================= REMINDER =================


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







  // ================= BIN EMPTY =================


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

  // =================================================
  // METAL DETECTION
  // =================================================


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









  // =================================================
  // SORTING PROCESS
  // =================================================


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









  // =================================================
  // SERVO TIMER
  // =================================================


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

// =================================================
// START SORTING FUNCTION
// =================================================


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











// =================================================
// MOTOR START FUNCTION
// =================================================


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











// =================================================
// MOTOR STOP FUNCTION
// =================================================


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
