#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>
#include <DFRobot_DHT11.h>

DFRobot_DHT11 DHT;

#define SOUND_SENSOR D1
#define DHT11_PIN D5
#define WET_SENSOR A0
#define DC_V D3
#define DC_G D4

char auth[] = BLYNK_AUTH_TOKEN;
char ssid[] = "Redmi";
char pass[] = "rakendhumurali";

unsigned long lastEvent = 0;
int f = 1; // Moved f to the global scope

void setup() {
  Serial.begin(9600);
  Blynk.begin(auth, ssid, pass);
  pinMode(WET_SENSOR, INPUT);
  pinMode(DHT11_PIN, INPUT);
  pinMode(SOUND_SENSOR, INPUT);
  pinMode(DC_V, OUTPUT);
  pinMode(DC_G, OUTPUT);
}

void loop() {
  Blynk.run();
  DHT.read(DHT11_PIN);
  int temp = DHT.temperature;
  Blynk.virtualWrite(V0, temp);
  
  // Read wet sensor value
  int ws_value = analogRead(WET_SENSOR);
  if (ws_value > 500) {
    Blynk.logEvent("Wet Diapers.");
  }
  
  // Read sound sensor value
  int soundState = digitalRead(SOUND_SENSOR);
  Serial.println(soundState);
  
  if (soundState == HIGH) {
    f = 0;
  } else {
    analogWrite(DC_V, 0);
    analogWrite(DC_G, 0);
  }
  
  if (f == 0) {
    Blynk.logEvent("Woken");
    analogWrite(DC_G, 0);
    for (int i = 255; i > 0; i--) {  // Decrease motor speed gradually
      analogWrite(DC_V, i);
      analogWrite(DC_G, 0);
      delay(10); // Delay to slow down the change, adjust as needed
    }
    f = 1; // Reset f to 1 to restart the process
  }
}
