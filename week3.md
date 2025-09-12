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
This week's fabrication prompt is:
> Create a ring that describes a part of your personality or a cause you believe in.
> Make it bold. Make it big. Have fun with it!

For my initial brainstorm, I focused on what kinds of mechanisms or technical qualities would be cool in the final product before thinking about the message I wanted to convey. To translate my messy handwriting in the picture below, here are some of the ideas I had:
- a ring as a slot with interchangeable parts (the top sketch)
- a looking glass mounted on top of a ring with changeable lenses of different materials (ruled out because I didn't actually have many materials on hand or from the shop)
- a ring as a holder for something else, e.g. a pin cushion for sewing, a vice (didn't know how to execute this), Apple stylus
- a puzzle mounted on a ring
- a ring with attachments that move with living hinges, e.g. plants swaying in the wind (bottom sketch)
 
<img width="400" alt="image" src="https://github.com/user-attachments/assets/3ceb8b00-2532-4e59-abd1-53a1bf8c3a73" />

When considering how these different ideas could work with parts of my identity or causes I care about, I settled on the last one. In the last few years, gardening has become a very healing activity for me. I re-landscaped my parents' house and plan on continuing to when I can. The process has completely revolutionized the way I see "gardens" and environmentalism. _Plus, it also felt like a callback to my old Doodle4Google entries in which middle school me attempted to use my finalist platform to campaign for environmentalism for several years._

Another inspiration was this meme/drawing that keeps showing up in my feed (I _think_ the original artist is Favorite Vegetable on X):
<img width="600" alt="image" src="https://github.com/user-attachments/assets/f72a6cdb-c438-46b5-a217-87ea7a68de0a" />

After some consultation with my partner (former DES INV 22 student and professional mechanical engineer), I learned that living hinges would likely break at this small of a scale. So I improved on my design by expanding to knuckles and increasing the stem size of each of the plants in the ring plantbed. 

<img width="600" alt="image" src="https://github.com/user-attachments/assets/26f8d738-230d-4e4c-a719-c8ca50273558" />

<br/>
<br/>
Then, I uploaded a photo of the sketch to Procreate to refine it.

<img width="600" alt="image" src="https://github.com/user-attachments/assets/7675cede-9ece-44bd-aeba-28d986152140" />
<img width="400" alt="image" src="https://github.com/user-attachments/assets/f5b56933-69de-4113-a7cb-6a28ba5a1976" />

<br/>
<br/>
In Illustrator, I tried to replicate the general shape of commercially made knuckles with my approximate ring size (a little over-estimated, just in case) and combined that design with the garden sketch to create the laser cutting file. _Note: the image on the right has the stroke sized up for visibility._

<br/>
<br/>
<img width="400" alt="Screenshot 2025-09-11 at 6 12 08 PM" src="https://github.com/user-attachments/assets/03c5883d-6ce6-495e-8850-156c8b790633" />
<img width="400" alt="Screenshot 2025-09-11 at 6 17 17 PM" src="https://github.com/user-attachments/assets/4fb0e252-e0a4-46e3-8b44-76e1206e9fc2" />

<br/>
<br/>
Next: (lasercutting!) [https://drive.google.com/file/d/1DB5aYrc7ZnoLjDyuRRvOmwxBLnHVdW7M/view?usp=sharing] <br/>

<img width="400" alt="image" src="https://github.com/user-attachments/assets/37d829f6-eed0-4951-9734-18e826f6b4e7" />

<img width="400" alt="image" src="https://github.com/user-attachments/assets/48d98b63-3ff7-4740-b3bf-133c260095c7" />

<img width="400" alt="image" src="https://github.com/user-attachments/assets/aabd3f54-1830-479e-ad6a-08d6ecb77f83" />

<img width="400" alt="image" src="https://github.com/user-attachments/assets/40537635-e5f6-48f2-8776-802a31fe01f9" />

<img width="400" alt="image" src="https://github.com/user-attachments/assets/787832d6-2b11-40a0-a3a4-320a5bb95534" />

https://drive.google.com/file/d/1R1mZiOCLUg67ZzbwT8xOVSuAKnNkELee/view?usp=sharing


