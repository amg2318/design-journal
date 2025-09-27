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
Here's the final wiring (spoiler: it remains the same through to the final):
</br>
<img width="400" src="https://github.com/user-attachments/assets/7672e580-977d-416d-88c3-d090a8f597fd" />

Here's the final circuit:
</br>
<img width="600" src="https://github.com/user-attachments/assets/5d9641a2-ab84-4781-aa53-85ddbdab4784" />

With the movement I had been picturing, I set up a non-permanent test of concept:
</br>
<img width="400" src="https://github.com/user-attachments/assets/27271f11-bd47-4087-b3fd-153b0fa98c30" />

This test of concept involved tying strings to a dowel rod that was taped to the servo. The combination of slow servo speed and the looseness of the strings meant that it [barely moved the paper](https://drive.google.com/file/d/1NMr_Zdx49CXrNykEsECmnS2-BhMcqVXq/view?usp=sharing), so that ultimately failed. I decided to look into different mechanisms for movement, specifically in two opposite directions and found a reverse-motion linkage:
</br>
<img width="492" height="479" alt="image" src="https://github.com/user-attachments/assets/ef2586e0-e81b-40ca-b2bc-611d4001a014" />

Based on this, I sketched out a way to use the linkage system:
</br>
<img width="400" src="https://github.com/user-attachments/assets/b264430d-5289-4ab9-bf8b-ab69fd1f1949" />

Then I designed some potential pieces for the linkage based on my drawings in Illustrator for laser cutting:
</br>
<img width="600" src="https://github.com/user-attachments/assets/867e1c88-c6d0-41a2-af58-3a3f97022a1b" />

Here's a picture of the parts getting laser cut:
</br>
<img width="400" src="https://github.com/user-attachments/assets/35a83287-1cba-4add-b214-f745939c60b3" />

Then, I started trying different ways to assemble the linkage using tape and pipe cleaners (so I could keep re-using the parts), changing the angles, places of attachment, and movement. [Here's the first proof of concept](https://drive.google.com/file/d/1hoHI1APXd9pxgtBTam5i-KhZc2EHECET/view?usp=sharing):
</br>
<img width="400" src="https://github.com/user-attachments/assets/ea4ff0d6-8381-4ff1-af69-21a87d18ec7d" />

It seemed to open the origami box the way I wanted it to, so I moved on to putting it into the box. It proved to be way more difficult than I expected, and didn't really fit in the box I had. None of the set-ups stayed together in the box for very long, but you can see one brief attempt [here](https://drive.google.com/file/d/1-wvTM3H8LO8OP2i2cf0Dmh8Yfm6cZCy5/view?usp=sharing). After failed attempts, I realized changing the whole orientation to be horizontal would make it much easier to support:
</br>
<img width="600" src="https://github.com/user-attachments/assets/013bd07c-41e0-4140-8b56-5322b5db603c" />
</br>
<img width="400" src="https://github.com/user-attachments/assets/c079a898-a01d-4a5d-aeb4-6c3d365267dd" />
<img width="400" src="https://github.com/user-attachments/assets/ecfc6fca-9399-44d4-b8b2-dd873032250c" />

The dowel rod in the center (as the stationary pivot) could be much shorter and servo could be attached to the bottom of the box instead of suspended. I also experimented with small cardboard boxes and slot size/placement to control the path of the linkage arms. However, I was doing a lot of this experimentation without the servo attached or running code, leading me to realize that none of this actually worked when hooked up:
- [This video](https://drive.google.com/file/d/13kcttxg_DpbsWDXGM--41ewYwYjlaw-7/view?usp=sharing) shows it barely moving because of too many supports and limiters.
- [This video](https://drive.google.com/file/d/10Q72l5ZsjxL1bb99l8zoAEU2VY5nzeWZ/view?usp=sharing) shows that with the supports moved and the center dowel removed compeletely, it starts to actually move the origami.

Throughout these changes, the servo would sometimes flip out and rip everything out of the box so I tried to smooth the movement in the code:
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
  Serial.print("Analog IN A0 value = ");
  Serial.println(sensorValue);
  Serial.println(val);


  if (sensorValue >= 100) { //for normal (high) values, servo should be stationary
    for(int i=1; i<80; i++) {
      myservo.write(100 + (i*10));
      delay(100);
    }
  } else if (sensorValue < 40){ //if something is blocking/covering the LDR, servo should turn
    
    delay(500);
  } else { //transitioning (so it doesn't wiggle)
    val = map(sensorValue, 40, 450, 0, 180); 
    int diff = abs(prev - val);
    if (diff > 3) {
      myservo.write(val);
    } 
  }
  prev = val;
  delay(50);
  //myservo.write(0);
}
```

The first few attempts didn't work, but after some simplification I finalized the code to ease in and out of the two positions I wanted at small increments:
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
  val = map(sensorValue, 40, 160, 90, 0);
  //for debugging and value setting
  Serial.print("Analog IN A0 value = ");
  Serial.println(sensorValue);

  if (sensorValue < 40) { 
    for(int i=1; i<+10; i++) {
      myservo.write(100 - i*10);
      delay(15);
    }
    myservo.write(0);
    val = 0;
    delay(500);
  } else {
      if (prev != 100) {
        for(int i=1; i<+10; i++) {
          myservo.write(i*10);
          delay(15);
        }
      } else {
        myservo.write(100);
      }  
      //delay(500);
    val = 100;
  } 
  prev = val;
  delay(500);
}
```

[This video](https://drive.google.com/file/d/1QO7pgr_MqTX9molHYryWiB__2kb2LXck/view?usp=sharing) shows it finally working! But it's really shaky and the servo is vibrating. Unfortunately, right after this video, the servo flipped and tore out the supports. I was never able to reproduce that movement despite trying to put things back exactly how I had it before. As time was running out and the origami and the box I had been using were getting progressively more beat up, I tried my best to replicate the same set up in a new box, resulting in this set up for Thursday's presentation:
</br>
<img width="600" src="https://github.com/user-attachments/assets/ad8676e6-211e-4ca4-aa7e-e47f8f508f15" />

Following the presentation, I cleaned up the box for the final photos and [demo video](https://drive.google.com/file/d/1cecbN05tKFCYqVWOgXeM1R4IDodoEn1U/view?usp=sharing), as well as slightly improved up on the internal set up:</br>
<img width="600" src="https://github.com/user-attachments/assets/b9c509df-28ed-451d-8e74-d5983b6ca459" />
<img width="600" src="https://github.com/user-attachments/assets/dcdb5ca7-cd89-4dd0-b041-386cd99bfcb8" />

</br>
<img width="400" src="https://github.com/user-attachments/assets/99b155fa-a6c0-4f55-95c9-28ae4b2343f2" />
<img width="400" src="https://github.com/user-attachments/assets/55866e94-6c15-452c-90e5-e2693a5464d9" />

In the end I removed all of the supports to allow for greater movement without over twisting the servo. The linkage movement is nearly diagonal but it would eventually kick out any forms of support I tried to add to stablize it. Another important change I made was cutting the dowels connected to the origami so that they didn't drag against the bottom of the box or snag on the cables. I found that attaching to the origami from a bit further away and from the back of the fold was more effective for the movement. I turned the front panel into a door to access the electronics and for people to get a peak at the mechanism inside as well.

Overall, I'm pretty proud of what I was able to make in the time alotted even though I spent many late nights on it and was very stressed out about the deadline. I felt like I chose a fairly difficult movement and mechanism, which was both a fun challenge but also a huge stressor. I think if I were to re-do this project or continue working on it, I would try a double-sided rack and pinion gear mechanism instead. I would have still faced a significant technical challenge (as I have little experience in building mechanisms like this) but it would have allowed for a more compact box and greater control over the movement of the origami. I would also try different smoothing functions that could have given the movement more personality than simply opening and closing, maybe to reflect the emotion of "excitement" I had initially hoped to convey through this piece. 

Thanks for reading!
