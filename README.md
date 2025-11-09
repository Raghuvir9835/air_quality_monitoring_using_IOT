#define BLYNK_TEMPLATE_ID "TMPL3x78sUoYY"
#define BLYNK_TEMPLATE_NAME "Air Quality Monitoring"
#define BLYNK_AUTH_TOKEN "H6U66uqZhXnnB99n9YHtsCsrtSMvXaKB"

#include <WiFi.h>
#include <WiFiClient.h>
#include <BlynkSimpleEsp32.h>

#include <Adafruit_Sensor.h>
#include <DHT.h>
#include <DHT_U.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>

// --- Sensor & LCD Config ---
#define DHTPIN 4
#define DHTTYPE DHT11
#define MQ135_PIN 34

DHT dht(DHTPIN, DHTTYPE);
LiquidCrystal_I2C lcd(0x27, 16, 2); 

// --- WiFi Credentials ---
char ssid[] = "R";     // 🔹 Replace with your WiFi SSID
char pass[] = "00998877"; // 🔹 Replace with your WiFi Password

// --- Setup ---
void setup() {
  Serial.begin(115200);

  // Blynk Start
  Blynk.begin(BLYNK_AUTH_TOKEN, ssid, pass);

  dht.begin();
  lcd.init();
  lcd.backlight();
  lcd.setCursor(0,0);
  lcd.print("  Air   Quality ");
  lcd.setCursor(0,1);
  lcd.print("   Monitoring   ");
  delay(2000);
  lcd.clear();
}

// --- Loop ---
void loop() {
  Blynk.run(); // Always run Blynk

  float h = dht.readHumidity();
  float t = dht.readTemperature();
  int gasValue = analogRead(MQ135_PIN);
  gasValue = gasValue -100;

  // --- Send data to Blynk ---
  Blynk.virtualWrite(V0, t);        // Temperature
  Blynk.virtualWrite(V1, h);        // Humidity
  Blynk.virtualWrite(V2, gasValue); // Gas Value

  // --- First Screen: Temp & Humidity ---
  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print("Temp: ");
  lcd.print(t,1);
  lcd.print("C");

  lcd.setCursor(0,1);
  lcd.print("Hum: ");
  lcd.print(h,0);
  lcd.print("%");

  Serial.println("---- Air Quality Monitor ----");
  Serial.print("Temperature: "); Serial.print(t); Serial.println(" °C");
  Serial.print("Humidity: "); Serial.print(h); Serial.println(" %");

  delay(3000);

  // --- Second Screen: Gas Value ---
  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print("Gas: ");
  lcd.print(gasValue);

  if (gasValue < 1200) {
    lcd.setCursor(0,1);
    lcd.print("Fresh Air   ");
    Serial.print("Gas Value: "); Serial.println(gasValue);
    Serial.println("Fresh Air");
    Blynk.virtualWrite(V3, "Fresh Air");
  } else {
    lcd.setCursor(0,1);
    lcd.print("Bad Air!    ");
    Serial.print("Gas Value: "); Serial.println(gasValue);
    Serial.println("Bad Air");
    Blynk.virtualWrite(V3, "Bad Air!");
  }
  if(gasValue > 1200){
  
    Blynk.logEvent("pollution_alert","Bad Air");
  }
  Serial.println("-----------------------------");

  delay(3000);
}


