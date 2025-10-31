# Ambient Display Project (continued)

This week we continued to work our concept for the ambient display project, pulling data from Strava API and refining an enclosure to compare the user's best and latest 5k times. 

## Electronics
With the wiring and basic coding foundation complete, we moved on to setting up the Strava API. Pulling data from Strava is a little more complicated than pulling from Open-Meteo because it requires 0Auth permissions from the user to access the data. After spending some time with Roopa and ChatGPT in office hours, we were able to gain access to my Strava account (Isabella also did this on her own account at the same time). 

First, I made an application in Strava to get the client id, client secret, and other information:
</br>
</br>
<img width="443" src="https://github.com/user-attachments/assets/333568fe-2346-462a-80c6-b63dc6eddd50" />
</br>
</br>
Then using the local host, I could authorize my own access and get the initial authorization key:
</br>
</br>
<img width="300" src="https://github.com/user-attachments/assets/d96a12b4-c419-42f4-ba3c-4b10d6edc01e" />
</br>
</br>
With this authorization key, I could then use curl requests in the terminal to pull my account-specific data (to test calling the API) and upgrade my access to "read-all" from just "read" (which would allow us to get more detailed data about each run):
</br>
</br>
<img width="600" src="https://github.com/user-attachments/assets/0652c7b5-ad75-48c9-ba3e-f2cba12b5d8e" />
</br>
</br>
So now with all the information we needed to access the Strava API, we could attempt the real code. Based on the [API documentation](https://developers.strava.com/docs/reference/#api-models-SummaryActivity), we neeeded to start with a search through all the user's activities for runs 5km or longer (since Strava records "Best Effort" times for each common distance within the distance you ran, for example for 400m, 5km and 10km). There's a lot more to the final functionality, but we wanted to test the most basic level of accessing the Strava API. So we wrote the following code (with some help from ChatGPT, including switching to a newer JSON library) to print the latest activity:

```
#include <secrets.h>
#include <WiFi.h>
#include <HTTPClient.h>
//#include <Arduino_JSON.h>
#include <ArduinoJson.h>
#include <Adafruit_NeoPixel.h>

// ---------- WiFi credentials ----------
const char* ssid = SECRET_SSID;
const char* password = SECRET_PASSWORD;

// ---------- Strava API ----------
String client_id = "182356";
String client_secret = "9c5681ffe4ccd83ef92169edd25241e4b138a976";
String refresh_token = "92c19fe6bd9b40c4270cd336310cea62805309f3";

String access_token;
unsigned long token_expires = 0;

void connectToWiFi() {
  Serial.println("Connecting to WiFi...");
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\n✅ WiFi connected!");
}

// Refresh access token using the refresh token
bool refreshAccessToken() {
  HTTPClient http;
  http.begin("https://www.strava.com/oauth/token");
  http.addHeader("Content-Type", "application/x-www-form-urlencoded");

  String postData = "client_id=" + client_id +
                    "&client_secret=" + client_secret +
                    "&grant_type=refresh_token" +
                    "&refresh_token=" + refresh_token;

  int httpCode = http.POST(postData);
  if (httpCode != 200) {
    Serial.printf("Token refresh failed, HTTP code: %d\n", httpCode);
    http.end();
    return false;
  }

  String payload = http.getString();
  http.end();

  StaticJsonDocument<1024> doc;
  DeserializationError error = deserializeJson(doc, payload);
  if (error) {
    Serial.println("Failed to parse token JSON");
    return false;
  }

  access_token   = doc["access_token"].as<String>();
  refresh_token  = doc["refresh_token"].as<String>();
  token_expires  = doc["expires_at"].as<unsigned long>();

  Serial.println("✅ Access token refreshed!");
  Serial.println("Access token: " + access_token);
  Serial.println("New refresh token: " + refresh_token);
  Serial.println("Expires at (epoch): " + String(token_expires));

  return true;
}

void fetchLatestActivity() {
  if (access_token == "") {
    Serial.println("No access token. Try refreshing first.");
    return;
  }

  HTTPClient http;
  http.begin("https://www.strava.com/api/v3/athlete/activities?per_page=1");
  http.addHeader("Authorization", "Bearer " + access_token);

  int httpCode = http.GET();
  if (httpCode != 200) {
    Serial.printf("Failed to get activities, HTTP code: %d\n", httpCode);
    http.end();
    return;
  }

  String payload = http.getString();
  http.end();

  StaticJsonDocument<4096> doc;
  DeserializationError error = deserializeJson(doc, payload);
  if (error) {
    Serial.println("Failed to parse activity JSON");
    return;
  }
  JsonObject act = doc[0];
  String name = act["name"].as<String>();
  float distance = act["distance"].as<float>() / 1000.0; // meters → km
  String type = act["type"].as<String>();
  String date = act["start_date_local"].as<String>();

  Serial.println("🏁 Latest Activity:");
  Serial.println("Name: " + name);
  Serial.println("Type: " + type);
  Serial.printf("Distance: %.2f km\n", distance);
  Serial.println("Date: " + date);
}


void setup() {
  Serial.begin(115200);
  connectToWiFi();

  if (refreshAccessToken()) {
    fetchLatestActivity();
  } else {
    Serial.println("❌ Could not refresh token.");
  }
}

void loop() {
  // Optionally refresh every few hours
  static unsigned long lastCheck = 0;
  if (millis() - lastCheck > 3600000) { // every 1 hour
    lastCheck = millis();
    refreshAccessToken();
  }
}
```

Here's the result using my account data (I hadn't used Strava prior to this project):
</br>
</br>
<img width="600" src="https://github.com/user-attachments/assets/c31e5b88-ffed-445d-8ad9-71d2b0cda3db" />
</br>
So I went on a run and then ran the program again, with it successfully showing the result of my run:
</br>
</br>
<img width="600" src="https://github.com/user-attachments/assets/c44ea205-214e-4353-ac6b-0ac0dbc74046" />


