# Expressive Mechanics (continued)

This week I continued to work on the expressive mechanics project (see previous week's journal for more details).
</br>

## Electronics
Since my mechanism wasn't working and I wanted to re-evaluate the concept, I decided I would make sure all of the software was squared away before continuing. First of all, I was running out of time and I decided to simplify the subject of the project to a paper airplane instead of a laser cut Snoopy and his dog house (as laser cutting time was very difficult to obtain during this project). Next, I made a sequential diagram and a system diagram to make sure I was accounting for all the parts:
</br>
<img width="800" src="https://github.com/user-attachments/assets/0ce2c36b-f922-4695-aa8b-ff2c8c9fb0f1" />
</br>
<img width="400" src="https://github.com/user-attachments/assets/56011da4-6147-4264-ab78-77c4163dbcbc" />

Then, I tackled combining the servo and DC motor. I wired both the servo and DC motor with the Arduino just by combining the two sample wirings, but realized that pins 9 and 10 on the Arduino can't be used when the servo is attached. Once that was sorted, I attempted to combine the two sample code files in a similar way:

```
/*
*  Example code control servo position with data sent from p5.js
*  in p5 posenet tracks positions of ears: tilt head to move servo
*  p5 code here: https://editor.p5js.org/loopstick/sketches/MWZxoSNoP
*/
#include <Servo.h>
Servo myservo;  // create servo object to control a servo
int pos = 0;    // variable to store the servo position
int prev;

int inc = 1;
int brightness = 10;

#include <L298N.h>
const unsigned int IN1 = 12;
const unsigned int IN2 = 11;
const unsigned int EN = 8;

// the setup routine runs once when you press reset:
void setup() {
  // initialize serial communication 
  Serial.begin(115200);
  myservo.attach(7);  // attaches the servo on pin 7 to the servo object
  Serial.println("Dr. Sudhu Test Code - Hi !"); // so we know something's working!!!
}

void loop() {

  // Check to see if there is any incoming serial data
  if(Serial.available() > 0){
    // If we're here, then serial data has been received
    // Read data off the serial port until we get to the endline delimeter ('\n')
    // Store all of this data into a string
    String rcvdSerialData = Serial.readStringUntil('\n'); 
    // Echo the data back on serial (for debugging purposes)
    Serial.print("Arduino Received: '");
    Serial.print(rcvdSerialData);
    Serial.println("'");

    int value = rcvdSerialData.toInt();
    // Serial.print(" - convert to Int '");
    // Serial.print(value);
    // Serial.println("'");

     pos = map(value, 0, 255, 40, 160);  // could remap here if necessary
    if (pos <= 95 && pos >= 85) {
      myservo.write(90); 
    } else {
      myservo.write(pos); // tell servo to go to position in variable 'pos'
    }

    //move motor at a constant speed
    motor.setSpeed(255);
    motor.forward();

    Serial.print("servo pos: '");
    Serial.print(pos);
    Serial.println("'");
  
    
  }
}
```

However, the code above doesn't work (only the servo moves). After reading the Arduino forums, I learned that I needed to control the motor through the servo library and the analogWrite() function (I couldn't get a good grasp on why but for some reason, the motor library gets overwritten). 

