# Final Project (continued)

This week we continued to work our concept for the final project, learning music making and how to use the p5 sound library, and how to overlap multiple random sounds together. 

## Learning music making

For our feasibility experiments, Alexis had [modified a public p5 file](https://editor.p5js.org/ajia-darkened/sketches/fIH6e2e2q) with sound manipulation to include sound files that sounded more ambient and nature-adjacent. Based on this, I went on a deep dive into music making videos to understand what the [p5 sound reference](https://p5js.org/reference/p5.sound/) was talking about. [This](https://youtu.be/eu0zpa7OiYA?si=fk_6BjrHq0pa1FI9) video was the most helpful in understanding the terminology and [this series by The Coding Train](https://youtu.be/Bk8rLzzSink?si=QiF0s0uCdKgHacHy) guided my understanding of sound manipulation in p5. 

The premise of Alexis's p5 program is that when the user presses a button corresponding to specific type of object (to be replaced by computer vision detection later), it plays a random sound continuously from a set associated with the label of the button. Her original version only had three categories, so I sourced more sounds to create five based on the best predictions made by the Teachable Machine model: stick, rock, pinecone, pineneedle, leaf. [Here is the p5 file.](https://editor.p5js.org/ajia-darkened/sketches/e-a8E_p8D)
</br>
</br>
<img width="800" alt="Screenshot 2025-12-12 at 3 30 56 PM" src="https://github.com/user-attachments/assets/ea18f842-a055-4f4e-bf05-7f30c5e33280" />
</br>
You can see and hear it working [here].

## Building up the functionality

The "five buttons" code would sometimes run into a buffer error since in switching between audio sets, it began loading the sound rather than playing it. So, I revised the code to work with our computer vision model, pre-load all the sounds, and layer them together as more objects were placed. I also added a slider in p5 to control volume, mimicking the sliders and potentiometers we wanted to add through serial later. Here is the p5 file for this version. I had picked the sound files somewhat randomly based on what was easy to find, and it really showed when testing ([video](https://drive.google.com/file/d/1z9dqLS7_6EfYGYPc5Vv4CYpvdw8UiQbb/view?usp=sharing)). 
</br>
</br>
<img width="600" alt="Screenshot 2025-12-12 at 3 48 48 PM" src="https://github.com/user-attachments/assets/068fba7a-9251-4582-8975-391d2f11e558" />
</br>
</br>
I attempted to find better sound files to make them sound better together, replacing some of the unusual or abrupt sound files I had added before. You can hear the difference [here](https://drive.google.com/file/d/17nyxIfTCZkyDxZvTV4BJw9jxZhLFSQlE/view?usp=sharing). 

I was pretty happy with this functionality, and decided to tackle the sound effects/manipulation in following weeks. 

In preparation for fabrication, I started soldering wires to the rotating and sliding potentiometers and used the built-in test code for potentiometers to make sure it worked. 
</br>
</br>
<img width="300" alt="image" src="https://github.com/user-attachments/assets/3231395d-31c4-4b6e-941a-934b9ad50b50" />
</br>
The actual wiring is really hard to see, so I made a quick fritzing diagram to show remember the connections (TinkerCAD doesn't have ESP32 nor sliding potentiometers so I did what I could):
</br>
</br>
<img width="600" alt="image" src="https://github.com/user-attachments/assets/2982823c-3c0f-48db-8305-f130033d2781" />

As you can see from the diagram, I began to plan out functionality of the buttons and dials to add into the diagram. With the wiring planned out and the wires soldered to each part, we could focus on developing the form. 