This progress provided us with a foundation for understanding JSON documents and objects, as well as how to interact with the Strava API. Next week, we will tackle the true functionality we want in the program. 
</br>

## Fabrication

To upgrade from the wooden laser cut box, we decided to make a rounded box using a combination of laser-cut acrylic and 3D printing:
</br>
<img width="600" src="https://github.com/user-attachments/assets/612f47b0-137a-4442-b0c5-bf0e53919b35" />

Then we designed the new version of the enclosure in Fusion:
</br>
<img width="400" src="https://github.com/user-attachments/assets/c1b95d93-ac26-43ee-885b-c8b78cc34cfa" />
<img width="400" src="https://github.com/user-attachments/assets/b83998cf-c31c-4bb5-81c1-ca29dce32ffe" />
</br>
</br>
And then we set it up to print on the Prusa 3D printer:
</br>
<img width="400" src="https://github.com/user-attachments/assets/79aff06d-b516-4d08-bdf7-017c4d139edf" />
</br>
</br>
However, due to poor plate adhesion and too small of a brim, the print warped towards the end of the print, making it unusuable for the final version:
</br>
<img width="400" src="https://github.com/user-attachments/assets/e152c11b-405e-415f-8ddd-30e19793f4bb" />
<img width="400" src="https://github.com/user-attachments/assets/7086c719-f7d8-4786-ba54-9fb0862c62e9" />
<img width="400" src="https://github.com/user-attachments/assets/471abe88-232c-46e5-9465-ad216faa2138" />
<img width="400" src="https://github.com/user-attachments/assets/48a276f6-f7ff-4545-a532-3b69cbd6782b" />
</br>
It was still usable enough for testing, so we tested the assembly with electronics and realized that the black PLA obscured the light too much and the slots for the LEDs came out smaller than expected (you can see it working with the LED simple test [here](https://drive.google.com/file/d/1COcR_1v7Wn5exQPmD2mvQxedGfx1DesR/view?usp=sharing):
</br>
<img width="400" src="https://github.com/user-attachments/assets/4d81aa66-afd5-4cc3-9521-0ceddb7e4ac4" />
</br>

We also practiced using the heat press for the brass inserts we wanted to use on the back of our enclosure:
</br>
<img width="400" src="https://github.com/user-attachments/assets/dfe0b3ba-679a-4976-91a0-9ffee53aaa5a" />

For next week, we plan to print a new version of the enclosure on the Bambu printer with better settings (more brim and organic instead of grid support) and white PLA (for better light diffusion) as well as laser cut acrylic for the front and back. 
