I had a lot of issues getting the same code for the button to work, but the potentiometer worked fine. I also managed to get the servo! So, to experiment I decided I would try to combine the RGB LED, potentiometer and the servo. The goal was to have the potentiometer control the servo movement as well as which color was showing on the RGB LED. 

Here's the first iteration (video) and the code associated:

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

I was having difficulties with the red LED turning on in the first third. Most of the range of the potentiometer in the first third seemed to just leave all of the LEDs off despite it still reading out values that should have lit the LED (dimly but still). I got around it by just turning it on and off since I couldn't figure out the issue. However, I wanted there to be more of a transition between the colors and updated the code accordingly. You can find the video of the second iteration here.

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
  Serial.print("sensor = ");
  Serial.print(sensorValue);
  Serial.print("\t output = ");
  Serial.println(outputValue);
  if (sensorValue <= 333) {
    if (sensorValue <= 100) {
      analogWrite(redLEDPin, outputValue + 100);
    } else {
      analogWrite (redLEDPin, outputValue + 200);
    }
    digitalWrite(greenLEDPin, LOW);
    digitalWrite(blueLEDPin, LOW);
    if (sensorValue > 250) {
      analogWrite(greenLEDPin, abs(250 - sensorValue));
    }
  } else if (sensorValue <= 666) {
    if (sensorValue <= 450) {
      analogWrite(greenLEDPin, outputValue - 100);
    } else {
      analogWrite(greenLEDPin, outputValue); // if in the middle third, light green
    }
    digitalWrite(redLEDPin, LOW);
    digitalWrite(blueLEDPin, LOW);
    if (sensorValue > 580) {
      analogWrite(blueLEDPin, abs(580 - sensorValue));
    }
  } else {
    analogWrite (blueLEDPin, outputValue);
    digitalWrite(greenLEDPin, LOW);
    digitalWrite(redLEDPin, LOW);
  }
  val = map(sensorValue, 0, 1023, 0, 180);
  myservo.write(val);
  delay(15);
}
```
