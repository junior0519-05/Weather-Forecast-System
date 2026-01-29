**********************************************************
* Exmaple Code of Inline Styling from Past Project I did *
**********************************************************

//IOT weather station made using ESP-32 Dev Board
//Following Modules are used:
//MQ-135 Air Quality Sensor
//DHT-11 Tempature and Humidity Sensor
//DS3231 RTC Modules (real-time-clock)

#include <WiFi.h>       //Wifi library for ESP-32
#include <WebServer.h>  //Webserver library for ESP-32
#include <DHT.h>    //library necessary for DHT-11 Module
#include <RTClib.h> //library necessary for RTC module


#define ssid "ESP32-Server" //name of network
#define password "Apples456" //password of network

#define A0 34 //analog 0 pin == GPIO 34

IPAddress localIP(192,168,1,1); //IP address of network (This is how one connects to the webpage- to connect one has to be on the ESP-32's network and type in localIP address into search engine)
IPAddress gatewayIP(192,168,1,1); //Allows access to network
IPAddress subNet(255,255,255,0);

WebServer server(80); //creates server object with port 80 (port for http)

//variable delcartions to hold information from RTC module

int hour,
    currentDOW;

String year,
      month,
      day,
      minute,
      second,
      convertedHour;

String AmorPm, //string var to hold whether it's AM or PM
       currentDayOfWeek; //holds current day of week

String daysOfWeek[7] = {"Sunday","Monday","Tuesday","Wednesday","Thursday","Friday","Saturday"}; //array to hold days of week

//pin declarations along with vars that hold information from MQ-135 and DHT-11 sensors

int dhtPin = 19,
    gasPin = A0,
    gasValue;

String  tempF,
        tempC,
        humidity,
        smallTempF,
        smallTempC,
        smallHum,
        airQuality;

DHT tempSen(dhtPin,DHT11); // creates object for tempature sensor
RTC_DS3231 realTimeClock; //creates object for rtc(real time clock) functionality 

//function prototypes
String getTime();
String getDayOfWeek();
String getTemperature();
String getAirQuality();
String zeroAdder(String x);


//code for UI of webpage//
//constant multiline string to allow us to write clean html code
const char index_html[] PROGMEM = R"rawliteral( 
 <!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>

    <style>

        body{
        
            background-color: rgb(48, 52, 63);
            border: 3px rgb(21,138,0) solid;
            border-radius: 6px;
            width: 500px;
            height: 400px;
            margin: auto;
            margin-top: 50px;
            color: rgb(95,173,86);

        }

        #VL1{

            border-left: 3px solid rgb(21,138,0);
            height: 130px;
            margin-left: 50px;
            margin-bottom: 0px;

        }

        .topSection{
            display: flex;
            flex-direction: row;
            align-items: center;
        }


        .timecontainer{

            display: flex;
            margin: auto;   
            gap: 8px;   
            font-size: 40px;
        
        }

        #VL2{
            border-right: 3px solid rgb(21,138,0);
            height: 130px;
            margin-left:5px;
            margin-bottom: 0px;
            margin-top: 0px;
        }

        .middlePortion{
            margin-top: 0px;
            font-size: 34px;
            display: flex;
            flex-direction: row;
            gap: 8px;
        }

        .temperatureOverlap{
            display: flex;
            flex-direction: row;
            gap: 8px;
            align-items:center;

        }

        .verticleText{
            display: flex;
            flex-direction: column;
            align-items: start;
        }

        .currentDate{

            display: flex;
            flex-direction: row;
            align-items: center;
            gap: 7px;
            font-size: 34px;
            justify-content:center;
            margin: auto;
        }

        .VL2{

            border-right: 1px solid rgb(21,138,0);
            height: 30px;
    
        }

        .bottomSection{
            display: flex;
            flex-direction: row;
            align-items: center;
            gap: 8px;
            font-size:40px;
        }

        #VL3{

            border-left: 3px solid rgb(21,138,0);
            height: 138px;
            margin-left: 10px;
            margin-bottom:0px;


        }

        .airqualOverlap{
            display: flex;
            flex-direction: column;
            align-items:start;
        }

        .lightMode{
            background-color: rgb(255, 255, 255);
            border: 3px rgb(50, 58, 69) solid;
            color: rgb(26, 61, 117);
            
        }

        body.lightMode #VL1{  /*do this to each verticle and horizontal line to change theme, get rid of any inline colors defined in the html to do so play around with it*/
            border-left: 3px solid rgb(50, 58, 69);
        }

        body.lightMode #VL2{
            border-right: 3px solid rgb(50, 58, 69);
        }


        body.lightMode .VL2{
            border-right: 3px solid rgb(50, 58, 69);
        }

        body.lightMode #VL3{
            border-left: 3px solid rgb(50, 58, 69);
        }

        .horizontalLine{
        
            border: rgb(21,138,0);
            height: 2px; 
            background-color: rgb(21,138,0); 
            margin-top: 0; 
            margin-bottom: 0px;

        }

        body.lightMode .horizontalLine{
            background-color: rgb(50, 58, 69);
        }

        .buttonContainer{
            display: flex;
            justify-content: center;
            margin-top: 10px;
        }

        #buttonTheme{
   
            background-color: rgb(48, 52, 63);
            border-color: rgb(21,138,0);
            color: rgb(95,173,86);
            font-size: 16px;
            height: 30px;
            width: 100px;
            text-align: center;
            border-radius: 6px;

        }

        body.lightMode #buttonTheme{
            background-color: rgb(255, 255, 255);
            border-color: rgb(255, 255, 255);
            color: rgb(26, 61, 117);
        }

    </style>

