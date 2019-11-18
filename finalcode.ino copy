#define BLYNK_PRINT Serial
#define EspSerial Serial1             // Hardware Serial on Mega
#define ESP8266_BAUD 115200           // ESP8266 baud rate

#include <ESP8266_Lib.h>
#include <BlynkSimpleShieldEsp8266.h>
#include <TimeLib.h>
#include <Adafruit_NeoPixel.h>
#include <Wire.h>
#include <Servo.h>

//Light strips
Adafruit_NeoPixel uv = Adafruit_NeoPixel(8, 8, NEO_GRB + NEO_KHZ800);
Adafruit_NeoPixel rgb = Adafruit_NeoPixel(15, 9, NEO_GRB + NEO_KHZ800);

//Sensors
#define TOTAL_CIRCUITS 2              // Set how many I2C circuits are attached to the Tentacle shield
char sensordata[30];                  // A 30 byte character array to hold incoming data from the sensors
byte sensor_bytes_received = 0;       // We need to know how many characters bytes have been received
byte code = 0;                        // used to hold the I2C response code.
byte in_char = 0;                     // used as a 1 byte buffer to store in bound bytes from the I2C Circuit.
int channel_ids[] = {99, 102};
char *channel_names[] = {"PH", "RTD"};

//Servo
Servo myservo;  // create servo object to control a servo
int pos = 0;    // variable to store the servo position

//Reading user input
WidgetTerminal terminal(V1);
int len;
char buffer[32];
int opMode = 0;

//Wifi
ESP8266 wifi(&EspSerial);
BlynkTimer timer;
char auth[] = "0eb32d72ee5546d791548ab0b72d3326";
char ssid[] = "ATT9gcn24B";
char pass[] = "9c+2vak?rne6";

//char auth[] = "bc8c19c587d5441eb4e736028e70da01";
//char ssid[] = "gnomedome";
//char pass[] = "dannydevito";

//Use this for HotSpot Wifi and comment out the ssid and pass above
//char ssid[] = "Samsung Galaxy S8 1609";
//char pass[] = "10101010";

//Tank variables
unsigned long startMilli;
unsigned long demoDay = 300000;
unsigned long demoHour = 12500;
unsigned long demo10Min = 2083;
unsigned long millisPerHour = 3600000;
unsigned long millisPer10min = 600000;
int hours = 24;
int fInterval = 0;
int uvInterval = 0;
int rgbInterval = 0;

void setup (void) {
  Serial.begin (9600);
  EspSerial.begin(ESP8266_BAUD);
  delay(10);
  Blynk.begin(auth, wifi, ssid, pass);
  screenClear();
  instructions();
  Wire.begin();
  uv.begin();
  uv.setBrightness(200);
  uv.show();
  rgb.begin();
  rgb.setBrightness(50);
  rgb.show();
  timer.setInterval(6000L, myTimerEvent);
  myservo.attach(52);
  myservo.write(1);
  startMilli = millis();
}

void loop (void) {
  Blynk.run();
  timer.run();
  Serial.flush();
  unsigned long runMilli = millis();

  //Standard Mode
  if (opMode == 2) {
    //Feed
    if ((runMilli - startMilli) >= (hours / fInterval)) {
      rotate();
      startMilli = runMilli;
      delay(3);
    }
    //UV lights on
    if (runMilli <= (uvInterval * millisPerHour)) {
      colorWipeUV(uv.Color(255, 255, 255), 50);
      terminal.flush();
      Serial.flush();
    }
    //UV lights off
    else {
      colorWipeUV(uv.Color(0, 0, 0), 50);
      terminal.flush();
      Serial.flush();
    }
    //RGB lights on
    if (runMilli <= (rgbInterval * millisPerHour)) {
      //rgb lights on
      rainbowRGB(20);
      terminal.flush();
      Serial.flush();
    }
    //RGB lights off
    else {
      colorWipeRGB(rgb.Color(0, 0, 0), 50);
      terminal.flush();
      Serial.flush();
    }
  }

  //Demo Mode
  if (opMode == 1) {
    //Feed
    if ((runMilli - startMilli) >= (demoDay / fInterval)) {
      rotate();
      startMilli = runMilli;
      delay(3);
    }
    //UV lights on
    if (runMilli <= (uvInterval * demoHour)) {
      //uv lights on
      colorWipeUV(uv.Color(255, 255, 255), 50);
    }
    //UV lights off
    else {
      colorWipeUV(uv.Color(0, 0, 0), 50);
    }
    //RGB lights on
    if (runMilli <= (rgbInterval * demoHour)) {
      //rgb lights on
      rainbowRGB(20);
    }
    //RGB lights off
    else {
      colorWipeRGB(rgb.Color(0, 0, 0), 50);
    }
    //Exits demo
    if (runMilli >= demoDay) {
      rotate();
      endDemo();
    }
  }
}

