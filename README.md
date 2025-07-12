# smoke_and_gas_detection_system
The system detects the fire and gas leakage in ours work places.

//code
#include <LiquidCrystal.h>
#include <SoftwareSerial.h>
#include <DHT.h>

// LCD pins: RS, EN, D4, D5, D6, D7
LiquidCrystal lcd(7, 6, 5, 4, 3, 2);

// GSM Module on pins 10 (TX) and 11 (RX)
SoftwareSerial gsm(10, 11);

// DHT settings
#define DHTPIN 9
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);

// Pin definitions
#define GAS_SENSOR A0
#define BUZZER 12
#define FAN 13

// Thresholds
#define GAS_THRESHOLD 300

void setup() {
  lcd.begin(16, 2);
  dht.begin();
  gsm.begin(9600);
  Serial.begin(9600);

  pinMode(GAS_SENSOR, INPUT);
  pinMode(BUZZER, OUTPUT);
  pinMode(FAN, OUTPUT);

  lcd.print("Gas Smoke Detect");
  delay(2000);
  lcd.clear();
}

void loop() {
  float h = dht.readHumidity();
  float t = dht.readTemperature();
  int gasValue = analogRead(GAS_SENSOR);

  lcd.setCursor(0, 0);
  lcd.print("Temp:");
  lcd.print(t);
  lcd.print("C ");

  lcd.setCursor(0, 1);
  lcd.print("Hum:");
  lcd.print(h);
  lcd.print("%");

  Serial.print("Gas Value: ");
  Serial.println(gasValue);

  if (gasValue > GAS_THRESHOLD) {
    digitalWrite(BUZZER, HIGH);
    digitalWrite(FAN, HIGH);

    lcd.setCursor(0, 1);
    lcd.print("Gas Alert!       ");
    sendSMS(gasValue, t, h);
    delay(10000);  // Wait to avoid spamming SMS
  } else {
    digitalWrite(BUZZER, LOW);
    digitalWrite(FAN, LOW);
  }

  delay(2000);
}

void sendSMS(int gas, float temp, float hum) {
  gsm.println("AT+CMGF=1"); // SMS text mode
  delay(1000);
  gsm.println("AT+CMGS=\"+91XXXXXXXXXX\""); // Replace with your phone number
  delay(1000);
  gsm.print("Gas Detected!\nGas Level: ");
  gsm.print(gas);
  gsm.print("\nTemp: ");
  gsm.print(temp);
  gsm.print(" C\nHumidity: ");
  gsm.print(hum);
  gsm.print("%");
  gsm.write(26); // Ctrl+Z to send
  delay(5000);
}
