# Final Project

For our final project, we were assigned to "design an object, device, system, structure, etc. that helps to create a version of the world you want to live in." I worked with [Alexis Chew] and [Jill Silva](https://www.figma.com/board/ObBzUwYM1VchBA8PILtv31/TechDesignFoundations_Journal_SILVA?node-id=0-1&t=2Xzcw9u5HeHFQ3bn-1).

This first week we focused on solidifying the concept of our project. We had all bonded over shared interest in Alexis's project proposal: a music making toy that relies on found natural objects to define the sound. However, that concept was fairly vague in terms of form and how it would be built. The goal was to inspire children to both use technology and immerse themselves in nature.

## Electronics

For this whole project, I took the lead on coding and electronics. To figure out the technical logistics of this project, I did some research on the best methods to achieve the following functionality:
* How to use computer vision to detect natural objects (is p5.js enough? is there an existing model we can draw from?) 
* How the sound would be produced, digitally (what library, coding language, etc.) and physically (how to incorporate a speaker)
* Sound production vs. sound synthesis/manipulation (I knew nothing about music and I'm tone deaf)
* How to include and code for a camera within the form of our project rather than built-in to the computer (i.e. what kind of camera should we get)

With some advice from professors Jeff Lubow and Sudhu Tewari, we constructed this system architecture diagram:
</br>
</br>
<img width="800" alt="image" src="https://github.com/user-attachments/assets/f654fe82-7ac3-4a8a-b26b-346115754c5f" />
</br>
</br>
To summarize, we planned to train our own computer vision model with Teachable Machine to detect a single natural object at a time using a USB-connected webcam (rather than attaching to a microcontroller), run in p5.js. The detected object would determine a sound file manipulated through Mozzi API and physical dials or sliders over a Serial connection. If we had time, we wanted to make slightly different sounds based on visual qualities of the object like color and type (e.g. a yellow ginko leaf and a red maple leaf would both play kinds of leaf-associated sounds). For the speaker, Sudhu gave us the following custom PCB board he made for us to experiment, and othewise suggested we hide a bluetooth speaker inside the form:
</br>
</br>
<img width="400" alt="image" src="https://github.com/user-attachments/assets/87a945a4-79e8-4933-aa13-fe895008bb64" />
<img width="400" alt="image" src="https://github.com/user-attachments/assets/e040fd65-d45d-48a4-be65-cca4f4083044" />
</br>
</br>
We then began ordering and purchasing our speaker and camera:
</br>
</br>
<img width="300" alt="image" src="https://github.com/user-attachments/assets/59c3d77d-a17c-4445-bc43-2f0c08b99971" />
<img width="500" alt="image" src="https://github.com/user-attachments/assets/6bcc8578-c791-4a5c-bae9-77c218c74b29" />
</br>

## Fabrication

For the form, we all brainstormed together by creating a moodboard and making a lot of sketches. Since our goal was to make a children's toy, we drew a lot of inspiration from Fisher Price and existing tech toys. We also took some inspiration from DJ decks and other music making devices. We also needed an arm for the camera look at the designated "object space", so studied the visual design of lamps too.
</br>
</br>
<img width="600" alt="image" src="https://github.com/user-attachments/assets/513fe15b-1173-4287-a0bb-e769a57293b6" />

<img width="400" alt="image" src="https://github.com/user-attachments/assets/e21e82fc-4b18-48aa-a3b9-641efc17ec10" />
<img width="400" alt="image" src="https://github.com/user-attachments/assets/d8b5cd35-91b6-4460-bf98-471e987d0654" />
<img width="400" alt="image" src="https://github.com/user-attachments/assets/a68cd8aa-2975-4986-b336-5b1ee4987627" />
<img width="400" alt="image" src="https://github.com/user-attachments/assets/56b2f5d2-ac8b-48ae-853d-bd04a2e68d42" />
</br>
</br>
Our main debate was over having a scanner or a more stationary object, but we ended up settling for the latter to ease the burden on figuring out electronics (since the camera and the microcontroller needed to be plugged into the computer anyway). We also considered including a segmented "object space" inspired by tchotchke shelves such that placement of the natural objects would manipulate the sound. Unfortunately, upon consulting the professors and doing some research, we found that would be difficult to implement within the scope and timeframe for this project. We settled on a smooth scanning surface and opted to at least explore multiple-object detection without specific placement. For this iteration of sketches, we drew on lamp design for the camera arm and designing for a clear "object space".
</br>
</br>
<img width="600" alt="image" src="https://github.com/user-attachments/assets/c2cb0611-96ac-481f-8a8f-8100259be747" />
</br>
</br>
We planned to explore sound synthesis, computer vision, and visual representations of the object detection with Neopixels (in case sound didn't work out, otherwise a nice-to-have to complement the music) in the next week. 

