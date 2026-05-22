#include <DHT.h>
#include <WiFi.h>
#include <PubSubClient.h>
#include <Wire.h>
#include <Adafruit_BMP085.h>
#include <Adafruit_GFX.h>
#include <Adafruit_SSD1306.h>

// DHT22 Setup
#define DHTPIN 4
#define DHTTYPE DHT22
DHT dht(DHTPIN, DHTTYPE);
#define MQ2_PIN 34
#define TRIG_PIN 5
#define ECHO_PIN 18

// OLED Setup
#define SCREEN_WIDTH 128
#define SCREEN_HEIGHT 64
Adafruit_SSD1306 display(SCREEN_WIDTH, SCREEN_HEIGHT, &Wire, -1);

// BMP180 Setup
Adafruit_BMP085 bmp;

// WiFi credentials
const char* ssid = "Wokwi-GUEST";
const char* password = "";

// HiveMQ broker
const char* mqtt_server = "broker.hivemq.com";
const int mqtt_port = 1883;

const char* topic_temp     = "revol/iot/temperature";
const char* topic_hum      = "revol/iot/humidity";
const char* topic_pressure = "revol/iot/pressure";
const char* topic_altitude = "revol/iot/altitude";
const char* topic_gas      = "revol/iot/gas";
const char* topic_distance = "revol/iot/distance";

WiFiClient espClient;
PubSubClient client(espClient);

void connectWiFi() {
    Serial.print("Connecting to WiFi");
    WiFi.begin(ssid, password);
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }
    Serial.println("\nWiFi Connected!");
}

void connectMQTT() {
    while (!client.connected()) {
        Serial.print("Connecting to MQTT...");
        if (client.connect("ESP32_Revol_002")) {
            Serial.println("Connected to HiveMQ!");
        } else {
            Serial.print("Failed, rc=");
            Serial.println(client.state());
            delay(2000);
        }
    }
}

void setup() {
    Serial.begin(115200);
    dht.begin();
    pinMode(TRIG_PIN, OUTPUT);
    pinMode(ECHO_PIN, INPUT);

    Wire.begin(21, 22);

    if (!display.begin(SSD1306_SWITCHCAPVCC, 0x3C)) {
        Serial.println("OLED not found!");
        while (1);
    }

    display.clearDisplay();
    display.setTextSize(1);
    display.setTextColor(WHITE);
    display.setCursor(10, 10);
    display.println("System Starting...");
    display.display();
    delay(2000);

    if (!bmp.begin()) {
        Serial.println("BMP180 not found!");
        while (1);
    }

    Serial.println("BMP180 found!");
    connectWiFi();
    client.setServer(mqtt_server, mqtt_port);
    connectMQTT();
    Serial.println("=== Revol IoT Multi Sensor Monitor ===");
}

void loop() {
    if (!client.connected()) {
        connectMQTT();
    }
    client.loop();

    // Read DHT22
    float temperature = dht.readTemperature();
    float humidity = dht.readHumidity();

    // Read Gas Sensor
    int gasValue = analogRead(MQ2_PIN);

    // Read BMP180
    float pressure = bmp.readPressure() / 100.0;
    float altitude = bmp.readAltitude();

    // Read Ultrasonic
    digitalWrite(TRIG_PIN, LOW);
    delayMicroseconds(2);
    digitalWrite(TRIG_PIN, HIGH);
    delayMicroseconds(10);
    digitalWrite(TRIG_PIN, LOW);
    long duration = pulseIn(ECHO_PIN, HIGH);
    float distance = duration * 0.034 / 2;

    if (isnan(temperature) || isnan(humidity)) {
        Serial.println("Failed to read DHT22!");
        return;
    }

    // OLED Display
    display.clearDisplay();
    display.setCursor(0, 0);
    display.println("IoT Monitor");
    display.print("Temp: "); display.print(temperature); display.println(" C");
    display.print("Hum : "); display.print(humidity);    display.println(" %");
    display.print("Pres: "); display.print(pressure);    display.println(" hPa");
    display.print("Alt : "); display.print(altitude);    display.println(" m");
    display.print("Gas : "); display.println(gasValue);
    display.print("Dist: "); display.print(distance);    display.println(" cm");
    display.display();

    // Convert to String
    char tempStr[10];
    char humStr[10];
    char presStr[10];
    char altStr[10];
    char gasStr[10];
    char distStr[10];

    dtostrf(temperature, 4, 2, tempStr);
    dtostrf(humidity, 4, 2, humStr);
    dtostrf(pressure, 4, 2, presStr);
    dtostrf(altitude, 4, 2, altStr);
    dtostrf((float)gasValue, 4, 2, gasStr);
    dtostrf(distance, 4, 2, distStr);

    // Publish MQTT
    client.publish(topic_temp, tempStr);
    client.publish(topic_hum, humStr);
    client.publish(topic_pressure, presStr);
    client.publish(topic_altitude, altStr);
    client.publish(topic_gas, gasStr);
    client.publish(topic_distance, distStr);

    // Serial Monitor
    Serial.println("--- Sensor Readings ---");
    Serial.print("Temperature : "); Serial.print(temperature); Serial.println(" °C");
    Serial.print("Humidity    : "); Serial.print(humidity);    Serial.println(" %");
    Serial.print("Pressure    : "); Serial.print(pressure);    Serial.println(" hPa");
    Serial.print("Altitude    : "); Serial.print(altitude);    Serial.println(" m");
    Serial.print("Gas Level   : "); Serial.println(gasValue);
    Serial.print("Distance    : "); Serial.print(distance);    Serial.println(" cm");
    Serial.println("Published to HiveMQ!");
    Serial.println("----------------------");

    delay(5000);
}
