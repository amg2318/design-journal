## Potentiometer Experiment

For this week's electronics experiment, I decided I would try to combine the RGB LED, potentiometer and the servo. The goal was to have the potentiometer control the servo movement as well as which color was showing on the RGB LED. 

Here's a picture of the wiring:
</br>
<img width="600" alt="image" src="https://github.com/user-attachments/assets/861543bc-5d4c-44f3-aedb-2b41e04a4c96" />

(insert circuit diagram)

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

However, I wanted there to be more of a gradual transition between the colors and updated the code accordingly which required using analog sensor values for all LEDs. I had some help from Isabella Fiorante who was working on [something similar](https://github.com/ifiorante/TDF-FA25/blob/main/week-4/potentiometer_trial). You can find the video of the second iteration here.

```

```

</br>
## 3D Printed Rings

This week's fabrication assignment was:
> 
