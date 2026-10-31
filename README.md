# IoT-Water-Quality-Monitoring-System
IoT-based water quality monitoring using ESP32 and sensor interfacing.
#define BLYNK_TEMPLATE_ID "TMPL394oNaQDr"
#define BLYNK_TEMPLATE_NAME "Smart Water Quality Monitoring"
#define BLYNK_AUTH_TOKEN "tZDcRR_qj-OzZ_V-VqvIEIB_qc3jmCaJ"

#define BLYNK_PRINT Serial

#include <WiFi.h>
#include <WiFiClient.h>
#include <Wire.h>
#include <LiquidCrystal_I2C.h>
#include <BlynkSimpleEsp32.h>
#include <ThingSpeak.h>

// WiFi Credentials
char ssid[] = "rawal";
char pass[] = "63624096";

// Blynk
char auth[] = BLYNK_AUTH_TOKEN;

// ThingSpeak
WiFiClient client;
unsigned long myChannelNumber = 3401098;
const char *myWriteAPIKey = "Z7XOATT8LNKVXNOV";

// LCD
LiquidCrystal_I2C lcd(0x27, 16, 2);

// TDS Sensor Pin
#define TDS_PIN 34

float ec = 0;
int tds = 0;

void setup()
{
  Serial.begin(115200);

  analogReadResolution(12);

  Wire.begin(21, 22);

  lcd.init();
  lcd.backlight();

  lcd.setCursor(0, 0);
  lcd.print("Water Quality");
  lcd.setCursor(0, 1);
  lcd.print("Monitoring");
  delay(2000);

  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Connecting...");

  Blynk.begin(auth, ssid, pass);

  ThingSpeak.begin(client);

  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("WiFi Connected");
  delay(2000);
  lcd.clear();
}

void loop()
{
  Blynk.run();

  int adcValue = analogRead(TDS_PIN);

  // ESP32 ADC -> Voltage
  float voltage = adcValue * 3.3 / 4095.0;

  // EC Calculation
  ec = voltage - 0.14;

  if (ec < 0)
  {
    ec = 0;
  }

  // TDS Calculation
  tds = (133.42 * pow(ec, 3)
        - 255.86 * pow(ec, 2)
        + 857.39 * ec) * 0.5;

  // Serial Monitor
  Serial.print("ADC: ");
  Serial.println(adcValue);

  Serial.print("Voltage: ");
  Serial.println(voltage, 2);

  Serial.print("TDS: ");
  Serial.println(tds);

  Serial.print("EC: ");
  Serial.println(ec, 2);

  // LCD Display
  lcd.setCursor(0, 0);
  lcd.print("TDS:");
  lcd.print(tds);
  lcd.print(" ppm   ");

  lcd.setCursor(0, 1);
  lcd.print("EC:");
  lcd.print(ec, 2);
  lcd.print("      ");

  // Blynk Update
  Blynk.virtualWrite(V0, tds);
  Blynk.virtualWrite(V1, ec);

  // ThingSpeak Update
  ThingSpeak.setField(1, tds);
  ThingSpeak.setField(2, ec);

  int statusCode = ThingSpeak.writeFields(myChannelNumber, myWriteAPIKey);

  if (statusCode == 200)
  {
    Serial.println("ThingSpeak Update Successful");
  }
  else
  {
    Serial.print("ThingSpeak Error Code: ");
    Serial.println(statusCode);
  }

  Serial.println("--------------------------------");

  delay(15000);   // ThingSpeak free limit
}
