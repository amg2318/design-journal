# Final Project (continued)

This week we continued to work our concept for the final project, adding manipulation of sound effects in our software and beginning the buttons panel. 

## Electronics

Picking up from the overlaying sounds, I began to experiment with various sound effects. At first I tried to add an distortion, noise, monosynth, or oscillator as many of the videos I watched recommended but it sounded really jarring with the calming ambient sounds. These sound additions were very harsh and electronic, adding beeping or droning sounds very similar to a fire alarm. I didn't record it because it was that bad. 
</br>
</br>
<img width="600" alt="Screenshot 2025-12-12 at 5 03 00 PM" src="https://github.com/user-attachments/assets/e7f72dc6-d757-476c-9c77-7352bd5c9c08" />

After struggling with that, the first slider I added was frequency of a low pass filter which was subtle but nice: [video](https://drive.google.com/file/d/1cj6Ad7CcosU1ckVtC_fjQgA_BJOCRU4l/view?usp=sharing)

Then I experimented with resonance on that filter controlled by the slider: [video](https://drive.google.com/file/d/1G9QAb0mHKD9IobV1F5R4sSPL3xZpIrch/view?usp=sharing) 

I also experimented with delay (feedback, time, ratio/echo), compressor, reverb, gain, drywet, and other filter types but they all ended up being fairly subtle. Alexis [tried a version](https://editor.p5js.org/ajia-darkened/sketches/OyiolOv0Y) using some of our old sound files that was slightly less subtle. Based on that, I made [this version](https://editor.p5js.org/ajia-darkened/sketches/LcRqDERyo) with resonance and frequence of the low pass filter, delay feedback, drywet of the filter, compressor threshold and playback speed of the current sound. However, it sounded crazy (no recording to spare your ears) because of tons of overlapping sounds.

I spent a long time trying to troubleshoot this and determined that the brevity of each sound file and the overlapping made it hard to distinguish what effect was happening. I decided that the next steps would be to find or record a longer sound file of similar sound profiles and remove the sound layering (sadly) but keep the same effects. 

## Fabrication

We received our 3D printed parts from Chris and Alexis lasercut the acrylic for the object space and control panel plate.
</br>
</br>
<img width="400" alt="image" src="https://github.com/user-attachments/assets/0ea8f181-b1c1-483f-b908-f3e923ffabda" />
<img width="400" alt="image" src="https://github.com/user-attachments/assets/b45ad58b-8730-4fdd-8ba0-7441ed341c18" />
</br>
</br>
We had initially planned to remove the brown covering on the object plate clear acrylic and frost it with an orbital sander, but it looked so good as is that we kept it that way. Unfortunately, the white control panel plate was just slightly too small (not very visible but it rattles when the whole form is moved). We also agreed that we didn't like how reflective it was so Jill made the plate in Fusion for 3D printing along with a first version of buttons and dials:
</br>
</br>
<img width="400" alt="image" src="https://github.com/user-attachments/assets/add0e76c-bdc2-4243-84b6-0e1fccac71bd" />
<img width="400" alt="image" src="https://github.com/user-attachments/assets/c469ad7f-5c4a-4095-8453-c83b432b14a5" />
</br>
</br>
Then we fit in the parts (without the breadboard) to test clearances and the interaction:
</br>
</br>
<img width="500" alt="image" src="https://github.com/user-attachments/assets/6816fab2-5693-414f-bd83-a6d9f2c2abe1" />
<img width="280" alt="image" src="https://github.com/user-attachments/assets/b072d86e-429b-4c8d-af2f-cdb0428375ce" />
</br>
</br>
The sliders jut out too far from the top of the plate, the dials were difficult to attach to the potentiometers, and the buttons knocked against each other so we opted for another iteration. 
</br>
</br>
<img width="400" alt="image" src="https://github.com/user-attachments/assets/e19336ac-6296-443b-aed3-8bd090a2b4aa" />
<img width="400" alt="image" src="https://github.com/user-attachments/assets/e8b27e17-4593-472e-88e7-c8704a766dc3" />
</br>
</br>
The electronics were dangling on the inside, making it hard to press the buttons well so we took measurements for an internal support. Inspired by Luis Somasundaram and Edna Ho, we decided for aesthetics we would have the buttons and dials resin printed for a nicer texture and interaction instead of hard PLA. We sent our buttons and dials to get resin printed by Chris, and planned to finish troubleshooting the audio and do final assembly next week. 
</br>
</br>
<img width="300" alt="image" src="https://github.com/user-attachments/assets/5a5b0730-dc73-44e2-bc5f-9bba838b17f8" />



