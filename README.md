# IOT-APPROACH-TO-SAVE-LIFE-USING-GPS-FOR-THE-TRAVELER-DURING-ACCIDENT
#include "DHT.h"
#define DHTPIN 2    
#define DHTTYPE DHT11 
DHT dht(DHTPIN, DHTTYPE);
   
int sensor;           //Variable to store analog value (0-1023)

const int xpin = A0;
const int ypin = A1;
const int zpin = A2;

#include <SoftwareSerial.h>

SoftwareSerial SIM900A(10,11); 

#include <TinyGPS++.h>
#include <SoftwareSerial.h>

static const int RXPin = 4, TXPin = 3;
static const uint32_t GPSBaud = 9600;

TinyGPSPlus gps;

SoftwareSerial ss(RXPin, TXPin);

void setup()
{
  Serial.begin(9600);
  Serial.println("DHT11 test!");
    dht.begin();
    
  Serial.begin(9600);      //Only for debugging
    Serial.begin(9600);  
    
  SIM900A.begin(9600);   // Setting the baud rate of GSM Module  
  Serial.begin(9600);    // Setting the baud rate of Serial Monitor (Arduino)
  Serial.println ("SIM900A Ready");
  delay(100);
  Serial.println ("Type s to send message or r to receive message"); 
  
  Serial.begin(9600);
  ss.begin(GPSBaud);

  Serial.println(F("Device GPS .ino"));
  Serial.println(F("A simple demonstration of TinyGPS++ with an attached GPS module"));
  Serial.print(F("Testing TinyGPS++ library v. "));
  
  Serial.println(F("satyanarayana"));
  Serial.println();
 
}



void loop()
{
  // This sketch displays information every time a new sentence is correctly encoded.
  while (ss.available() > 0)
    if (gps.encode(ss.read()))
      displayInfo();

  if (millis() > 5000 && gps.charsProcessed() < 10)
  {
    Serial.println(F("No GPS detected: check wiring."));
    while(true);
  }
  
  
  // Wait a few seconds between measurements.
  delay(2000);
  // Reading temperature or humidity takes about 250 milliseconds!
  // Sensor readings may also be up to 2 seconds 'old' (its a very slow sensor)
  float h = dht.readHumidity();
  // Read temperature as Celsius (the default)
  float t = dht.readTemperature();
  // Read temperature as Fahrenheit (isFahrenheit = true)
  float f = dht.readTemperature(true);
  // Check if any reads failed and exit early (to try again).
  if (isnan(h) || isnan(t) || isnan(f)) {
    Serial.println("Failed to read from DHT sensor!");
    return;
  }
  // Compute heat index in Fahrenheit (the default)
  float hif = dht.computeHeatIndex(f, h);
  // Compute heat index in Celsius (isFahreheit = false)
  float hic = dht.computeHeatIndex(t, h, false);

  Serial.print("Humidity: ");
  Serial.print(h);
  Serial.print(" %\t");
  Serial.print("Temperature: ");
  Serial.print(t);
  Serial.print(" *C ");
  Serial.print(f);
  Serial.print(" *F\t");
  Serial.print("Heat index: ");
  Serial.print(hic);
  Serial.print(" *C ");
  Serial.print(hif);
  Serial.println(" *F");
  
  

  
  sensor = analogRead(A0);
Serial.println(sensor);

int x = analogRead(xpin); 
delay(50);
int y = analogRead(ypin);
delay(50);
int z = analogRead(zpin);
delay(50);

Serial.print(x-322);
Serial.print("\t");
Serial.print(y-320);
Serial.print("\t");
Serial.print(z-263);
Serial.print("\n"); //While sensor is not moving, analog pin receive 1023~1024 value

 
 if (Serial.available()>0)
   switch(Serial.read())
  {
    case 's':
      SendMessage();
      break;
    case 'r':
      RecieveMessage();
      break;
  }

 if (SIM900A.available()>0)
   Serial.write(SIM900A.read());
   
  if (sensor>340)
  {
        Serial.print("Vibration is Occured ");
        SendMessage();
  }
  
  else{ 
        Serial.print("NO Vibration  ");
        Serial.print("\n");
  }

 if
 (h>95.00||t>35.00||f>91.00||hic>46.00||hif>114.00)
 {
  Serial.print("tempreature is high\n");
  SendMessage();
  Serial.print("\n");
 }
  else{ 
        Serial.print("NO tempreature is high ");
        Serial.print("\n");
  }
  
 if
 (x>65||y>374||z>387)
{
 Serial.print("Accelometer Changes\n ");
 SendMessage();
    Serial.print("\n");
 }
  else{ 
       Serial.print("NO changes in accelometer  ");
        Serial.print("\n");
 }
  
delay(50); //Small delay
}


 void SendMessage()
{
  Serial.println ("Sending Message");
  SIM900A.println("AT+CMGF=1");    //Sets the GSM Module in Text Mode
  delay(1000);
  Serial.println ("Set SMS Number");
  SIM900A.println("AT+CMGS=\"+917286970883\"\r"); //Mobile phone number to send message
  delay(1000);
  Serial.println ("Set SMS Content");
  SIM900A.println("Accident occured detected at ");// Messsage content
  delay(50);
 
  SIM900A.print("http://maps.google.com/?q=loc:");
  SIM900A.print(gps.location.lat(), 6);
  SIM900A.print(",");
  SIM900A.println(gps.location.lng(), 6);

 Serial.println ("Finish");  
  SIM900A.println((char)26);// ASCII code of CTRL+Z
  delay(1000);
  Serial.println ("Message has been sent ->SMS ");
}


 void RecieveMessage()
{
  Serial.println ("SIM900A receive SMS");
  delay (1000);
  SIM900A.println("AT+CNMI=2,2,0,0,0"); // AT Command to receive a live SMS
  delay(1000);
  Serial.write ("Unread Message done");
 }

void displayInfo()
{
  Serial.print(F("Location: ")); 
  if (gps.location.isValid())
  {
    Serial.print(gps.location.lat(), 6);
    Serial.print(F(","));
    Serial.print(gps.location.lng(), 6);
  }
  else
  {
    Serial.print(F("INVALID"));
  }

  Serial.print(F("  Date/Time: "));
  if (gps.date.isValid())
  {
    Serial.print(gps.date.month());
    Serial.print(F("/"));
    Serial.print(gps.date.day());
    Serial.print(F("/"));
    Serial.print(gps.date.year());
  }
  else
  {
    Serial.print(F("INVALID"));
  }

  Serial.print(F(" "));
  if (gps.time.isValid())
  {
    if (gps.time.hour() < 10) Serial.print(F("0"));
    Serial.print(gps.time.hour());
    Serial.print(F(":"));
    if (gps.time.minute() < 10) Serial.print(F("0"));
    Serial.print(gps.time.minute());
    Serial.print(F(":"));
    if (gps.time.second() < 10) Serial.print(F("0"));
    Serial.print(gps.time.second());
    Serial.print(F("."));
    if (gps.time.centisecond() < 10) Serial.print(F("0"));
    Serial.print(gps.time.centisecond());
  }
  else
  {
    Serial.print(F("INVALID"));
  }

  Serial.println();
}
