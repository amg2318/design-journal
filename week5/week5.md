# Emotive Origami Project

This week the main focus is the [emotive origami project](https://docs.google.com/document/d/1Cw4nj2my8vcnbyRD-siceU4XP0ueKQAaXggEdhvWza0/edit?usp=sharing):
> Using your knowledge of Arduino, sensors, and servo control, design and build a unique moving sculpture that communicates a specific emotion.  The goal is to have a kinetic art piece that communicates an emotion and captivates the viewer. The sculpture should respond to user interaction through sensor input, producing meaningful motion that expresses the chosen emotion.
>
> Input: Sensor data collected and processed by a microcontroller
>
> Output: Actuator (e.g., servo) motion driven by the processed data
>
> Interaction: The sensor-to-motion relationship should be clear and understandable

As soon as I read the brief, I wanted to try to use the [Japanese Tato box](https://youtu.be/urDWQM2gV6c?si=_Dme3z4LiOegoztX) that I've been making since I was a kid. My mom would use patterned origami paper to make these boxes and put a small candy within as a treat when I was little, so I want to try to replicate the excitement I had when opening them. The idea is for the box to open and unfurl when a hand reaches for it. 

[Here's](https://drive.google.com/file/d/1JVSbocWWL25QVB7VniqdYrThjr5bHCYW/view?usp=sharing) what it looks like:
</br>
<img width="400" src="https://github.com/user-attachments/assets/dc8a663c-4ac1-497a-b341-52f939f8fbff" />
<img width="400" alt="image" src="https://github.com/user-attachments/assets/adc1c03f-566e-4745-ba7c-dffa2c499c03" />
</br>

## Electronics: Servo and Ultrasonic

In preparation for the origami project, I wanted the values for the ultrasonic to control the movement of the servo.

Here's the wiring:
</br>
<img width="600" src="https://github.com/user-attachments/assets/b70f7be0-9c6a-4ea7-9ebd-4b70d25d7d63" />

Here's the circuit:
</br>
<img width="600" src="https://github.com/user-attachments/assets/6659eb39-fc59-4de1-8db7-9b71b907b309" />

Here's the code:
```
#include <Ultrasonic.h>
#include <Servo.h>
/*
 * Pass as a parameter the trigger and echo pin, respectively,
 * or only the signal pin (for sensors 3 pins), like:
 * Ultrasonic ultrasonic(13);
 */
Ultrasonic ultrasonic(11, 12);
int distCM;
int distInch;

Servo myservo;
int val;

void setup() {
  Serial.begin(9600);
  myservo.attach(9); // initializing servo
}

void loop() {
  distCM = ultrasonic.read(CM);
  Serial.print("Distance in CM: ");
  Serial.println(distCM);
  delay(1000);
  //Move the servo accordingly
  val = map(distCM, 0, 357, 0, 180);
  myservo.write(val);
  delay(15); // buffer
}
```

I found that the ultrasonic values fluctuate way too much and are too unpredictable to be useful for what I wanted to do. I tried the [PIR using the Adafruit tutorial](https://learn.adafruit.com/pir-passive-infrared-proximity-motion-sensor/using-a-pir-w-arduino). You can see a video of that [here](https://drive.google.com/file/d/1O_spvbCgLP3Sic6Dcn6udEKfOVKHLVfW/view?usp=sharing). Instead, [I decided to use the LDR](https://drive.google.com/file/d/1GvJwql4XiRaD6fX87av19kYw-pYz_lLC/view?usp=sharing):

```
#include <Servo.h>
Servo myservo;
int sensorValue;
int val;
int prev;

void setup() {
  myservo.attach(12); // initializing servo
  Serial.begin(9600);
}

void loop() {
  sensorValue = analogRead(A0); //read in LDR values

  //for debugging and value setting
  //Serial.print("Analog IN A0 value = ");
  //Serial.println(sensorValue);
  //Serial.println(val);


  if (sensorValue >= 100) { //for normal (high) values, servo should be stationary
    myservo.write(0);
  } else if (sensorValue < 40){ //if something is blocking/covering the LDR, servo should turn
    myservo.write(180);
    delay(500);
  } else { //transitioning (so it doesn't wiggle)
    val = map(sensorValue, 40, 450, 180, 0); 
    int diff = abs(prev - val);
    if (diff > 3) {
      myservo.write(val);
    } 
  }
  prev = val;
  delay(50);
}
```
Here's the wiring:
</br>
<img width="400" src="https://github.com/user-attachments/assets/7672e580-977d-416d-88c3-d090a8f597fd" />

With the movement I had been picturing, I set up a non-permanent test of concept:
</br>
<img width="400" src="https://github.com/user-attachments/assets/27271f11-bd47-4087-b3fd-153b0fa98c30" />

This test of concept involved tying strings to a dowel rod that was taped to the servo. The combination of slow servo speed and the looseness of the strings meant that it [barely moved the paper](https://drive.google.com/file/d/1NMr_Zdx49CXrNykEsECmnS2-BhMcqVXq/view?usp=sharing), so that ultimately failed. I'm going to try cutting some skewers for the arms with string joints or something so that the sides are pulled more horizontally. 



