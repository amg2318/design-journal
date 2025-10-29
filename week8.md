# Ambient Display Project

We were assigned the [ambient display project](https://docs.google.com/document/d/1QnhFRa3eoGZuhHrPJAbla0htHPtGgmTuGSE_jI-3cew/edit?tab=t.0#heading=h.u9fow02o2yq7):
>Create an ambient device  that presents and communicates information through changes in form, movement, sound, color, or light in a subtle and aesthetically pleasing way, using API calls to pull information from the real world to drive your ambient display. This project may be completed using groups of two. 

My partner for this project is Isabella Fiorante, and you can find her journal [here](https://github.com/ifiorante/TDF-FA25/tree/main). As we're both runners, we decided to pull data from the fitness app Strava and compare run times visually. 

## Electronics

This week I focused on setting up the foundation: understanding and practicing API calls and the ESP32 Feather board. Once I soldered the pins onto the board and set up the Arduino IDE for the ESP32, I worked on [Roopa's Weather API example](https://github.com/roopa-ramanujam/ESP32-web-api-example?tab=readme-ov-file). Troubleshooting this code and reading the [API documentation for Open-Meteo](https://open-meteo.com/en/docs) taught me about how the board is connecting to WiFi and general structure of API calls and their JSON output:

```
#include <secrets.h>
#include <WiFi.h>
#include <HTTPClient.h>
#include <Arduino_JSON.h>
#include <Adafruit_NeoPixel.h>

// ---------- WiFi credentials ----------
const char* ssid = SECRET_SSID;
const char* password = SECRET_PASSWORD;

// ---------- Location (Jacobs Hall, Berkeley) ----------
const double latitude  = 37.876270;
const double longitude = -122.258499;

// ---------- Open-Meteo API ----------
String weather_api = "http://api.open-meteo.com/v1/forecast?latitude="
                     + String(latitude, 6)
                     + "&longitude="
                     + String(longitude, 6)
                     + "&current_weather=true"
                     + "&temperature_unit=fahrenheit";

// ---------- Variables ----------
double currentTemp = -1;  // ⭐ start as invalid
double currentWind = 0;
unsigned long lastUpdate = 0;
const unsigned long updateInterval = 60 * 1000; // refresh every 1 min

// ---------- Onboard NeoPixel ----------
#define PIN_NEOPIXEL        0
#define NEOPIXEL_I2C_POWER  2
#define NUMPIXELS           1

Adafruit_NeoPixel pixel(NUMPIXELS, PIN_NEOPIXEL, NEO_GRB + NEO_KHZ800);

// ---------- Setup ----------
void setup() {
  Serial.begin(115200);
  delay(1000);

  pinMode(NEOPIXEL_I2C_POWER, OUTPUT);
  digitalWrite(NEOPIXEL_I2C_POWER, HIGH); // ⭐ ensure power enabled

  pixel.begin();
  pixel.clear();
  pixel.show();

  connectToWiFi();
  delay(1000);
  getWeatherData();  // initial fetch
}

// ---------- Main Loop ----------
void loop() {
  if (millis() - lastUpdate > updateInterval) {
    getWeatherData();
    lastUpdate = millis();
  }

  // ⭐ Only show light if temperature is valid
  if (currentTemp != -1) {
    visualizeWeather(currentTemp, currentWind);
  } else {
    pixel.clear();
    pixel.show(); // keep LED off when no valid data
  }
}

// ---------- WiFi Connection ----------
void connectToWiFi() {
  Serial.println("Connecting to WiFi...");
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\n✅ WiFi connected!");
}

// ---------- Fetch Weather ----------
void getWeatherData() {
  Serial.print("Requesting: ");
  Serial.println(weather_api);

  if (WiFi.status() == WL_CONNECTED) {
    HTTPClient http;
    http.begin(weather_api);
    int httpResponseCode = http.GET();

    Serial.print("HTTP response code: ");
    Serial.println(httpResponseCode);

    if (httpResponseCode == 200) {
      String payload = http.getString();
      JSONVar jsonObject = JSON.parse(payload);

      if (JSON.typeof(jsonObject) == "undefined") {
        Serial.println("Parsing input failed!");
        currentTemp = -1;  // ⭐ invalidate reading
        http.end();
        return;
      }

      currentTemp = double(jsonObject["current_weather"]["temperature"]);
      currentWind = double(jsonObject["current_weather"]["windspeed"]);

      Serial.print("🌡 Temp: ");
      Serial.print(currentTemp);
      Serial.println(" °F");
      Serial.print("💨 Wind: ");
      Serial.print(currentWind);
      Serial.println(" mph");
    } else {
      Serial.print("HTTP error: ");
      Serial.println(httpResponseCode);
      currentTemp = -1;  // ⭐ invalidate reading
    }
    http.end();
  } else {
    Serial.println("WiFi disconnected. Reconnecting...");
    WiFi.reconnect();
    currentTemp = -1;  // ⭐ invalidate reading
  }
}

// ---------- LED Visualization ----------
void visualizeWeather(double tempF, double windspeed) {
  int red = 0, green = 0, blue = 0;

  if (tempF <= 65) {
    blue = 255;
  } else if (tempF <= 80) {
    green = 255;
  } else {
    red = 255;
  }

  int blinkDelay = map((int)windspeed, 0, 30, 800, 100);

  pixel.setPixelColor(0, pixel.Color(red, green, blue));
  pixel.show();
  delay(blinkDelay);

  pixel.clear();
  pixel.show();
  delay(blinkDelay);
}
```

You can see the weather API tutorial working in [this video](https://drive.google.com/file/d/1kOvlLwP7Vyqj30UbXdfyuSzneW3hRkMg/view?usp=sharing). Based on what we had learned, we brainstormed ideas for our project and settled on comparing 5k run times in a visually aesthetic "bar chart" that would emphasize the difference between the user's latest 5k time and their record (best) time. Here are some of our concept sketches:
</br>
<img width="400" src="https://github.com/user-attachments/assets/96c5996d-bbf5-4a09-af8c-7e0e0338dab8" />
<img width="400" src="https://github.com/user-attachments/assets/f860db52-39a7-400b-9a90-ab40a9149d61" />
<img width="600" src="https://github.com/user-attachments/assets/060ba85f-bf2c-4544-8cf1-7eeb3edbba45" />
</br>
Our original idea was to use servos that would fill curved bars to different lengths but decided that was too complicated for our level of experience with mechanisms. Instead, we went with a linear bar chart using neopixels so that we could spend more time refining the code and overall finish. To get started on this concept, we carefully soldered together two pairs of two 8-pixel sticks for greater visual clarity:
</br>
<img width="400" src="https://github.com/user-attachments/assets/b7ef0f3e-27ab-4a63-8a8c-61c4d604e45b" />
</br>
Once soldered, we wired both expanded Neopixel sticks to the ESP32 and adapted the sample code to test the wiring:

```
 
// NeoPixel stick simple sketch (c) 2024 Sudhu Tewari
// tested with ESP32 Feather V2 10/16/25
// modified from:
// NeoPixel Ring simple sketch (c) 2013 Shae Erisson
// Released under the GPLv3 license to match the rest of the
// Adafruit NeoPixel library

// this code should light all LED sequentially: red, green, blue.
// this is a good test to make sure we've got the wiring and code configured correctly

#include <Adafruit_NeoPixel.h>

// Which pin on the Arduino is connected to the NeoPixels?
#define PIN1        12 // Data IN pin
#define PIN2        15 // Data IN pin
// How many NeoPixels are attached to the Arduino?
#define NUMPIXELS 16 // NeoPixel stick size

// When setting up the NeoPixel library, we tell it how many pixels,
// and which pin to use to send signals. Note that for older NeoPixel
// strips you might need to change the third parameter -- see the
// strandtest example for more information on possible values.
Adafruit_NeoPixel pixelsA(NUMPIXELS, PIN1, NEO_GRB + NEO_KHZ800);
Adafruit_NeoPixel pixelsB(NUMPIXELS, PIN2, NEO_GRB + NEO_KHZ800);

#define DELAYVAL 500 // Time (in milliseconds) to pause between pixels

void setup() {
  pixelsA.begin(); // INITIALIZE NeoPixel strip object (REQUIRED)
  pixelsB.begin();
}

void loop() {
  pixelsA.clear(); // Set all pixel colors to 'off'
  pixelsB.clear();
  // The first NeoPixel in a strand is #0, second is 1, all the way up
  // to the count of pixels minus one.
  for(int i=0; i<NUMPIXELS; i++) { // For each pixel...

   // pixels.Color() takes RGB values, from 0,0,0 up to 255,255,255
   // Here we're using a moderately bright red color:
  
    pixelsA.setPixelColor(i, pixelsA.Color(127, 0, 0)); // 127 = 50% of max brightness >> also 50% current draw
    pixelsB.setPixelColor(i, pixelsB.Color(127, 0, 0));
    pixelsA.show();   // Send the updated pixel colors to the hardware.
    pixelsB.show();
    delay(DELAYVAL); // Pause before next pass through loop
  }

    //GREEN
    for(int i=0; i<NUMPIXELS; i++) { // For each pixel...
      pixelsA.setPixelColor(i, pixelsA.Color(0, 127, 0));
      pixelsB.setPixelColor(i, pixelsB.Color(0, 127, 0));
      pixelsA.show();   // Send the updated pixel colors to the hardware.
      pixelsB.show();
      delay(DELAYVAL); // Pause before next pass through loop
  }

    //BLUE 
    for(int i=0; i<NUMPIXELS; i++) { // For each pixel...
    pixelsA.setPixelColor(i, pixelsA.Color(0, 0, 127));
    pixelsB.setPixelColor(i, pixelsB.Color(0, 0, 127));
    pixelsA.show();   // Send the updated pixel colors to the hardware.
    pixelsB.show();
    delay(DELAYVAL); // Pause before next pass through loop
  }
}
```
You can see both sticks successfully lit and changing colors [here](https://drive.google.com/file/d/1vJdSezX88GQLXEy_zVLmpaKA7t2uhfXi/view?usp=sharing). 

## Fabrication

Based on our sketches, we also prototyped an early version of the enclosure to test sizing and spacing. We carefully measured the Neopixel sticks and the breadboards and plugged these measurements into a box generator. We then edited the designs in Illustrator to incorporate the gaps for the Neopixels, trying to utilize padding and white space for aesthetics:
</br>
<img width="600" src="https://github.com/user-attachments/assets/2e3c11fb-5b37-4337-8cb3-bb77c447eb61" />

We lasercut the box pieces from wood (cheap but rigid enough to make sure it actually held our stuff) and assembled the electronics within it with tape:
</br>
<img width="400" src="https://github.com/user-attachments/assets/8db39e41-2ec8-43ef-8e0d-98f56835f3bf" />
<img width="400" src="https://github.com/user-attachments/assets/6b8351f0-c7c2-4eab-a1dd-165ef23e439a" />

While experimenting with this prototype, we struggled with the cables breaking off of the Neopixels frequently and the gaps (which were based on the width of the LED and not the width of the stick) were awkward and difficult to align the Neopixels with. For next week, we plan to refine our design of the box and 3D print a first version based on our findings. 