```
/*
*  Example code control servo position with data sent from p5.js
*  in p5 posenet tracks positions of ears: tilt head to move servo
*  p5 code here: https://editor.p5js.org/loopstick/sketches/MWZxoSNoP
*/
#include <Servo.h>
Servo myservo;  // create servo object to control a servo
int pos = 0;    // variable to store the servo position
int prev;

int inc = 1;
int brightness = 10;

#include <L298N.h>
const unsigned int IN1 = 12;
const unsigned int IN2 = 11;
const unsigned int EN = 8;
L298N motor(EN, IN1, IN2);


// the setup routine runs once when you press reset:
void setup() {
  // initialize serial communication 
  Serial.begin(115200);
  myservo.attach(7);  // attaches the servo on pin 7 to the servo object
  pinMode(IN1, OUTPUT);
  pinMode(IN2, OUTPUT);
  pinMode(EN, OUTPUT);
  digitalWrite(EN, LOW);
  Serial.println("Dr. Sudhu Test Code - Hi !"); // so we know something's working!!!
}

void loop() {

  // Check to see if there is any incoming serial data
  if(Serial.available() > 0){
    // If we're here, then serial data has been received
    // Read data off the serial port until we get to the endline delimeter ('\n')
    // Store all of this data into a string
    String rcvdSerialData = Serial.readStringUntil('\n'); 
    // Echo the data back on serial (for debugging purposes)
    Serial.print("Arduino Received: '");
    Serial.print(rcvdSerialData);
    Serial.println("'");

    int value = rcvdSerialData.toInt();
    // Serial.print(" - convert to Int '");
    // Serial.print(value);
    // Serial.println("'");

     pos = map(value, 0, 255, 60, 120);  // could remap here if necessary
    if (pos <= 95 && pos >= 85) {
      myservo.write(90); 
    } else {
      myservo.write(pos); // tell servo to go to position in variable 'pos'
    }
       
    Serial.print("servo pos: '");
    Serial.print(pos);
    Serial.println("'");
  
    digitalWrite(IN1, HIGH);
    digitalWrite(IN2, LOW);
    analogWrite(EN, 255);
  }
}
```
You can see this working [here](https://drive.google.com/file/d/17JVa0-EzoJUUx7lgKT_ewQEmvtRk4pXt/view?usp=sharing), where the servo responds to my movement and the DC motor runs continuously. 

After the crank and slider failed, I returned to my original concept of the rack and pinion. Knowing I would have to move the gear back and forth with the DC motor, I found [some suggestions online](https://forum.arduino.cc/t/control-dc-motor-using-millis/477543/4) about how to time the motor's movement and change directions.

## Fabrication
Switching gears (haha), I moved to making the rack and pinion. I used a [gear generator](https://evolventdesign.com/pages/spur-gear-generator?srsltid=AfmBOoo9ogfie3oP8P_NJ532cGcqHbzFfdTRe5e0UyVMZ8yJirWwyB0U) and based my measurements off of this example mechanism from class I had measured:
</br>
<img width="400" src="https://github.com/user-attachments/assets/7932d143-bd43-48d7-8cf6-59417bac2abb" />
</br>

Then I laser cut the pieces and a box (not pictured):
</br>
<img width="400" src="https://github.com/user-attachments/assets/8f64de09-ebb7-48bd-b56e-4f6d5c11d8a4" />
</br>
<img width="400" src="https://github.com/user-attachments/assets/c6d19dad-d6e0-4423-8822-b8fa64858ec8" />

And then began testing it out in the box (which I realized was a bit too small for real testing):
<img width="400" src="https://github.com/user-attachments/assets/78e768cd-df0b-424e-bab3-365664495d7d" />

The 3D printed support was found in the studio, made by Alistair for something else so I don't have a file for it unfortunately. However, the size ratio was so off it didn't work with the rack and pinion anyway, and it was clear I needed to make custom rails. 

There is roughly a day of intermission as I lost my wallet and wasn't able to keep working on the project or access the makerspace :(. I was kindly given an extension (thank you, Lauryn!), but I still needed to make something for the presentation (two hours from re-obtaining my wallet). I focused on showing the tilt motion and everything I had working at that point:
<img width="600" src="https://github.com/user-attachments/assets/d1784583-2fce-4697-972d-fc52103025d2" />

I reused one of those found 3D supports to hold part of the platform steady and secured the servo inside the box to move the arm linkage, creating the [tilt I had hoped for with the paper airplane](https://drive.google.com/file/d/1B-Ng2h4IQfu0Mg6GuOBFmFKSxoFJ9-MP/view?usp=sharing). (Additional video [here](https://drive.google.com/file/d/1JsiY2LeU6jo2QNHn_s25C7oQ-NvFJD5A/view?usp=sharing))

Thanks to the feedback from presenting, I found a lot of other people had used dwelling or cam-and-piston mechanisms. I took inspiration from that and looked into that kind of mechanism more, finding it to be simpler and more effective for what I wanted than the rack and pinion. From all the examples I had seen presented, I had only seen circular cams being used. My search showed that a more organic shape would ease the movement in and out without changing anything in the code or speed, which I thought would make for a more natural flying motion.

I laser cut a custom cam, feet for the sticks I'd been using in the linkages, and some new arms:
</br>
<img width="600" src="https://github.com/user-attachments/assets/440efcdb-f274-455f-b94a-05b7bcbb098d" />
</br>

Then made a cardboard box to test for size so I didn't waste anymore wood. The dwelling mechanism worked immediately, though I did find sanding the edges of the foot (like a chamfer? is that the right word?) helped smooth the movement considerably ([video with motor only](https://drive.google.com/file/d/11PCSYXRyVZcygr62Yw9d0sp28HLZaK-o/view?usp=sharing), [video with the whole mechanism assembled](https://drive.google.com/file/d/1-7B9DKwvEkOd6kkn4rpfGFMlqwYq2rga/view?usp=sharing))

I cut a new box, tucked away the electronics and made the final version:
</br>
<img width="400" src="https://github.com/user-attachments/assets/80c789a1-7e06-48bb-ae42-0c2d5a9171d6" />
<img width="400" src="https://github.com/user-attachments/assets/26cf2e55-ee3b-4f97-8517-a8c0aefb29c3" />

You can see it working [here](https://drive.google.com/file/d/1s3mT7FCFI00flDbgREQtFK_EKq_dKElt/view?usp=sharing). 

## Reflection

This was a really challenging project, and I'm really proud of what I accomplished. I think I should have looked into mechanisms that provide the kind of movement I'm looking for first before just jumping into what I thought might work. If I had more time, I would try to add a little bit more smoothing on the servo movement since even asking AI to help didn't really work (I think it's partially a lag issue and partially the values are weird). I think building more sturdy cardboard boxes for prototyping is a helpful technique that I really only tried at the end since Studio Foundations made me build a nice cardboard box, but it would save a lot of wood costs. I learned a lot about pose nets, timing functions, and new mechanisms and I'm excited to use what I learned in the next project. 

As always, thanks for reading!
