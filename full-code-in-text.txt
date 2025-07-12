// Define Blynk template ID
#define BLYNK_TEMPLATE_ID "TMPL6UhKuwHUp"
// Define Blynk template name
#define BLYNK_TEMPLATE_NAME "cmpe331 proje"
// Define Blynk authorization token
#define BLYNK_AUTH_TOKEN "nwzHfmtew3ngJKBauxuDkzmgSbZjgbPl"
#define BLYNK_PRINT Serial

// Include necessary libraries
#include <ESP8266_Lib.h>
#include <BlynkSimpleShieldEsp8266.h>
#include <SoftwareSerial.h>
#include <Servo.h>

// Define WiFi credentials
char ssid[] = "TheSenate";     //SSID of the wi-fi network you want ESP module to connect to 
char pass[] = "mustafa_deniz"; //password of the wi-fi network you want ESP module to connect to

// Define software serial for ESP
SoftwareSerial EspSerial(2, 3); // The RX, TX nodes on the ESP module are connected to 
                                // digital pins 2, 3 on Arduino UNO
// Define ESP8266 baud rate
#define ESP8266_BAUD 38400

// Initialize ESP8266
ESP8266 wifi(&EspSerial);

// Define servo objects
Servo servo1;
Servo servo2;

// Blynk function to send the input from the app to servo1
BLYNK_WRITE(V1) {
  int ServoState = param.asInt();  // param değişkeni burada kullanılabilir
  controlServo(V1, servo1, ServoState);  // ServoState'i fonksiyona geçirin
}

// Blynk function to send the input from the app to servo2
BLYNK_WRITE(V2) {
  int ServoState = param.asInt();
  controlServo(V2, servo2, ServoState);
}

// Custom function to control servos based on value from the app
void controlServo(int virtualPin, Servo servo, int ServoState) {  
  if (ServoState == 1) {
    servo.writeMicroseconds(2000);    //Here, if we send 1 (meaning on switch) from the app, servos will rotate in a direction.
    delay(167);                       //This delay function determines how long do you want servo to rotate. delay(167) corresponds to 90 degrees in real world.
    servo.writeMicroseconds(1500);
  } else {                            //If the input is 0(meaning off switch) we turn the servo back to its original position.
    servo.writeMicroseconds(1000);
    delay(167);
    servo.writeMicroseconds(1500);
  }
}

// Define sensor pins
int lm35_1_Pin = A0;
int lm35_2_Pin = A1; 
int smokeA3 = A3;
int data = 0;
int sensorThres = 100;

// Define Blynk timer
BlynkTimer timer;

// Function to send sensor data to the app
void sendSensorData() {
  int data = analogRead(smokeA3); 
  Blynk.virtualWrite(V0, data);   //Sending data from gas sensor

  //The notification function for the gas sensor. 500 is the critical value for sending a notification
  if(data>500){
    Blynk.logEvent("gas_leak_1", "GAS LEAK DETECTED");
  }

  float temperature1 = readTemperature(lm35_1_Pin);
  if (temperature1 != -1.0) {             //For more information about this if statement, check the readTemperature code below
    Blynk.virtualWrite(V3, temperature1); //Sending data from temperature sensor
  }
  delay(400);

  float temperature2 = readTemperature(lm35_2_Pin);
  if (temperature2 != -1.0) {
    Blynk.virtualWrite(V4, temperature2);
  }

  //The notification function for the temperature sensor. 50 is the critical value for sending a notification
  if(temperature1 > 50){
    Blynk.logEvent("fire_1", "POSSIBLE FIRE IN BEDROOM");
  }
  if(temperature2 > 50){
    Blynk.logEvent("fire_2", "POSSIBLE FIRE IN LIVING ROOM");
  }

  delay(400);
}

// Custom function to read temperature from sensors
float readTemperature(int pin) {
  int value = analogRead(pin);
  float voltage = value * (5.0 / 1023.0);  // Convert the analog reading to voltage
  float temperature = voltage * 100;  // Convert the voltage to temperature

  if (temperature < 3.0 || temperature > 70.0) {   //In order to prevent the sensor from sending some values that are inconsistent with reality, the function only accepts temperature between certain values
    return -1.0;  // Return an invalid temperature
  } else {
    return temperature;
  }
}

// Setup function
void setup()
{
  pinMode(smokeA3, INPUT);
  Serial.begin(115200);
  EspSerial.begin(ESP8266_BAUD);
  delay(10);
  servo1.attach(13); //Servos attached to digital pins 12 and 13
  servo2.attach(12); 
  Blynk.begin(BLYNK_AUTH_TOKEN, wifi, ssid, pass);
  timer.setInterval(2500L, sendSensorData);
}

// Main loop function
void loop() 
{
  Blynk.run();
  timer.run();
}