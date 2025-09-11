## LED Experiment
This week's electronics prompt is:
> Do "something interesting" with 1+ LEDS and 1 LDR. 

Upon finishing the overview tutorial, I first experimented with including a second LED that lit at lower sensor readings and the original at higher sensor readings, each at a brightness reflective of the strength of light read by the LDR.

Code:
```
void setup() {
  pinMode(11, OUTPUT);
  pinMode(9, OUTPUT);
  Serial.begin(9600);
}

void loop() {
  int sensorValue = analogRead(A0);
  Serial.print("Analog IN A0 value = ");
  Serial.println(sensorValue);
  if (sensorValue >= 50) {
    analogWrite(9, sensorValue);
    digitalWrite(11, LOW);
  } else {
    analogWrite(11, sensorValue/10);
    digitalWrite(9, LOW);
  }
  delay(100);
}
```
[Here's](https://drive.google.com/file/d/1s06oIskS0ISfKvkFh1-rUU41RbVzYNmA/view?usp=sharing) a video of what that looks like.
 
But that didn't seem interesting enough. I added another LDR and LED, and decided to take the rate of change to determine the fade rate.
<img width="400" alt="image" src="https://github.com/user-attachments/assets/78a734a5-e6b8-474b-80a6-e1af9828a052" />

(insert circuit diagram when I find a good way to do that for free)

For the [first iteration](https://drive.google.com/file/d/17X2XXrZP_aC7mB7s_i-xOIYH8K4gZv0N/view?usp=sharing), the red LED fade rate was based on immediate rate of change of the first LDR sensor, the green LED was simply turned on/off by covering the second LDR sensor, and the blue LED would blink unless the first LDR sensor was covered. (A challenge of this was tuning the values so the differences in light were apparent and discovering only certain channels can have PWM output!)

I decided I could make things even more interesting by having the green LED light up based on the rate of change of the second LDR readings over time. [Here's](https://drive.google.com/file/d/1_2PDKTkrjyIjYCkDmV7RYwbK92ZMWVkY/view?usp=sharing) what the final iteration looks like. 

Code:
```
int brightness = 0;
int brightness2 = 0;
int fadeAmount = 10;
int prevValue = 0;
int vals[10];
int counter = 0;
int avgRate = 0;

void setup() {
  pinMode(11, OUTPUT); //blue LED
  pinMode(9, OUTPUT); //green LED
  pinMode(6, OUTPUT); //red LED
  Serial.begin(9600);
}

void loop() {
  int newSensorValue = analogRead(A0);
  vals[counter] = analogRead(A1);
  fadeAmount = prevValue/newSensorValue; //finding the rate of change to control the red LED
  //Serial.println(fadeAmount); //for testing
  analogWrite(6, brightness);
  brightness = brightness + fadeAmount*10; 
  if (brightness <= 0 || brightness >= 255) {
    fadeAmount = -fadeAmount;
  }
  // green LED uses the rate of change of the second sensor readings to change its fade rate
  int sum = 0;
  for (int i=0; i<10; i++){
    sum += vals[i];
  }
  analogWrite(9, brightness2);
  avgRate = sum/10;
  brightness2 = brightness2 + avgRate; 
  if (brightness2 <= 0 || brightness2 >= 255) {
    avgRate = -avgRate;
  }
  //blue LED should turn on and off rapidly while the sensor is not pressed
  //Serial.println(newSensorValue); //for testing
  if (newSensorValue >= 50){ //set to 50 when lights are on in the apartment, 10 during the day with windows shut for covered sensor reading
    for (int i=0; i<=20; i++){
      digitalWrite(11, HIGH);
      digitalWrite(11, LOW);
    }
  } else {
    digitalWrite(11, LOW);
  }
  // reset
  delay(100);
  digitalWrite(11, LOW);
  prevValue = newSensorValue;
  if (counter == 9){
    counter = 0;
  } else {
    counter++;
  }
}
```

## Making a ring


<img width="400" alt="image" src="https://github.com/user-attachments/assets/3ceb8b00-2532-4e59-abd1-53a1bf8c3a73" />

<img width="600" alt="image" src="https://github.com/user-attachments/assets/26f8d738-230d-4e4c-a719-c8ca50273558" />

https://drive.google.com/file/d/1DB5aYrc7ZnoLjDyuRRvOmwxBLnHVdW7M/view?usp=sharing

<img width="400" alt="image" src="https://github.com/user-attachments/assets/37d829f6-eed0-4951-9734-18e826f6b4e7" />

<img width="400" alt="image" src="https://github.com/user-attachments/assets/48d98b63-3ff7-4740-b3bf-133c260095c7" />

<img width="400" alt="image" src="https://github.com/user-attachments/assets/aabd3f54-1830-479e-ad6a-08d6ecb77f83" />

<img width="400" alt="image" src="https://github.com/user-attachments/assets/40537635-e5f6-48f2-8776-802a31fe01f9" />

<img width="400" alt="image" src="https://github.com/user-attachments/assets/787832d6-2b11-40a0-a3a4-320a5bb95534" />

https://drive.google.com/file/d/1R1mZiOCLUg67ZzbwT8xOVSuAKnNkELee/view?usp=sharing


