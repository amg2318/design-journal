## Potentiometer Experiment

For this week's electronics experiment, I decided I would try to combine the RGB LED, potentiometer and the servo. The goal was to have the potentiometer control the servo movement as well as which color was showing on the RGB LED. 

Here's a picture of the wiring:
</br>
<img width="600" alt="image" src="https://github.com/user-attachments/assets/861543bc-5d4c-44f3-aedb-2b41e04a4c96" />

Here's the circuit diagram:
</br>
<img width="800" src="https://github.com/user-attachments/assets/952c9556-9abe-4e99-8028-9b283a66a58f" />


Here's the first iteration ([video](https://drive.google.com/file/d/1NB7KVbpYn_PooYlIUoySLBr6USCV2Wq3/view?usp=sharing)) and the code associated:

```

#include <Servo.h>

Servo myservo;
int potpin = A0;
int sensorValue = 0; // value read from the potentiometer
int outputValue = 0; // adjusted for LED
int val;

const int redLEDPin = 12;
const int greenLEDPin = 6;
const int blueLEDPin = 4; 

void setup() {
  myservo.attach(9);
  pinMode(redLEDPin, OUTPUT);
  pinMode(greenLEDPin, OUTPUT);
  pinMode(blueLEDPin, OUTPUT);
  Serial.begin(9600);
}

void loop() {
  sensorValue = analogRead(potpin); // read the analog in value from the portentiometer
  outputValue = map(sensorValue, 0, 1023, 0, 255);   // map it to the range of the analog out:

  //for debugging
  Serial.print("sensor = ");
  Serial.print(sensorValue);
  Serial.print("\t output = ");
  Serial.println(outputValue);

  if (sensorValue <= 333) { 
    digitalWrite(redLEDPin, HIGH); // if in the first third, light red
    digitalWrite(greenLEDPin, LOW);
    digitalWrite(blueLEDPin, LOW);
  } else if (sensorValue <= 666) {
    analogWrite(greenLEDPin, outputValue); // if in the middle third, light green
    digitalWrite(redLEDPin, LOW);
    digitalWrite(blueLEDPin, LOW);
  } else {
    analogWrite(blueLEDPin, outputValue); // if in the last third, light blue
    digitalWrite(greenLEDPin, LOW);
    digitalWrite(redLEDPin, LOW);
  }
  val = map(sensorValue, 0, 1023, 0, 180);
  myservo.write(val);
  delay(15);
}
```

