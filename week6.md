# Expressive Mechanics Project

For the next two weeks, we have been assigned the [expressive mechanics project](https://docs.google.com/document/d/1nwybCpWlqz64tKHli3nwZz0LOl2eumrLExoOhAn2S7s/edit?tab=t.0):

>Design and construct a kinetic mechanism that captivates the audience through wonderment. Your mechanism should respond to live camera input using computer vision and machine learning, translating visual information into expressive movement. The goal is to explore how mechanical systems can interact with and react to their environment in surprising, engaging ways.
You will use machine learning algorithms in p5.js to process camera input and send serial messages to an Arduino Uno, which will control motors that animate your mechanism.
>
>Input: Camera input processed with machine learning in p5.js
>
>Output: DC motor motion controlled via Arduino
>
>Interaction: The relationship between camera input and movement should be perceivable and meaningful to the viewer

After much brainstorming, I decided on a concept inspired by the series of comics about Snoopy and the Red Baron (see below). The idea was that a laser cut version of Snoopy on his dog house would be bobbing up in down in the air and people who walked up to the camera could control the tilt of the dog house, giving the illusion of the person controlling how Snoopy flies. 
</br>
</br>
<img width="200" src="https://github.com/user-attachments/assets/f669e54b-44ca-4ba2-b5af-87e14b580043" />
<img width="420" src="https://github.com/user-attachments/assets/5d0a0cbd-eaf4-4a22-8c21-adb65f3dfc66" />
</br>

Based on this idea, I sketched out the project concept in more detail:
<img width="600" src="https://github.com/user-attachments/assets/02cd15e5-f1b1-4496-8e70-d9ccef1c2469" />
</br>
From my last project, I had learned about linkages and had received feedback about using a rack and pinion for linear motion. I thought it might be a good idea to combine my learnings together, utilizing both a servo and DC motor. The servo would move based on the tilt of the person's arms in the webcam, moving a U-shaped linkage that would pull on opposite sides of the dog house to create the same tilting motion. At the same time, the DC motor would slowly bob the dog house up and down continuously through a rack and pinion. Tilting the dog house at the same time as moving it up and down would be challenging, as the servo would have to be attached to the rack. However, I believed this would create the most realistic "flying" motion. 

## Electronics
The main assignment this week for electronics was to get the DC motor or servo responding to the webcam in p5.js via the Arduino board. Since I knew I would need both eventually, I got the [nose example to work for the DC Motor](https://drive.google.com/file/d/1tYP_OFp3BdrSI2DmA4Zx1F9h1uCbegnA/view?usp=sharing) and the [ears example to work with the servo](https://drive.google.com/file/d/1SliUagTSICEKs02AkLumvEG9fHOwhc_S/view?usp=sharing), since the tilting motion was similar to what I wanted. Reviewing the [skeleton tracking example](https://editor.p5js.org/jeffThompson/sketches/acsbpNkef) from class, I looked into the documentation to find parts of the pose net that corresponded to what I wanted. 

I adapted the example p5.js code to include wrists and shoulders instead (it uses the [sample Arduino servo code](https://drive.google.com/file/d/1R1STVD0zu86MtX8HBSkcyhbAYayGRU22/view?usp=sharing)):
```
let pHtmlMsg;
let serialOptions = { baudRate: 115200  };
let serial;
let MtrSpd = 100;
let inc = 1;

let video;
let bodyPose;
let poses = [];

function preload() {
  bodyPose = ml5.bodyPose();
}

function setup() {
	createCanvas(640, 480);
  
  video = createCapture(VIDEO);
  video.size(640, 480);
  video.hide();
  
  bodyPose.detectStart(video, gotPoses);

  // Setup Web Serial using serial.js
  serial = new Serial();
  serial.on(SerialEvents.CONNECTION_OPENED, onSerialConnectionOpened);
  serial.on(SerialEvents.CONNECTION_CLOSED, onSerialConnectionClosed);
  serial.on(SerialEvents.DATA_RECEIVED, onSerialDataReceived);
  serial.on(SerialEvents.ERROR_OCCURRED, onSerialErrorOccurred);

  // If we have previously approved ports, attempt to connect with them
  serial.autoConnectAndOpenPreviouslyApprovedPort(serialOptions);

  // Add in a lil <p> element to provide messages. This is optional
  pHtmlMsg = createP("Click anywhere on this page to open the serial connection dialog");
  pHtmlMsg.style('color', 'deeppink');
}

function gotPoses(newPoses) {
  poses = newPoses;
}

function draw() {
	 image(video, 0, 0, width, height);
  
   if (poses.length > 0) {
		//print(poses);
		//gathering arm data
		let left_wrist = poses[0].left_wrist;
		let left_shoulder = poses[0].left_shoulder;
		let right_wrist = poses[0].right_wrist;
		let right_shoulder = poses[0].right_shoulder;
			
		 //marking parts I'm measuring
     fill (0, 0, 225);
		 circle(left_wrist.x, left_wrist.y, 16);
		 circle(left_shoulder.x, left_shoulder.y, 16);
		 circle(right_wrist.x, right_wrist.y, 16);
		 circle(right_shoulder.x, right_shoulder.y, 16);
		 
		 //checking if the arms are in line
		 let left_diff = left_shoulder.y - left_wrist.y;
         //print(left_diff);
		 let right_diff = right_shoulder.y - right_wrist.y;
         //print(right_diff);
		 let total_diff = left_diff - right_diff;
		 total_diff = constrain(total_diff, -70, 70);
     print(total_diff);
     MtrSpd = map(total_diff, -70, 70, 0, 255);
		 // send data to Arduino
           
     MtrSpd = int(MtrSpd);             // convert to integer (not sure this is necessary)
		 serial.writeLine(MtrSpd);         // send the interger to Arduino
		 
		}	
}

// helper functions below ==========================================================================

/**
 * Callback function by serial.js when there is an error on web serial
 * 
 * @param {} eventSender 
 */
 function onSerialErrorOccurred(eventSender, error) {
  console.log("onSerialErrorOccurred", error);
  pHtmlMsg.html(error);
}

/**
 * Callback function by serial.js when web serial connection is opened
 * 
 * @param {} eventSender 
 */
function onSerialConnectionOpened(eventSender) {
  console.log("onSerialConnectionOpened");
  pHtmlMsg.html("Serial connection opened successfully");
}

/**
 * Callback function by serial.js when web serial connection is closed
 * 
 * @param {} eventSender 
 */
function onSerialConnectionClosed(eventSender) {
  console.log("onSerialConnectionClosed");
  pHtmlMsg.html("onSerialConnectionClosed");
}

/**
 * Callback function serial.js when new web serial data is received
 * 
 * @param {*} eventSender 
 * @param {String} newData new data received over serial
 */
function onSerialDataReceived(eventSender, newData) {
  console.log("onSerialDataReceived", newData);
  pHtmlMsg.html("onSerialDataReceived: " + newData);
}

/**
 * Called automatically by the browser through p5.js when mouse clicked
 */
function mouseClicked() {
  if (!serial.isOpen()) {
    serial.connectAndOpen(null, serialOptions);
  }
}
```
You can see it working [here](https://drive.google.com/file/d/1R1STVD0zu86MtX8HBSkcyhbAYayGRU22/view?usp=sharing), where the arm position controls the servo. I purposefully left out the elbows, even though those are keypoints in the pose net too, to make it easier to manipulate the mechanism and for me to document the process on my own.

Similar to previous assignments and the last project, the Servo twitches and struggles with unreliable readings so I added one line of code based on some Coding Train tutorials about the pose net: 
```
if (left_wrist.confidence > 0.1 && left_shoulder.confidence > 0.1 && right_wrist.confidence > 0.1 && right_shoulder.confidence > 0.1) {
```
This line immediately follows the "gathering arm data" section and encompasses the rest of the code in the draw() function. The pose net includes a "confidence level" for the position of each key point, allowing me to restrict it to more certain values and root out the outliers. Increasing this value to 0.3 as in many Coding Train examples proved not to be responsive enough. The servo still twitched when there were multiple people in the camera, but overall I was satisfied with the servo movement.

## Fabrication
With my coding and electronics working at a level ready for testing, I went to Chris Meyer's office hours to get advice on the mechanism. He suggested I try a crank and slider mechanism instead of a rack and pinion, and use an L-shaped linkage with the servo:
<img width="600" src="https://github.com/user-attachments/assets/f7de1c49-badf-43c5-8916-d8b117e7f247" />
</br>

Based on this advice, I make a cardboard prototype to get a general sense of the size and mechanism:
</br>
<img width="400" src="https://github.com/user-attachments/assets/56b59f5c-4cb7-476b-a12f-f20b00e923b0" />

And then I laser cut a variety of pieces that might be useful for setting up the mechanism:
<img width="600" src="https://github.com/user-attachments/assets/4e31b20c-c7bb-4211-8319-ed3cf6db4afb" />
</br>
<img width="600" src="https://github.com/user-attachments/assets/5572d5bd-ff2b-4392-b579-736f2fbd75b5" />

I made a basic set up with the DC motor attached to the wheel and a slot in a larger box to test the movement ([video](https://drive.google.com/file/d/1Oe10Xf8RKXG2ZhVQlZA10dv6tMDh9rT0/view?usp=sharing)). It generally made the vertical arm move up and down, but dragged the box with it since nothing was firmly attached (you can see in the video I'm holding down the motor with my hand so it doesn't spin on its own).
</br>
<img width="400" src="https://github.com/user-attachments/assets/f859894f-5742-43e0-9fdb-ca3490e8779e" />
</br>

However, after several days and trying multiple sizes of wheels, arms, and support rails as well as various orientations, I just wasn't getting the crank and slider mechanism to work. [Here](https://drive.google.com/file/d/1F67ENKrk7HdCdV72c_aIs_4ox6ja17fv/view?usp=sharing) you can see that the arm attached to the circle isn't actually pushing the vertical arm at all. I could help it along sometimes (as you can see [here](https://drive.google.com/file/d/1YMe7m67QXKyg2p3sizPf0KlRrvRM3pV_/view?usp=sharing)) and was experimenting with spacing of the arms because I was convinced for a while that there wasn't enough clearance. I tried to replicate the "helping it along" through supports, but all these experiments resulted in what the people in the studio coined "motor screaming" (sorry lil guy). 

Doing a [search](https://alistairstutorials.co.uk/tutorial13.html), I found that finding the correct measurements for the wheel and arms would actually be quite difficult and required a bit of math that wasn't quite working for me.

Unfortunately all that testing used up my time this week, but I plan to iterate on this design and try a different mechanism for vertical movement. Thanks for reading!