</head>

<body>

    <script>

        function toggleLightMode(){
            var element = document.body;
            element.classList.toggle("lightMode");
        }

        setInterval(() => {

            fetch("/second")
            .then(response => response.text())
            .then (data=>{
                document.getElementById("second").textContent = data;
            });

        },1000);

        setInterval(() => {

            fetch("/hour")
            .then(response => response.text())
            .then (data=>{
                document.getElementById("hour").textContent = data;
            });

        },1000);

        setInterval(() => {

            fetch("/minute")
            .then(response => response.text())
            .then (data=>{
                document.getElementById("minute").textContent = data;
            });

        },1000);

        setInterval(() => {

            fetch("/ampm")
            .then(response => response.text())
            .then (data=>{
                document.getElementById("ampm").textContent = data;
            });

        },1000);

        setInterval(() => {

            fetch("/dayofweek")
            .then(response => response.text())
            .then (data=>{
                document.getElementById("dayofweek").textContent = data;
            });

        },1000);

        setInterval(() => {

            fetch("/month")
            .then(response => response.text())
            .then (data=>{
                document.getElementById("month").textContent = data;
            });

        },1000);

        setInterval(() => {

            fetch("/day")
            .then(response => response.text())
            .then (data=>{
                document.getElementById("day").textContent = data;
            });

        },1000);

        setInterval(() => {

            fetch("/year")
            .then(response => response.text())
            .then (data=>{
                document.getElementById("year").textContent = data;
            });

        },1000);

        setInterval(() => {

            fetch("/tempf")
            .then(response => response.text())
            .then (data=>{
                document.getElementById("tempf").textContent = data;
            });

        },1000);

        setInterval(() => {

            fetch("/tempc")
            .then(response => response.text())
            .then (data=>{
                document.getElementById("tempc").textContent = data;
            });

        },1000);

        setInterval(() => {

            fetch("/humidity")
            .then(response => response.text())
            .then (data=>{
                document.getElementById("humidity").textContent = data;
            });

        },1000);

        setInterval(() => {

            fetch("/airquality")
            .then(response => response.text())
            .then (data=>{
                document.getElementById("airquality").textContent = data;
            });

        },1000);

    </script>

    <div class = "topSection">

      <div style = "margin-left: 50px; font-size: 32px;">

        <p id = "dayofweek">-</p>

      </div>

        <div id = "VL1"></div>

        <div class = "timecontainer">


            <p id = "hour">-</p>

            <p>:</p>

            <p id = "minute">-</p>
            
            <p>:</p>

            <p id = "second">-</p>

            <p id = "ampm" >-</p>


        </div>

    </div>
    
    <hr class = "horizontalLine">


    <div class = "middlePortion">

        <div class = "temperatureOverlap">

            <div class = verticleText>

                <p style="font-size: 24px; margin-left: 5px; margin-bottom: 0px;">Current</p>
                <p style="font-size: 24px; margin-left: 5px; margin-top: 0px;">Temp:</p>
            
            </div>

            <p id = "tempf">-</p>

            <p>°F</p>

            <div class = "VL2"></div>
            
            <p id = "tempc">-</p>
            
            <p>°C</p>

        </div>

        <div id = "VL2"></div>

        <div class = "currentDate">

            <p id = "month">-</p>

            <div class = "VL2"></div>

            <p id = "day">-</p>

            <div class = "VL2"></div>

            <p id = "year" style = "margin-right: 4px;">-</p>

        </div> 

    </div>

    <hr class = "horizontalLine">

    <div class = "bottomSection">

        <p style="font-size: 24px; margin-left: 8px;">Humidity:</p>

        <p id = "humidity">-</p>
        <p>%</p>

        <div id = "VL3"></div>

        <div class = "airqualOverlap">

            <p style="font-size: 24px; margin-bottom: 0px;">Air</p>
            <p style="font-size: 24px; margin-top: 0px;">Quality:</p> 

        </div>

        <p id = "airquality" style="font-size: 40px; margin-left: 5px;">-</p>



    </div>

    <div class="buttonContainer">

        <button id="buttonTheme" onclick="toggleLightMode()">Theme</button>

    </div>
    
</body>

</html>
  )rawliteral";

void displayUI(){ //function that displays/sends UI (html code) to webserver

  server.send_P(200,"text/html",index_html);

}

