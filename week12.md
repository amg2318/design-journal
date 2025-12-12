# Final Project (continued)

This week we continued to work our concept for the final project, conducting feasibility experiments to ensure our concept was achievable within the timeframe (~5 weeks). I experimented with computer vision models, Alexis researched sound synthesis and Jill experimented with connecting computer vision to Neopixels. I will briefly cover their experiments, but mostly focus on my own discoveries with computer vision.

## Experiment 1: Teachable Machine

At its minimum viability, our project needed to detect at least one natural object at a time. Based on Jeff's recommendation, I decided to experiment with training my own Teachable Machine model and getting it to work in p5.js. I gathered a wide array of natural objects, since we weren't really sure exactly what specific objects we wanted to train the model with:
</br>
</br>
<img width="300" alt="image" src="https://github.com/user-attachments/assets/e3bfb772-0fdf-4e12-b439-2b3914c434b9" />
<img width="500" alt="image" src="https://github.com/user-attachments/assets/8c634830-48e4-40c6-9e6d-6ecde2fb0394" />
</br>
</br>
For this first attempt, I tried out just three classes: neutral, stick and leaf. [Here's](https://drive.google.com/file/d/1r4hIX5gHpBlTp2RpB3kx7GP8odgR8dzX/view?usp=sharing) a relatively long video of me experimenting with that model.

For the second attempt, I just tried to group all the objects I had at once and incorporated type as different classes (e.g. red leaf, yellow leaf, etc.). I couldn't find much variety in the other objects as much, at least in the neighborhood around my apartment, my parent's backyard, and the park by their house. It seemed to struggle with differentiating between acorns and rocks, and failed to detect in low light conditions. I edited the same version a little bit to try to get some better results. [Here's](https://drive.google.com/file/d/1yHohSVMHnXZZcJ6rnA1vD1u0fqL0lyaa/view?usp=sharing) another relatively long video of how that one worked.

I was pretty satisfied with this attempt as I planned to experiment with multiple item detection later in the week, and used the built-in "transfer to p5.js" function in Teachable Machine. However, it seems that feature is a little out of date. You can find my simple p5.js program [here](https://editor.p5js.org/ajia-darkened/sketches/32sx2D22s). 

Based on this program, Jill was able to experiment with what we could do with this detection. Using some slightly altered code from Sudhu's p5.js serial connection tutorial, she was able to get an [LED to light up when a leaf was detected](https://drive.google.com/file/d/1vtj2b8VbH2mImwvg4GHcLcHKxjPww_RH/view?usp=sharing):
</br>
</br>
<img width="200" alt="image" src="https://github.com/user-attachments/assets/6ce3efe5-9446-4f70-b790-1dd1059ab950" />
</br>
Then, based on [Alexis's p5 sound library explorations](https://editor.p5js.org/ajia-darkened/sketches/fIH6e2e2q), Jill combined both of our feasibility experiments to [make a sound play when a particular object was detected](https://drive.google.com/file/d/1XYgGAuX7QLBFvkJzniDo9cIOns1KFk0N/view?usp=sharing). 

## Experiment 2: Roboflow + Multiple Object Detection

I did some research and found that I could train my own COCO-SSD model on natural objects through Roboflow and Google Colab. There are multiple different ways to achieve this within their platform, but their main promoted method is through Roboflow Rapid.
</br>
</br>
<img width="800" alt="image" src="https://github.com/user-attachments/assets/d3242ef9-fa31-40f4-b2f3-ede69ff85c5c" />
</br>
</br>
Roboflow Rapid uses existing models (both official and user-created) to quickly make a detection model based on some images you include and labels of objects to find, which they try to match to what they have already. Here's what that big photo I took looks like after using Roboflow Rapid:
</br>
</br>
<img width="600" alt="image" src="https://github.com/user-attachments/assets/217f4711-caa4-42b8-a159-8acf2c6eb3cc" />
</br>
</br>
The model is output as an API you can call from your code, but fully publishing it like this costs money unfortunately. So I tried the more manual method (Roboflow Instant, which isn't very "instant"), where I manually make labels on each of the items in photos myself:
</br>
</br>
<img width="800" alt="image" src="https://github.com/user-attachments/assets/05899ecd-4f50-44ab-b6e0-300908b6d19e" />
</br>
</br>
Ideally, I would do this with 100+ photos but I wanted to just make sure I could use it and tried to train the model with what I had. I ended up importing images from my Teachable Machine models also to help with the numbers. Then, I exported it to [Google Colab](https://colab.research.google.com/drive/1SycN5GS7OvosmuruDpsE1jkHfburwN1A?usp=sharing) to generate a file for me to add to p5.js with the "help" of ChatGPT. I got to this point at the time of the feasibility experiment presentation and thought that it seemed like the next steps were fairly simple. 

I was very wrong.

Over the course of many, many hours of work, the code in Google Colab would just not function regardless of which AI I put it in to debug and then finally through a series of real phone calls with the Roboflow team I discussed what I was trying to do and determined it would (1) probably cost money in order to make near-real-time detections and (2) be too complicated for me to figure out when we have a lot of other aspects of the project remaining. 

Very sadly, we decided to stick with single object detection and that I would learn how to do this at another time in the future. For the following week, I planned to learn sound synthesis, the p5 sound library, and start building in the sound functionality into our program.
