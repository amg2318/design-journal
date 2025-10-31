# Ambient Display Project (continued)

In our final week of working on the ambient display project, we reach the desired functionality of lighting an aesthetic enclosure like a bar chart to visualize the comparison of a runner's best and latest 5k times. 

## Electronics

We began this week by making sequential diagram to better understand what was required of our code:
</br>
![seqdiagram](https://github.com/user-attachments/assets/2052e4e7-f94e-4c36-8a28-a4d0aa02c2f1)

After carefully understanding how to get the information we wanted from the Strava API, we made a version that would call the API once on upload and use the built-in loop to check if we needed to refresh our access:
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
