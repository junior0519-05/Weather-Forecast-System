# Weather-Forecast-System
esp32c3 weather station




read me 


attach code here for now babba ::

*****************************
* Code for Babbita Forecast *
*****************************

#include <Arduino.h>
#include <WiFi.h> //library for wifi features
#include <WebServer.h> //library for local webserver hosting
#include <Wire.h>
#include <Adafruit_Sensor.h>
#include <Adafruit_BME280.h>

#define ssid "WeatherMonitor" //name of network
#define password "Babbita345" //password of network

IPAddress localIP(192,168,1,1); //IP address of network
IPAddress gatewayIP(192,168,1,1); //Allows access to network
IPAddress subNet(255,255,255,0);

WebServer server(80); //creates server object with port 80 (port for http)

#define SEALEVELPRESSURE_HPA (1013.25) //declare the base sea level pressue

Adafruit_BME280 weatherSensor; //object for BME sesnor



void setup() {

  Serial.begin(115200);

  weatherSensor.begin(0x76); //pass in the i2c address of the device


  WiFi.softAPConfig(localIP, gatewayIP,subNet); //configutes IP settings
  WiFi.softAP(ssid,password); //creates network with given name and password and allows for connections to be established

  server.begin(); //initiate webserver


}

void loop() {

   Serial.print("Temperature = ");
    Serial.print(weatherSensor.readTemperature());
    Serial.println(" °C");

    Serial.print("Pressure = ");

    Serial.print(weatherSensor.readPressure() / 100.0F);
    Serial.println(" hPa");

    Serial.print("Approx. Altitude = ");
    Serial.print(weatherSensor.readAltitude(SEALEVELPRESSURE_HPA));
    Serial.println(" m");

    Serial.print("Humidity = ");
    Serial.print(weatherSensor.readHumidity());
    Serial.println(" %");

    Serial.println();
  
  server.handleClient();
  
  delay(1000);




}





--