You might notice that the code is slightly different for the red LED compared to the other two. I tried to expand on the "AnalogInOutSerial" code to light the RGB LED by using the same value mapping, but for some reason it wasn't working for the first third of values. Printing out the values showed that the LED should have lit red and there didn't seem to be a wiring issue so I used digitalWrite instead of using the analog sensor value. (I really want to know why this doesn't work!)

To further investigate the issue, I commented out everything except for an analogWrite line for each LED color and I experimented with different values. For some reason, the green would light up for any value but the red and blue would only light up at very high values (200+).

However, I wanted there to be more of a gradual transition between the colors and updated the code accordingly which required using analog sensor values for all LEDs. I had some help from Isabella Fiorante who was working on [something similar](https://github.com/ifiorante/TDF-FA25/blob/main/week-4/potentiometer_trial). It still didn't work, so I tried this but to no avail:

```
#include <Servo.h>

Servo myservo;
int potpin = A0; // the input pin for the potentiometer
int sensorValue = 0; // value read from the potentiometer
int outputValue = 0; // adjusted for LED
int val; // sensor value adjusted for servo

const int redLEDPin = 12;
const int greenLEDPin = 6;
const int blueLEDPin = 4; 

void setup() {
  myservo.attach(9); // initializing servo
  pinMode(redLEDPin, OUTPUT);
  pinMode(greenLEDPin, OUTPUT);
  pinMode(blueLEDPin, OUTPUT);
  Serial.begin(9600); // initializing serial communication at 9600bd
}

void loop() {
  sensorValue = analogRead(potpin); // read the analog in value from the portentiometer

  // for debugging:
  //Serial.print("sensor = "); 
  //Serial.print(sensorValue);
  Serial.print("\t output = ");
  Serial.println(outputValue);

  //LEDs
  if (sensorValue <= 340) {  // for right third of values on the potentiometer (red)
    digitalWrite(greenLEDPin, LOW);
    digitalWrite(blueLEDPin, LOW);
    outputValue = map(sensorValue, 0, 1023, 0, 255);   // map third value to LED scale
    analogWrite(redLEDPin, outputValue);
  } else if (sensorValue <= 683) { // for middle third of values on the potentiometer (green)
    digitalWrite(redLEDPin, LOW);
    digitalWrite(blueLEDPin, LOW);
    outputValue = map(sensorValue - 340, 0, 1023, 0, 255);   // map third value to LED scale
    analogWrite(greenLEDPin, outputValue);
  } else { // for left third of values on the potentiometer (blue)
    digitalWrite(greenLEDPin, LOW);
    digitalWrite(redLEDPin, LOW);
    outputValue = map(sensorValue - 684, 0, 1023, 0, 255);   // map third value to LED scale
    analogWrite(blueLEDPin, outputValue);
  }

  //Servo
  val = map(sensorValue, 0, 1023, 0, 180); // map sensor value to servo values
  myservo.write(val);
  delay(15); // servo lag time
}

```
</br>
After some guidance from Sudhu, I spread out my circuit to debug the wiring more easily and used some test code that he sent to debug my code: 

```
/*
  Analog Input

  Read an analog sensor on analog pin 0 and
  map the sensor value to a range of colors
  displayed on an RGB LED

  The circuit:
  - potentiometer
    center pin of the potentiometer to the analog input 0
    one side pin (either one) to ground
    the other side pin to +5V
  - RGB LED
    cathode (longest leg) attached to ground
    red - pin 9
    green - pin 10
    blue - pin 11
 
    modified from http://www.arduino.cc/en/Tutorial/AnalogInput
*/

const int sensorPin = A0; // set the input pin for the potentiometer
int sensorValue = 0;      // variable to store the value coming from the sensor 
int val = 0;              // remap value
const int redLED = 12;       // set the pin for the red LED
const int greenLED = 6;    // set the pin for the green LED
const int blueLED = 4;     // set the pin for the blue LED
int redBrightness = 0;      //variable to store the brightness of the red LED
int greenBrightness = 0;      //variable to store the brightness of the green LED
int blueBrightness = 0;      //variable to store the brightness of the blue LED

void setup() {
  pinMode(redLED, OUTPUT);    // declare the ledPin as an OUTPUT:
  pinMode(greenLED, OUTPUT);  // declare the ledPin as an OUTPUT:
  pinMode(blueLED, OUTPUT);   // declare the ledPin as an OUTPUT:
  Serial.begin(9600);  // initialize serial communication
  //TestEm();
}

void loop() {
  // read the value from the sensor:
  sensorValue = analogRead(sensorPin);
  
  // print out the value you read:
  Serial.print("sensorValue: "); Serial.print(sensorValue);

  // map sensor value to RGB brightness
  val = sensorValue;
  
  if(val >= 0 && val <= 341){
    // RED 
    redBrightness = map(val, 0, 341, 255, 0); 
    greenBrightness = map(val, 0, 341, 0, 255);  
    blueBrightness = 0;        // extra security
  } else if(val > 341 && val <= 682){
    // GREEN
    redBrightness = 0;  // extra security
    greenBrightness = map(val, 342, 682, 255, 0);
    blueBrightness = map(val, 342, 682, 0, 255);  
  } else if (val > 682 && val <= 1023){ 
    //BLUE
    blueBrightness = map(val, 683, 1023, 255, 0);
    greenBrightness = 0;  // extra security
    redBrightness = map(val, 683, 1023, 0, 255);
  } else {
    Serial.println("val remap: OUT OF RANGE");
  }
  // write to RGB LED
  analogWrite(redLED, redBrightness);
  analogWrite(greenLED, greenBrightness);
  analogWrite(blueLED, blueBrightness);

  // print out RGB brightness values
  Serial.print("\t RGB: "); Serial.print(redBrightness);
  Serial.print(", "); Serial.print(greenBrightness);
  Serial.print(", "); Serial.println(blueBrightness);
  delay(1000);
}

// ---------------------------------
void TestEm() {
  //Test the LED.
  Serial.println("Starting test");
  analogWrite(redLED, 255);
  Serial.println("Red");
  delay(3000);
  analogWrite(redLED, 0);
  delay(3000);
  analogWrite(greenLED, 255);
  Serial.println("green");
  delay(3000);
  analogWrite(greenLED, 0);
  delay(3000);
  analogWrite(blueLED, 255);
  Serial.println("blue");
  delay(3000);
  analogWrite(blueLED, 0);
  delay(3000);
}
// ---------------------------------
```

I realized that I was using the map function incorrectly in my previous code iterations and that the resistors are too strong, preventing the dimming effect from being as perceivable. So, I updated my circuit to optimize relative brightness of the various colors and switched to the cycling of colors (i.e. instead of three colors changing brightness, the colors blend together in the transitions) with simultaneous servo movement, all controlled by the potentiometer position. I also realized that the potentiometer values aren't very accurate and fluctuate as it moves, causing color flickers and the servo to reverse directions suddenly sometimes. 

Here's the final circuit:

Here's the final wiring:

Here's the final code:

You can see it working here. While this week was challenging, I deepened my understanding of electronics and Arduino, and still had a lot of fun in the process. 
</br>

## 3D Printed Rings

This week's fabrication assignment was:
> Recreate your laser cut ring for 3DP. For a second ring, design it for 3DP.

I returned to my Illustrator from last week and removed the living hinges since I didn't think would transfer well to 3D printing, resulting in the following image:
</br>
<img width="400" alt="Screenshot 2025-09-16 at 9 24 46 PM" src="https://github.com/user-attachments/assets/627b67f2-15cb-45c7-8632-8c7048bcd524" />

I imported the file into Fusion and to replicate the contrast and depth, I extruded the petal and leaf patterns to a higher point than the rest of the ring:
</br>
<img width="400" src="https://github.com/user-attachments/assets/16ef5afd-c447-4840-9cce-98ccf8eed322" />

Then, onto the next ring. My initial inclination was to try to transfer a classic pattern from Chinese enamel dishware (common in Peranakan households in Singapore, like my mom's) onto the surface of the ring. Here's an example from "The Peranakan House" in Katong:
</br>
<img width="600" alt="image" src="https://github.com/user-attachments/assets/bbccfcdd-6742-42a2-9d6e-eefe77608150" />

I struggled for a while trying to replicate the pattern in any legible way in Illustrator before changing my mind. I then thought I would try to use an existing line drawing I had of a basset hound as a centerpiece for a ring: 
</br>
<img width="400" alt="image" src="https://github.com/user-attachments/assets/6be7dd61-a0c5-4ca5-9374-aee5b0f3b238" />
<img width="400" alt="Screenshot 2025-09-18 at 11 37 30 PM" src="https://github.com/user-attachments/assets/6e0f4b91-6987-4d42-a670-ab03e67dbcac" />

Fusion had all kinds of errors on the transfer of this (after processing in Illustrator) that I couldn't seem to resolve. Since the transfer between softwares seemed to be the source of my problems, I decided to just model the ring from scratch within Fusion and see what kinds of things I could make. I ended up with two different ring designs, one with a lego hand sticking up and the other with BB8 from Star Wars:
</br>
<img width="400" alt="Screenshot 2025-09-18 at 11 38 37 PM" src="https://github.com/user-attachments/assets/7d8e470e-b956-4c46-91ea-c25c0c6ee37d" />
<img width="300" alt="Screenshot 2025-09-17 at 8 28 08 AM" src="https://github.com/user-attachments/assets/d599ac03-0688-4020-bbbd-c3afa918e5fd" />

</br>

_Note: From my minimal previous 3D printing experience, I knew I should flatten part of the ring so it could stand on the bed which is why both of the above designs have a plane and/or a slice that I may have cut too deep into the ring._

I ultimately decided the lego hand was too simple (which I regret), and decided to print the BB8 with the flower knuckles. I printed on the Prusa MK4 and generated organic supports (ignore the settings shown in the slicer, they are not the ones I used in the end):
</br>
<img width="800" alt="image" src="https://github.com/user-attachments/assets/a32c24c7-7847-417a-ae9d-ce105e5e5c62" />
</br>

While the first print ultimately failed due to 3D printer issues (z-axis was too high? filament clogging? -- some things other people said when they saw it), I learned that I had not adjusted the size when importing my design from Illustrator to Fusion causing the knuckles (which was mostly complete despite the issues) not to fit on my hand:
</br>
<img width="400" src="https://github.com/user-attachments/assets/082dd2d0-526c-4bfd-834b-c769b1116c78" />
<img width="400" alt="image" src="https://github.com/user-attachments/assets/2cc0fede-9641-43cf-9a6b-6da5b48d9bea" />

So, I tried to use a different printer! I forgot to take pictures of the second version of the knuckle ring (which still ended up a little small), but here is BB8 fresh off the printer:
</br>
<img width="400" alt="image" src="https://github.com/user-attachments/assets/6612f027-5f2c-4acf-94be-4f1ce43f65dc" />
<img width="400" alt="image" src="https://github.com/user-attachments/assets/f93b5e74-16ee-4d29-af1f-8bc8a84fa288" />
</br>

I had some paint on hand so I painted the rings to make it more obvious what the figures were (in the future I should just get better at modeling). [Here's](https://drive.google.com/file/d/19EHnM84CXPORIfCHtB_7XhKR2ghoL0n-/view?usp=sharing) how they turned out:
</br>
<img width="400" alt="image" src="https://github.com/user-attachments/assets/152ede74-31ef-4b6a-a316-f473d20cc2b2" />
<img width="400" alt="image" src="https://github.com/user-attachments/assets/480911db-35f2-4cef-ba16-2766eb8890fd" />
</br>
In retrospect, I should have tried to go for my first idea, or at least some version of a pattern wrapped around the band of the ring. I think that would have been a more interesting and challenging use of 3D printing.

Thanks for reading! Onto the first project! :)