void setup() {
  Serial.begin(115200);

  tempSen.begin(); //initiates use tempature sensor
  realTimeClock.begin(); // initilizes use rtc module

  //realTimeClock.adjust(DateTime(2025,6,22,12,18,0)); //sets time of module to June 16,2025 and 12:00:00 AM [setting time is based on 24 hours system] (upload once then comment out so RTC module keeps track of time)

  //functions are called in setup() so immidiately upon starting the page inital values are acquired
  getTime();
  getDayOfWeek();
  getTemperature();
  getAirQuality();


  WiFi.softAPConfig(localIP, gatewayIP,subNet); //configures IP settings
  WiFi.softAP(ssid,password); //creates network with given name and password and allows for connections to be established

  server.begin(); //initiates webserver


  server.on("/",displayUI); //server.on with the "/" makes it so as soon as someone enters page it calls display text function

  //This series of server.on() function calls sets up paths which the webpage uses to request
  //the live data that it needs to display, it requires this data via the setInterval() functions defined in the <script> in the html
  //data is fetched and it's sent in to the webpage as a replacement of the temporary variable every second

  server.on("/second",[](){
    
      server.send(200,"text/plain",zeroAdder(second));
  });

  server.on("/hour",[](){
    
      server.send(200,"text/plain",zeroAdder(convertedHour));
  });

  server.on("/minute",[](){
    
      server.send(200,"text/plain",zeroAdder(minute));
  });

  server.on("/ampm",[](){
    
      server.send(200,"text/plain",String(AmorPm));
  });

  server.on("/dayofweek",[](){
    
      server.send(200,"text/plain",String(currentDayOfWeek));
  });

  server.on("/month",[](){
    
      server.send(200,"text/plain",zeroAdder(month));
  });

  server.on("/day",[](){
    
      server.send(200,"text/plain",zeroAdder(day));
  });

  server.on("/year",[](){
    
      server.send(200,"text/plain",zeroAdder(year));
  });

  server.on("/tempf",[](){
    
      server.send(200,"text/plain",smallTempF);
  });

   server.on("/tempc",[](){
    
      server.send(200,"text/plain",smallTempC);
  });

  server.on("/humidity",[](){
    
      server.send(200,"text/plain",smallHum);
  });

   server.on("/airquality",[](){
    
      server.send(200,"text/plain",airQuality);
  });

}

void loop() {
  //functions are called repeatedly for constant data 
  getTime();
  getDayOfWeek();
  getTemperature();
  getAirQuality();

  server.handleClient(); //handles all incoming client requests (people that join the webserver)

}

String getTime(){

  DateTime information = realTimeClock.now(); //retreives current time from rtc module and stores it into information object

  //retrieves and stores necessary imformation from information object using built in functions from RTC library
  year = information.year();
  month = information.month();
  day = information.day();
  hour = information.hour();
  minute = information.minute();
  second = information.second();


  if(hour >= 12){ //if the current hour (based on 24 hour system is >=12) set the clock to PM

    AmorPm = "PM";

    if (hour > 12){ //converts the hour from the 24 hour system to a 12 the 12 hour system and stores it into variable
      convertedHour = (hour - 12);
      
    }

    else if(hour == 12){ //special case when the clock hits the 12'th hour meaning it's 12pm
      convertedHour = 12;
      
    }

  }

  else{ //if the current hour (based on 24 hour system) is < 12 set the clock to AM
    AmorPm = "AM";
    

    if (hour == 0){ //converts the 24 hour system of 0:00:00 to 12 hour system (12:00 AM)
      convertedHour = 12;

    }

    else{ //if given hour is in between 0 and 12 just leave it as is
      convertedHour = hour;
    }

  }


  return (year,month,day,convertedHour,minute,second,AmorPm);
 
}

String getDayOfWeek(){

  DateTime information = realTimeClock.now(); //retreives current time from rtc module and stores it into information object

  currentDOW = information.dayOfTheWeek();

  currentDayOfWeek = daysOfWeek[currentDOW]; //stores day of week into string using array and index of it

  return currentDayOfWeek;

}

String getTemperature(){

  tempF = tempSen.readTemperature(true); //reads and stores the tempature in degrees Farenheit
  tempC = tempSen.readTemperature();    //reads and stores the tempature in degrees Celcius
  humidity = tempSen.readHumidity();    //reads and stores the humidity

  //substring method is used to trim the tempature and humidity readings
  smallTempF = tempF.substring(0,4);
  smallTempC = tempC.substring(0,4);
  smallHum =  humidity.substring(0,2);

  return(smallTempF,smallTempC,smallHum);

}

//function that retrieves air quality from MQ-135 and returns it to user
String getAirQuality(){

  gasValue = analogRead(gasPin); //Reads analog pin connected to sensor (0-1023); clean air = lower analog value, polluted air = high analog value


  if (gasValue <= 250){

      airQuality = "Good";
      
    }

  else if (gasValue <=450){

      airQuality = "Moderate";
    
    }

  else{
      airQuality = "Poor";
    }

  return(airQuality);

}

String zeroAdder(String x){ //function that "adds" a 0 for eligiblity purposes

  if (x.length() == 1){ //if given number is 1 charecter add a zero to front of the it
    return "0"+x;
  }
  else{

    return(x); //returns x with the 0 now in front

  }

}





