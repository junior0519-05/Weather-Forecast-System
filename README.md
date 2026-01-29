# Weather-Forecast-System
esp32c3 weather station




read me 


attach code here for now babba ::


current code for weather forecast system babbita central

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
////// CSSS


const char STYLE_CSS[] PROGMEM = R"rawliteral(
* { margin: 0; padding: 0; box-sizing: border-box; }
body {
  font-family: 'Roboto', sans-serif;
  background: linear-gradient(135deg, #2c3e50, #000000);
  color: #ffffff;
  min-height: 100vh;
  padding: 16px;
}

body::before {
  content: "";
  position: absolute;
  top: -20px;
  right: -20px;
  width: 150px; 
  height: 150px;
  border-radius: 50%;
  box-shadow: 0 0 50px rgba(253, 187, 45, 0.4);
  z-index: -1; 
  transition: all 1s ease;
}

body.day-mode::before{
  background: radial-gradient(circle, #fdbb2d 0%, #ff8c00 70%, transparent 100%);
  box-shadow: 0 0 50px rgba(253, 187, 45, 0.4);
}

body.night-mode::before{
  background: radial-gradient(circle, #bdc3c7 0%, #2c3e50 70%, transparent 100%);
  box-shadow: 0 0 40px rgba(255, 255, 255, 0.2);
}
.container { max-width: 480px; margin: 0 auto; }
header { text-align: center; margin-bottom: 24px; }
h1 { font-size: 2.1rem; font-weight: 500; margin-bottom: 4px; }
#location { font-size: 1rem; opacity: 0.8; margin-bottom: 8px; }
#time { font-size: 1.4rem; font-weight: 300; letter-spacing: 2px; }
.card {
  background: rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(10px);
  -webkit-backdrop-filter: blur(10px);
  border-radius: 16px;
  border: 1px solid rgba(255,255,255,0.12);
  padding: 20px;
  margin-bottom: 20px;
  box-shadow: 0 8px 32px rgba(0,0,0,0.25);
}
.current-conditions { display: flex; justify-content: space-between; align-items: center; }
.temp-big { font-size: 4.8rem; font-weight: 300; line-height: 0.9; }
.temp-big sup { font-size: 2.2rem; vertical-align: 1.2rem; opacity: 0.8; }
.condition-details { text-align: right; font-size: 1.3rem; line-height: 1.5; opacity: 0.9; }
.stats-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 12px; }
.stat-item { text-align: center; padding: 16px 8px; }
.stat-item .label { font-size: 0.8rem; opacity: 0.7; margin-bottom: 6px; text-transform: uppercase; }
.stat-item .value { font-size: 1.4rem; font-weight: 500; }
footer { text-align: center; margin-top: 32px; font-size: 0.85rem; opacity: 0.6; }
)rawliteral";

// ==========================================
// 2. JAVASCRIPT FILE (script.js)
// ==========================================
const char SCRIPT_JS[] PROGMEM = R"rawliteral(
function updateTime() {
  const now = new Date();
  document.getElementById('time').textContent = now.toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'});
}
function fetchData() {
  fetch('/data').then(res => res.json()).then(data => {
    document.getElementById('temperature').textContent = data.temp.toFixed(1);
    document.getElementById('temp-now').textContent = data.temp.toFixed(1) + "°";
    document.getElementById('hum-now').textContent = data.hum + "%";
    document.getElementById('pres-now').textContent = Math.round(data.pres);
    document.getElementById('humidity').textContent = data.hum + "%";
    document.getElementById('pressure').textContent = Math.round(data.pres) + " hPa";
    document.getElementById('last-update').textContent = new Date().toLocaleTimeString([], {hour: '2-digit', minute:'2-digit', second:'2-digit'});
  });
}
function updateTheme() {
const hour = new Date().getHours();
  const body = document.body;

  // Between 5:00 AM (5) and 7:00 PM (19)
  if (hour >= 5 && hour < 19) {
    body.classList.add('day-mode');
    body.classList.remove('night-mode');
  } else {
    body.classList.add('night-mode');
    body.classList.remove('day-mode');
  }
}
setInterval(updateTime, 1000);
setInterval(fetchData, 5000);
window.onload = () => { updateTime(); fetchData(); updateTheme(); };
)rawliteral";




/// RAW HTML _----------------------------------------------------

const char INDEX_HTML[] PROGMEM = R"rawliteral(
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>C3 Weather Hub</title>
  <link rel="stylesheet" href="/style.css">
</head>
<body>
  <div class="container">
    <header>
      <h1>Weather Station</h1>
      <p id="location">Indoor — Palmdale, CA</p>
      <div id="time">--:--</div>
    </header>
    <main>
      <div class="current-conditions card">
        <div class="temp-big"><span id="temperature">--</span><sup>°C</sup></div>
        <div class="condition-details">
          <div id="humidity">-- %</div>
          <div id="pressure">-- hPa</div>
        </div>
      </div>
      <div class="stats-grid">
        <div class="card stat-item">
          <div class="label">Temp</div>
          <div class="value" id="temp-now">--</div>
        </div>
        <div class="card stat-item">
          <div class="label">Humidity</div>
          <div class="value" id="hum-now">--</div>
        </div>
        <div class="card stat-item">
          <div class="label">Pressure</div>
          <div class="value" id="pres-now">--</div>
        </div>
      </div>
    </main>
    <footer><p>ESP32-C3 • BME280 • Updated <span id="last-update">--</span></p></footer>
  </div>

  <script src="/script.js"></script>
</body>
</html>
)rawliteral";
//new





// raw HTML ---------------------------------------------------
Adafruit_BME280 weatherSensor; //object for BME sesnor
void handleData() { //this void function gathers info to send to the webserver which gets called in the void loop. // currently just a simple web command to send simple readings babbita 
                              // this is a raw API being sent, basically sending the raw values as a json string, in which the web dev assigns those strings into the webserver.
  String json = "{\"temp\":" + String(weatherSensor.readTemperature(), 2) +  // assigns the string to "temp" to also attach the value along with it 
                ",\"hum\":" + String(weatherSensor.readHumidity(), 1) + 
                ",\"pres\":" + String(weatherSensor.readPressure() / 100.0F, 2) + "}";
  server.send(200, "application/json", json); //status 200 is a standard response for a successful http request.and then application/json grabs the whole thing as a category
}

void setup() {

Serial.begin(115200);

weatherSensor.begin(0x76); //pass in the i2c address of the device

WiFi.softAPConfig(localIP, gatewayIP,subNet); //configutes IP settings 
WiFi.softAP(ssid,password); //creates network with given name and password and allows for connections to be established

// Main Page
server.on("/", []() { server.send_P(200, "text/html", INDEX_HTML); });
  
// The separated CSS and JS routes
server.on("/style.css", []() { server.send_P(200, "text/css", STYLE_CSS); });
server.on("/script.js", []() { server.send_P(200, "application/javascript", SCRIPT_JS); });   //sends the scripts
  
  // Data API
server.on("/data", handleData); // sends the function which manipulated the data into a string.

server.begin(); //initiate webserver

}

void loop() {

server.handleClient(); // does EVERYTHING in terms of updating the sensor readings.

static unsigned long lastTime = 0;
if(millis() - lastTime > 2000) {
    Serial.print("Temp: ");
    Serial.println(weatherSensor.readTemperature()); 
    lastTime = millis();
}
/*Serial.print("Temperature = "); 
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
*/
}





--

