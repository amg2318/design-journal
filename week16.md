# Final Project (continued)

This week we finished our final project and presented it at the Jacobs Design Showcase!

## Fabrication

We received our resin-printed buttons and dials from Chris in a very cute drop-off:
</br>
</br>
<img width="400" src="https://github.com/user-attachments/assets/792c3e84-8ce8-46bb-a142-d7af2ce0a877" />
</br>
</br>
After copious sanding by Jill and Alexis (I was doing code and music stuff), some painting, and the internal support print finishing, we began final assembly:
</br>
</br>
<img width="300" src="https://github.com/user-attachments/assets/121a8bf0-8bee-4761-b191-79f566cd7bf9" />
<img width="300" src="https://github.com/user-attachments/assets/8ce261b7-1911-4ced-ba8b-71f0bd057a36" />
<img width="300" src="https://github.com/user-attachments/assets/e4c6e4b7-601f-422a-b261-bce686e7cc4e" />
<img width="500" src="https://github.com/user-attachments/assets/49d6867e-ec56-4007-a554-26344f9c2fb8" />
</br>
</br>
Here's what it ended up looking like (kudos to Luis for taking these pics):
</br>
</br>
<img width="400" src="https://github.com/user-attachments/assets/f03fd698-61d6-408e-8a07-465246044e0e" />
<img width="400" src="https://github.com/user-attachments/assets/2aead8d8-2415-4694-8a88-25c144734783" />
</br>

## Electronics

While the form was being completed by Jill and Alexis, I recorded some sound files based on specific runs of my old code (with the random sound from a set) that sounded like how we would expect the object to sound. You can find the audio files [here](https://drive.google.com/drive/folders/1ebgZpdzNxOG9uc5TXHSNXRBB7gr9xMpc?usp=drive_link). Then, I replaced each of the sets of sound files with one of the longer recordings. You can see us testing and listening together while deciding on sounds:
</br>
</br>
<img width="300" src="https://github.com/user-attachments/assets/6a625b26-77f2-4b1f-b67c-cd721654bf4b" />
<img width="300" src="https://github.com/user-attachments/assets/34b887be-e1ad-467a-9460-357defd57c1e" />
</br>
</br>
Once we settled on sounds we liked and I added in a cross-fade between sounds to replace simultaneous play, I re-wired the electronics and wrote a simple Arduino program to send over the button and potentiometer values over serial:
</br>
```
// ---------------------------------------------------------
// Arduino: Always send exactly 8 values
// 6 potentiometers + 2 buttons → CSV
// Format: v0,v1,v2,v3,v4,v5,b0,b1
// ---------------------------------------------------------

const int potPins[6] = {A0, A1, A2, A3, A4, A5};
const int buttonPins[2] = {13, 14};

void setup() {
  delay(1000);
  Serial.begin(115200);

  // Buttons use internal pullups → pressed = LOW
  for (int i = 0; i < 2; i++) {
    pinMode(buttonPins[i], INPUT_PULLUP);
  }
}

void loop() {
  int pots[6];
  int buttons[2];

  // Read potentiometers
  for (int i = 0; i < 6; i++) {
    pots[i] = analogRead(potPins[i]);
  }

  // Read buttons (invert so pressed = 1)
  buttons[0] = digitalRead(buttonPins[0]);
  buttons[1] = digitalRead(buttonPins[1]);

  // --------- Guaranteed 8 values, in order ---------
  Serial.print(pots[0]); Serial.print(',');
  Serial.print(pots[1]); Serial.print(',');
  Serial.print(pots[2]); Serial.print(',');
  Serial.print(pots[3]); Serial.print(',');
  Serial.print(pots[4]); Serial.print(',');
  Serial.print(pots[5]); Serial.print(',');
  Serial.print(buttons[0]); Serial.print(',');
  Serial.println(buttons[1]);     // newline always at end
  // -------------------------------------------------

  delay(10); 
}
```
</br>
Then, I updated the p5 code to parse the serial data and replaced the p5 sliders with the potentiometer values. [Here's a video](https://drive.google.com/file/d/1fsV6m50Cm3JV2s89DEkIFeYGIUtg3g_0/view?usp=sharing) of it working before showcase.

## Jacobs Showcase + Reflection

Here are a couple of pictures from the Jacobs Showcase of our project. Sabrina Merlo also took [this video] of her trying out our project. 
</br>
</br>
<img width="300" src="https://github.com/user-attachments/assets/7824b7ed-02fc-4228-9481-84762af28e3f" />
</br>
</br>
Overall, I'm really happy with our final product. It looks really good and the functionality is very clear and discernable, even though we had to cut a lot of aspects of the project I was excited about (e.g. multiple detection, Neopixels, layering sound). Working with Alexis and Jill was wonderful. Getting feedback from the critics was exciting, especially since many of them brought up features we had already attempted or considered. We got some pointers for how to explore our original idea of having different compartments that manipulated the sound, as well as suggestions for different ways to approach multiple detection. With more time, I would have liked to make more sound files and attempt layering the sounds again since I do really believe it's possible. In the future, I might also look into sound generation based on other critic suggestions as well. 

This is my last entry, so thank you very much for reading! I know this project was less visual than the last few due to the code-heavy nature but I hope it still communicated my team's hard work and many iterations. :)