//Ends demo and switches to idle mode
void endDemo() {
  terminal.println("Hey, its been a day!");
  terminal.println("End of demo reached. Device will need restart.");
  terminal.flush();
  opMode = 0;
}

//Timed event to read data from both sensors
void myTimerEvent() {
  sensorCycle();
}

//Displays operation instructions
void instructions() {
  terminal.println("Use the top slider to select operating mode. 1 is the demo. 2 is standard.");
  terminal.println("Use the sliders in the middle to control how often to feed and leave the lights on.");
  terminal.println("Important messages will be displayed in this terminal.");
  terminal.flush();
}

//Clears terminal screen to remove old text
void screenClear() {
  for (int i = 0; i <= 25; i++) {
    terminal.println("");
  }
  terminal.flush();
}

//Rotates servo
void rotate() {
  terminal.println("Dispensing food.");
  terminal.flush();
  Serial.flush();
  myservo.write(1);
  for (pos = 1; pos <= 180; pos += 1) { // goes from 1 degrees to 180 degrees
    myservo.write(pos);              // tell servo to go to position in variable 'pos'
    delay(5);                       // waits 5ms for the servo to reach the position
  }
  //Serial.flush();
  for (pos = 180; pos >= 1; pos -= 1) { // goes from 180 degrees to 1 degrees
    myservo.write(pos);              // tell servo to go to position in variable 'pos'
    delay(5);                       // waits 5ms for the servo to reach the position
  }
}

//Reads from the sensors
void sensorCycle() {
  for (int channel = 0; channel < TOTAL_CIRCUITS; channel++) {         // loop through all the sensors
    Wire.beginTransmission(channel_ids[channel]);                    // call the circuit by its ID number.
    Wire.write('r');                                                 // request a reading by sending 'r'
    Wire.endTransmission();                                          // end the I2C data transmission.
    Wire.flush();
    Serial.flush();
    delay(1000);                                                     // AS circuits need a 1 second before the reading is ready
    sensor_bytes_received = 0;                                       // reset data counter
    memset(sensordata, 0, sizeof(sensordata));                       // clear sensordata array;
    Wire.requestFrom(channel_ids[channel], 8, 1);                    // call the circuit and request 8 bytes
    code = Wire.read();

    for (int bytes = 8; Wire.available() <= bytes; bytes++) {        // are there bytes to receive?
      in_char = Wire.read();                                         // receive a byte.
      if (in_char == 0 || (Wire.peek() == '\n') || (Wire.peek() == '\r')) {
        Wire.endTransmission();                                      // end the I2C data transmission.
        Serial.flush();
        break;                                                       // exit the while loop, we're done here
      }
      else {
        sensordata[sensor_bytes_received] = in_char;                 // append this byte to the sensor data array.
        sensor_bytes_received++;
      }
    }

    Serial.flush();
    switch (code) {                             // switch case based on what the response code is.
      case 1:                                   // decimal 1  means the command was successful.
        if (channel == 1) {
          int dataDegF = atoi(sensordata);
          int degF = (dataDegF * (1.8)) + 32;
          readRTD(degF);
          delay(1000);
          Serial.flush();
        }

        if (channel == 0) {
          readPH(sensordata);
          delay(1000);
          Serial.flush();
        }
        break;                                  // exits the switch case.

      case 2:                                   // decimal 2 means the command has failed.
        terminal.println("command failed");       // print the error
        break;                                  // exits the switch case.

      case 254:                                 // decimal 254  means the command has not yet been finished calculating.
        terminal.println("circuit not ready");    // print the error
        break;                                  // exits the switch case.

      case 255:                                 // decimal 255 means there is no further data to send.
        terminal.println("no data");              // print the error
        break;                                  // exits the switch case.
    }
  }
}

