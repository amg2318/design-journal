# Ambient Display (continued)

In our final week of the ambient display project, Isabella and I were able to reach the functionality we were aiming for: an aesthetic box that visualizes the comparison between a runner's best and latest 5k times with LEDs. 

## Fabrication

I'm writing about the fabrication first this week since the bulk of it was completed before our coding and electronics section finished, despite working on them simultaneously. Since our first attempt at 3d printing a box failed, we edited our file and tried again with different print settings on the Bambu with white PLA filament:
</br>
</br>
<img width="300" src="https://github.com/user-attachments/assets/3d7a4cf3-ff63-4e38-a678-191e937e4c91" />
<img width="300" src="https://github.com/user-attachments/assets/680d138b-cf47-4f8c-a509-6d8f51e8930d" />
</br>
</br>
At first we tried printing face down, but it seemed like that print was going strangely (see above) so we stopped the print and flipped the print so that the open side was face down instead:
</br>
</br>
<img width="300" src="https://github.com/user-attachments/assets/08df4db5-8e0b-42d8-a67a-ee62a4a8245a" />
<img width="300" src="https://github.com/user-attachments/assets/1b396089-3f33-4c35-aada-cf0ec7d00286" />
<img width="300" src="https://github.com/user-attachments/assets/23cfaab5-ce90-488d-a523-37b09890012b" />
</br>
</br>
This time, our enclosure printed so cleanly and we were able to set the brass inserts in easily. Then, we designed the front insert and the back cover for the box in Illustrator:
</br>
</br>
<img width="200" src="https://github.com/user-attachments/assets/a82933f3-5807-472c-8df1-fea938a3945c" />
<img width="200" src="https://github.com/user-attachments/assets/4a13e682-e49a-45e7-8062-d80fe5704274" />
</br>
</br>
We cut a lot of pieces of clear acrylic trying to get the size exactly right to fit in the slot in our 3D printed enclosure, but ultimately cut some scrap frosted acrylic for the final fit. You can see that the first few attempts just barely didn't fit the enclosure:
</br>
</br>
<img width="300" src="https://github.com/user-attachments/assets/9a381887-b0c8-48e0-9ac9-dd5561bc923f" />
<img width="300" src="https://github.com/user-attachments/assets/b74cb3de-9dc7-42c8-918b-bfffddaaa4c0" />
</br>
</br>
With our fabrication pieces completed, we assembled the enclosure with the electronics and ran the Neopixel test on it as you can see [here](https://drive.google.com/file/d/1ZZO0jw7mNUXohPvjgUlJToTK8QE3OtWQ/view?usp=sharing). Then we focused on finishing up our code. 

## Electronics

We began this week by making a sequential diagram to better understand what our code needed to do:
</br>
![seqdiagram](https://github.com/user-attachments/assets/b5dc1cfa-86da-494c-a495-aa4bd7982722)
</br>

In our first version, we had the program pull from the API once on upload to the ESP32 and use the loop function to refresh the access code:
```
#include <secrets.h>
#include <WiFi.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>
#include <Adafruit_NeoPixel.h>

// Which pin on the Arduino is connected to the NeoPixels?
#define PIN1        12 // Data IN pin
#define PIN2        15 // Data IN pin
// How many NeoPixels are attached to the Arduino?
#define NUMPIXELS 16 // NeoPixel stick size
Adafruit_NeoPixel pixels1(NUMPIXELS, PIN1, NEO_GRB + NEO_KHZ800);
Adafruit_NeoPixel pixels2(NUMPIXELS, PIN2, NEO_GRB + NEO_KHZ800);

// ---------- WiFi credentials ----------
const char* ssid = SECRET_SSID;
const char* password = SECRET_PASSWORD;

// ---------- Strava API ----------
// String client_id = "182356";
// String client_secret = "9c5681ffe4ccd83ef92169edd25241e4b138a976";
// String refresh_token = "92c19fe6bd9b40c4270cd336310cea62805309f3";

String client_id = "183082";
String client_secret = "4f4b9fbf0ba5384e6e50b59add344bd33ca8734c";
String refresh_token = "e342b07ab1404e4aa5afe630f9df0db76e4b70de";

String access_token;
unsigned long token_expires = 0;

// ---------- Data structures ----------
struct TimeArray {
  int times[2]; // [latest, best]
};

struct BestEffort {
  String name;
  float distance;
  int elapsed_time;
  int pr_rank;
  bool valid;
};

// ---------- Function declarations ----------
void connectToWiFi();
bool refreshAccessToken();
BestEffort getBestEffort(long id);
TimeArray getBestandLatest(const char* typeWanted);

// ---------- Function definitions ----------
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

BestEffort getBestEffort(long long id) {
  BestEffort result = {"", 0, 0, 0, false};
  HTTPClient http;
  String url = "https://www.strava.com/api/v3/activities/" + String(id);
  http.begin(url);
  http.addHeader("Authorization", "Bearer " + access_token);
  int code = http.GET();
  if (code != 200) {
    Serial.printf("Detail fetch error: %d\n", code);
    http.end();
    return result;
  }

  DynamicJsonDocument doc(8192);
  DeserializationError err = deserializeJson(doc, http.getString());
  if (err) {
    Serial.println("Parse error detail");
    http.end();
    return result;
  }

  JsonArray efforts = doc["best_efforts"].as<JsonArray>();
  if (efforts.isNull()) {
    Serial.println("No best efforts for this run.");
    http.end();
    return result;
  }

  for (JsonObject effort : efforts) {
    if (effort["distance"] == 5000.0) {
      result.name = effort["name"].as<String>();
      result.distance = effort["distance"].as<float>();
      result.elapsed_time = effort["elapsed_time"].as<int>();
      result.pr_rank = effort["pr_rank"].isNull() ? 0 : effort["pr_rank"].as<int>();
      result.valid = true;
    }
  }
  http.end();
  return result;
}

TimeArray getBestandLatest(const char* typeWanted) {
  HTTPClient http;
  int page = 1;
  bool more = true;

  TimeArray data = {{0, 0}};
  bool latest = false;
  bool best = false;

  while (more && page <= 5) {
    String url = "https://www.strava.com/api/v3/athlete/activities?page=" + String(page) + "&per_page=30";
    http.begin(url);
    http.addHeader("Authorization", "Bearer " + access_token);
    int code = http.GET();
    if (code != 200) { Serial.printf("Error %d\n", code); break; }

    DynamicJsonDocument doc(32768);
    DeserializationError err = deserializeJson(doc, http.getString());
    if (err) { Serial.println("Parse error"); break; }
    if (doc.size() == 0) { more = false; break; }

    for (JsonObject act : doc.as<JsonArray>()) {
      const char* actType = act["type"];
      if (strcmp(actType, typeWanted) == 0 && act["distance"] > 5000.0) {
        long long id = act["id"].as<long long>();
        //Serial.println(String(id));
        BestEffort effort = getBestEffort(id);
        if (effort.valid) {
          if (!latest) {
            data.times[0] = effort.elapsed_time;
            latest = true;
          }
          if (!best && effort.pr_rank == 1) {
            data.times[1] = effort.elapsed_time;
            best = true;
          }
          if (latest && best) {
            http.end();
            return data;
          }
        }
      }
    }
    page++;
    http.end();
    delay(1000);
  }
  return data;
}

// ---------- Setup ----------
void setup() {
  Serial.begin(115200);
  connectToWiFi();
  pixels1.begin();
  pixels2.begin();
  pixels1.clear();
  pixels2.clear();

  if (refreshAccessToken()) {
    TimeArray test = getBestandLatest("Run");
    Serial.println("Latest 5k time: " + String(test.times[0]) + " s");
    int num_Latest = map(test.times[0], 1500, 1800, 2, 15);
    //Serial.println("Number of lights for latest: " + String(num_Latest));
    for(int i=0; i<=num_Latest; i++) {
      pixels1.setPixelColor(i, pixels1.Color(0, 127, 0));
      pixels1.show();
    }
    Serial.println("Best 5k time: " + String(test.times[1]) + " s");
    int num_Best = map(test.times[1], 1500, 1800, 2, 15);
    //Serial.println("Number of lights for best: " + String(num_Best));
    for(int i=0; i<=num_Best; i++) {
      pixels2.setPixelColor(i, pixels2.Color(0, 0, 127));
      pixels2.show();
    }
  } else {
    Serial.println("❌ Could not refresh token.");
  }
}

void loop() {
  static unsigned long lastCheck = 0;
  if (millis() - lastCheck > 3600000) { // every 1 hour
    lastCheck = millis();
    refreshAccessToken();
  }
}
```

We experimented a lot with how the values would map to the number of pixels and which side showed the latest or best values. For example, we initially thought the latest time should take up the whole Neopixel stick and the best time would display based on the scale of the difference between the two times (see below). This made it seem like the runner is always slow though, and also defeated the point of showing both times (in our opinion):
</br>
</br>
<img width="400" src="https://github.com/user-attachments/assets/28657b5d-8596-4836-9e1f-2700cddf1128" />
</br>
</br>
Then, we experimented with mapping the values based on our general ability to run a 5K (in the end choosing 25-30 minutes, with 25 minutes being faster than we could achieve and requiring at least some of the pixels to be lit while still making the difference in times observable). Since we were working simultaneously (Isabella was doing fabrication while I was experimenting with the code), I would use an ESP32 that wasn't attached to the box or Neopixels to test sometimes and print the values out instead:
</br>
</br>
<img width="400" src="https://github.com/user-attachments/assets/d535f604-9b06-497b-8021-f9ec4e315a70" />
</br>
Above you can see the times are the same but the number of pixels is changing. I also added 2 extra Neopixels to be lit by default so that the bars of light weren't too small compared to the box.
</br>
</br>
We learned a lot about error codes and how to check if the Strava API was really connecting with the ESP32. Also, we kept having to increase the memory of the initial JSON doc (discovered through debugging with ChatGPT), especially when pulling from Isabella's data, as she is a long time user with many activities to search through. While this code worked very reliably, we needed the code to run constantly to be truly "ambient". Because I was setting new personal records on Strava every time I ran a 5k (we ran a few times during the project to test the readings for the latest data), we also wanted to add a special interaction for when the user beat their best time with a rainbow animation (vibe coded with ChatGPT). Here is the final code, an accumlation of many hours of working through parse errors and memory allocation:
```
#include <secrets.h>
#include <WiFi.h>
#include <HTTPClient.h>
#include <ArduinoJson.h>
#include <Adafruit_NeoPixel.h>

// ---------- NeoPixel Setup ----------
#define PIN1 12
#define PIN2 15
#define NUMPIXELS 16
Adafruit_NeoPixel pixels1(NUMPIXELS, PIN1, NEO_GRB + NEO_KHZ800);
Adafruit_NeoPixel pixels2(NUMPIXELS, PIN2, NEO_GRB + NEO_KHZ800);

// ---------- WiFi credentials ----------
const char* ssid = SECRET_SSID;
const char* password = SECRET_PASSWORD;

// ---------- Strava API credentials ----------
const char* client_id = CLIENT_ID;
const char* client_secret = CLIENT_SECRET;
const char* refresh_token = REFRESH_TOKEN;

String access_token;
unsigned long token_expires = 0;

// ---------- Data structures ----------
struct TimeArray {
  int times[2]; // [latest, best]
};

struct BestEffort {
  String name;
  float distance;
  int elapsed_time;
  int pr_rank;
  bool valid;
};

// ---------- Function declarations ----------
void connectToWiFi();
bool refreshAccessToken();
BestEffort getBestEffort(long long id);
TimeArray getBestandLatest(const char* typeWanted);
void updateLights();
void rainbowWave();
void showTimeBars(TimeArray test);

// ---------- Connect to WiFi ----------
void connectToWiFi() {
  Serial.println("Connecting to WiFi...");
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.println("\n✅ WiFi connected!");
}

// ---------- Refresh access token ----------
bool refreshAccessToken() {
  HTTPClient http;
  http.begin("https://www.strava.com/oauth/token");
  http.addHeader("Content-Type", "application/x-www-form-urlencoded");

  String postData = "client_id=" + String(client_id) +
                    "&client_secret=" + String(client_secret) +
                    "&grant_type=refresh_token" +
                    "&refresh_token=" + String(refresh_token);

  int httpCode = http.POST(postData);
  if (httpCode != 200) {
    Serial.printf("Token refresh failed, HTTP code: %d\n", httpCode);
    http.end();
    return false;
  }

  String payload = http.getString();
  http.end();

  StaticJsonDocument<1024> doc;
  if (deserializeJson(doc, payload)) {
    Serial.println("Failed to parse token JSON");
    return false;
  }

  access_token   = doc["access_token"].as<String>();
  refresh_token  = doc["refresh_token"].as<String>();
  token_expires  = doc["expires_at"].as<unsigned long>();

  Serial.println("✅ Access token refreshed!");
  return true;
}

// ---------- Fetch a single activity’s 5k effort ----------
BestEffort getBestEffort(long long id) {
  BestEffort result = {"", 0, 0, 0, false};
  HTTPClient http;
  String url = "https://www.strava.com/api/v3/activities/" + String(id);
  http.begin(url);
  http.addHeader("Authorization", "Bearer " + access_token);
  int code = http.GET();

  if (code != 200) {
    Serial.printf("⚠️ Detail fetch error for activity %lld: HTTP %d\n", id, code);
    http.end();
    return result;
  }

  DynamicJsonDocument doc(16384); // Larger buffer for big JSONs
  DeserializationError err = deserializeJson(doc, http.getString());
  if (err) {
    Serial.printf("⚠️ JSON parse error for activity %lld: %s\n", id, err.c_str());
    http.end();
    return result;
  }

  JsonArray efforts = doc["best_efforts"].as<JsonArray>();
  if (efforts.isNull()) {
    http.end();
    return result;
  }

  for (JsonObject effort : efforts) {
    if (effort["distance"] == 5000.0) {
      result.name = effort["name"].as<String>();
      result.distance = effort["distance"].as<float>();
      result.elapsed_time = effort["elapsed_time"].as<int>();
      result.pr_rank = effort["pr_rank"].isNull() ? 0 : effort["pr_rank"].as<int>();
      result.valid = true;
      break;
    }
  }
  http.end();
  return result;
}

// ---------- Get best and latest 5k times ----------
TimeArray getBestandLatest(const char* typeWanted) {
  HTTPClient http;
  int page = 1;
  bool more = true;

  TimeArray data = {{0, 0}};
  bool latestFound = false;
  bool bestFound = false;

  while (more && page <= 3) {  // limit pages for efficiency
    String url = "https://www.strava.com/api/v3/athlete/activities?page=" + String(page) + "&per_page=30";
    http.begin(url);
    http.addHeader("Authorization", "Bearer " + access_token);

    int code = http.GET();
    if (code != 200) {
      Serial.printf("⚠️ Activity list error: HTTP %d\n", code);
      http.end();
      break;
    }

    WiFiClient* stream = http.getStreamPtr();  // 🟢 use stream instead of getString()
    DynamicJsonDocument doc(8192);             // small buffer, reused for each activity

    // 🟢 Use DeserializationStreamIterator to parse progressively
    DeserializationError err = deserializeJson(doc, *stream);
    if (err) {
      Serial.printf("⚠️ Parse stream error (page %d): %s\n", page, err.c_str());
      http.end();
      break;
    }

    JsonArray activities = doc.as<JsonArray>();
    if (activities.isNull() || activities.size() == 0) {
      more = false;
      http.end();
      break;
    }

    for (JsonObject act : activities) {
      const char* actType = act["type"];
      float dist = act["distance"].as<float>();

      if (strcmp(actType, typeWanted) == 0 && dist > 5000.0) {
        long long id = act["id"].as<long long>();
        BestEffort effort = getBestEffort(id);

        if (effort.valid) {
          if (!latestFound) {
            data.times[0] = effort.elapsed_time;
            latestFound = true;
          }
          if (!bestFound && effort.pr_rank == 1) {
            data.times[1] = effort.elapsed_time;
            bestFound = true;
          }
        }
      }

      if (latestFound && bestFound) {
        http.end();
        return data;
      }
    }

    page++;
    http.end();
    delay(500);
  }

  return data;
}

// ---------- Show progress bars ----------
void showTimeBars(TimeArray test) {
  int num_Latest = map(test.times[0], 1500, 1800, 2, 15);
  int num_Best = map(test.times[1], 1500, 1800, 2, 15);

  pixels1.clear();
  pixels2.clear();

  for (int i = 0; i <= num_Latest; i++) {
    pixels1.setPixelColor(i, pixels1.Color(0, 127, 0));
  }
  pixels1.show();

  for (int i = 0; i <= num_Best; i++) {
    pixels2.setPixelColor(i, pixels2.Color(0, 0, 127));
  }
  pixels2.show();
}

// ---------- Rainbow celebration animation ----------
void rainbowWave() {
  Serial.println("🌈 New personal best! Celebrating...");
  for (int j = 0; j < 256; j++) {
    for (int i = 0; i < NUMPIXELS; i++) {
      int color = (i * 256 / NUMPIXELS + j) & 255;
      uint32_t rgb = pixels1.gamma32(pixels1.ColorHSV(color * 256));
      pixels1.setPixelColor(i, rgb);
      pixels2.setPixelColor(i, rgb);
      delay(100);
    }
    pixels1.show();
    pixels2.show();
  }
  pixels1.clear();
  pixels2.clear();
  pixels1.show();
  pixels2.show();
}

// ---------- Update data + lights ----------
void updateLights() {
  if (!refreshAccessToken()) {
    Serial.println("❌ Could not refresh token.");
    return;
  }

  TimeArray test = getBestandLatest("Run");
  Serial.println("Latest 5k time: " + String(test.times[0]) + " s");
  Serial.println("Best 5k time: " + String(test.times[1]) + " s");

  if (test.times[0] > 0 && test.times[1] > 0) {
    if (test.times[0] == test.times[1]) {
      rainbowWave();  // 🌈 celebrate new PR!
    }
  }

  showTimeBars(test);
}

// ---------- Setup ----------
void setup() {
  Serial.begin(115200);
  connectToWiFi();

  pixels1.begin();
  pixels2.begin();
  pixels1.clear();
  pixels2.clear();

  updateLights(); // Run once immediately
}

// ---------- Loop ----------
void loop() {
  static unsigned long lastCheck = 0;
  static unsigned long lastLightUpdate = 0;
  unsigned long now = millis();

  // Update lights every minute
  if (now - lastLightUpdate > 60000) {
    lastLightUpdate = now;
    Serial.println("\n🔄 Updating Strava data + lights...");
    updateLights();
  }

  // Refresh access token hourly
  if (now - lastCheck > 3600000) {
    lastCheck = now;
    refreshAccessToken();
  }
}
```
You can see the program run on my Strava data, demonstrating the rainbow wave as the result of best and latest times being the same [here](https://drive.google.com/file/d/1MeHh8rtwF2PL9WLBg9V_KLg-j0v-wxoe/view?usp=sharing). Here's what it looks like with Isabella's Strava data:
</br>
</br>
<img width="400" src="https://github.com/user-attachments/assets/abddec31-cead-4029-9ef9-92b62a61b977" />
<img width="400" src="https://github.com/user-attachments/assets/f166957c-a129-499d-b20a-350db0968601" />
</br>
</br>
[Here is our final video](https://drive.google.com/file/d/1i9K1tnwmSVJLz_u_KoZypqGiL_PWi5TC/view?usp=drive_link) showing the upload of our latest activities on Strava after running and the lights changing accordingly.

### Reflection
I'm really proud of what we accomplished and working with Isabella was probably my best group project experience to date. If I were to keep working on this, we would add some indicator of which bar is which by engraving the acrylic with icons (a clock for the latest and a trophy for the best time maybe?) though we had left it off to allow the enclosure to be relatively situation agnostic. I think it could be cool to compare different things with the same enclosure, or maybe have changeable plates for the front based on what is being measured. We would also reduce the delay between the rainbow lights and showing the bars, and maybe extend the length of time it takes for that animation to occur. For me, the most challenging part was dealing with the memory load of Strava and parsing the JSON file. Despite its difficulty, I learned a lot and I want to continue to explore using APIs in the future and maybe find better ways of managing memory, maybe with different boards or code libraries.

### Bloopers
We wanted to film the box working at the track so I was rewiring it to use the lithium-ion battery after my run at Edwards Stadium. Unfortunately, it didn't end up working out but here are some pics of us trying to make it work.
</br>
</br>
<img width="400" src="https://github.com/user-attachments/assets/45b3b4fe-da47-480d-afb1-930a53491eaf" />
<img width="400" src="https://github.com/user-attachments/assets/8c6a67c2-5912-41ae-94e1-fb4639d910f0" />

Thanks for reading!
