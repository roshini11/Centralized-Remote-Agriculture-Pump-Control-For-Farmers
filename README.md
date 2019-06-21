# Centralized-Remote-Agriculture-Pump-Control-For-Farmers
#include <ESP8266WiFi.h>
#include <PubSubClient.h>
#include <Servo.h>

Servo myservo; 
const char* ssid = "SB-IOT1";
const char* password = "sb@iot11";

#define LED D0
#define ORG "2crxye"
#define DEVICE_TYPE "rnodemcu"
#define DEVICE_ID "12345"
#define TOKEN "Qm9tTY?CEsHyWip665"
String command;

char server[] = ORG ".messaging.internetofthings.ibmcloud.com";
char topic[] = "iot-2/cmd/home/fmt/String";
char authMethod[] = "use-token-auth";
char token[] = TOKEN;
char clientId[] = "d:" ORG ":" DEVICE_TYPE ":" DEVICE_ID;
//Serial.println(clientID);
void callback(char* topic, byte* payload, unsigned int payloadLength);
WiFiClient wifiClient;
//void callback(char* topic, byte* payload, unsigned int payloadLength);
PubSubClient client(server, 1883, callback, wifiClient);
int pos;
int a=0;
int sensor_pin=A0;
int moisture;
void setup()
{myservo.attach(D2); 
   Serial.begin(115200);
  Serial.println();
  //pinMode(D1,OUTPUT);
   pinMode(D0,OUTPUT);
   // Serial.begin(9600);

   //Serial.println("Reading From the Sensor ...");

   //delay(2000);
    wifiConnect();
  mqttConnect();
}

void loop() {
  if(a==0)
  {
   moisture = analogRead(sensor_pin);


   Serial.print("Moisture : ");

   Serial.print(moisture);

  if(moisture<800)
  {
    Serial.println("wet");
   digitalWrite(D0,LOW);
  for (pos = 180; pos >= 0; pos -= 1) { // goes from 180 degrees to 0 degrees
    myservo.write(pos);              // tell servo to go to position in variable 'pos'
    delay(15);                    // waits 15ms for the servo to reach the position
  }
  }   
   else
   {
    Serial.println("dry");
    digitalWrite(D0,HIGH);
       for (pos = 0; pos <= 180; pos += 1) { // goes from 0 degrees to 180 degrees
    // in steps of 1 degree
    myservo.write(pos);              // tell servo to go to position in variable 'pos'
    delay(15);                   // waits 15ms for the servo to reach the position
  
   }
}
  }
  if (!client.loop()) {
    mqttConnect();
  }
delay(100);


}

void wifiConnect() {
  Serial.print("Connecting to "); Serial.print(ssid);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
  Serial.print("nWiFi connected, IP address: "); Serial.println(WiFi.localIP());
}

void mqttConnect() {
  if (!client.connected()) {
    Serial.print("Reconnecting MQTT client to "); Serial.println(server);
    while (!client.connect(clientId, authMethod, token)) {
      Serial.print(".");
      delay(500);
    }
    initManagedDevice();
    Serial.println();
  }
}
void initManagedDevice() {
  if (client.subscribe(topic)) {
    Serial.println("subscribe to cmd OK");
  } else {
    Serial.println("subscribe to cmd FAILED");
  }
}


void callback(char* topic, byte* payload, unsigned int payloadLength) {
  Serial.print("callback invoked for topic: "); Serial.println(topic);
  for (int i = 0; i < payloadLength; i++) {
    //Serial.println((char)payload[i]);
    command += (char)payload[i];
  }
Serial.println(command);
control_func();
command="";
  }
  

void PublishData(int moisture){
 if (!!!client.connected()) {
 Serial.print("Reconnecting client to ");
 Serial.println(server);
 while (!!!client.connect(clientId, authMethod, token)) {
 Serial.print(".");
 delay(500);
 }
 Serial.println();
 }
  String payload = "{\"d\":{\"moisture\":";
  payload += moisture;
 
  payload += "}}";
 Serial.print("Sending payload: ");
 Serial.println(payload);
  
 if (client.publish(topic, (char*) payload.c_str())) {
 Serial.println("Publish ok");
 } else {
 Serial.println("Publish failed");
 }
}
void control_func()
{
 if(command=="TURN ON")
  {
       digitalWrite(D0,HIGH);
       for (pos = 0; pos <= 180; pos += 1) { // goes from 0 degrees to 180 degrees
    // in steps of 1 degree
    myservo.write(pos);              // tell servo to go to position in variable 'pos'
    delay(15);
    a++;    
   // waits 15ms for the servo to reach the position
  }
  }
  else if(command=="TURN OFF")
  {
    digitalWrite(D0,LOW);
  for (pos = 180; pos >= 0; pos -= 1) { // goes from 180 degrees to 0 degrees
    myservo.write(pos);              // tell servo to go to position in variable 'pos'
    delay(15);
    a++;// waits 15ms for the servo to reach the position
  }
  }
  else if(command=="AUTOMATIC")
  {
    a=0;
  }
  delay(1000);
  
}  