//Takes user input for feeding data and displays it
BLYNK_WRITE(V2) {
  fInterval = param.asInt();
  terminal.print("\nCurrently feeding ");
  terminal.print(fInterval);
  terminal.print(" times a day.\n");
  terminal.flush();
  Serial.flush();
}

//Takes user input for UV light data and displays it
BLYNK_WRITE(V3) {
  uvInterval = param.asInt();
  terminal.print("\nUV lights are on for ");
  terminal.print(uvInterval);
  terminal.print(" hours.\n");
  terminal.flush();
  Serial.flush();
}

//Takes user input for RGB light data and displays it
BLYNK_WRITE(V4) {
  rgbInterval = param.asInt();
  terminal.print("\nRGB lights are on for ");
  terminal.print(rgbInterval);
  terminal.print(" hours.\n");
  terminal.flush();
  Serial.flush();
}

//Takes user input for operation mode and displays it
BLYNK_WRITE(V5) {
  opMode = param.asInt();
  if (opMode == 0) {
    terminal.println("Entering idle mode.");
  }
  if (opMode == 1) {
    terminal.println("Entering demo mode.");
  }
  if (opMode == 2) {
    terminal.println("Entering standard mode.");
  }
  terminal.flush();
  Serial.flush();
}

//Prints temperature data, warnings are based on coldwater environment demands
void readRTD(int data) {
  int Tdata = data;
  Blynk.virtualWrite(V6, data);
  Serial.flush();
  //Temp is too low
  if (Tdata < 68) {
    int cold = 68 - Tdata;
    terminal.print("Temperature is below the minimum recommended value by ");
    terminal.print(cold);
    terminal.print(" degrees F.\n");
    terminal.flush();
  }
  //Temp is too high
  if (Tdata > 72) {
    int hot = Tdata - 72;
    terminal.print("Temperature is above the maximum recommended value by ");
    terminal.print(hot);
    terminal.print(" degrees F.\n");
    terminal.flush();
  }
}

//Prints pH data, warnings are based on aquarium pH demands
void readPH(char data[]) {
  int Pdata = atoi(data);
  Blynk.virtualWrite(V7, data);
  Serial.flush();
  //pH too low
  if (Pdata < 6.8) {
    int low = 6.8 - Pdata;
    terminal.print("pH is below the minimum recommended value by ");
    terminal.print(low);
    terminal.print(".\n");
    terminal.flush();
  }
  //pH too high
  if (Pdata > 7.2) {
    int high = Pdata - 7.2;
    terminal.print("pH is above the maximum recommended value by ");
    terminal.print(high);
    terminal.print(".\n");
    terminal.flush();
  }
}

//Cycles through color wheel
uint32_t Wheel(byte WheelPos) {
  WheelPos = 255 - WheelPos;
  if (WheelPos < 85) {
    return rgb.Color(255 - WheelPos * 3, 0, WheelPos * 3);
  }
  if (WheelPos < 170) {
    WheelPos -= 85;
    return rgb.Color(0, WheelPos * 3, 255 - WheelPos * 3);
  }
  WheelPos -= 170;
  return rgb.Color(WheelPos * 3, 255 - WheelPos * 3, 0);
}

//Sets uv strip to a solid color
void colorWipeUV(uint32_t c, uint8_t wait) {
  for (uint16_t i = 0; i < uv.numPixels(); i++) {
    uv.setPixelColor(i, c);
    uv.show();
    delay(wait);
  }
}

//Sets rgb strip to a solid color
void colorWipeRGB(uint32_t c, uint8_t wait) {
  for (uint16_t i = 0; i < rgb.numPixels(); i++) {
    rgb.setPixelColor(i, c);
    rgb.show();
    delay(wait);
  }
}

//Sets rgb strip to rainbow color pattern
void rainbowRGB(uint8_t wait) {
  uint16_t i, j;

  for (j = 0; j < 256; j++) {
    for (i = 0; i < rgb.numPixels(); i++) {
      rgb.setPixelColor(i, Wheel((i + j) & 255));
    }
    rgb.show();
    delay(wait);
  }
}
