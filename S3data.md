# ESP32S3SuperMini Documentation

## Table of Contents

1. [Introduction](#introduction)
2. [Hardware Description](#hardware-description)
   - [Product Specifications](#product-specifications)
   - [Pinout Diagram](#pinout-diagram)
   - [Dimensions](#dimensions)
   - [Schematic](#schematic)
3. [External Power Supply](#external-power-supply)
4. [WiFi Antenna](#wifi-antenna)
5. [Getting Started](#getting-started)
   - [Hardware Setup](#hardware-setup)
   - [Software Setup](#software-setup)
   - [Blinking LED](#blinking-led)
   - [WiFi Control LED](#wifi-control-led)
6. [WiFi Usage](#wifi-usage)
   - [Scan WiFi Networks (Station Mode)](#scan-wifi-networks-station-mode)
   - [Connect to WiFi Network](#connect-to-wifi-network)
   - [WiFi Hotspot (SoftAP)](#wifi-hotspot)
7. [Bluetooth Usage](#bluetooth-usage)
   - [Scan Bluetooth Devices](#scan-bluetooth-devices)
   - [As Bluetooth Server](#as-bluetooth-server)
8. [ChatGPT Integration](#chatgpt-integration)
   - [Overview](#overview)
   - [Prerequisites](#prerequisites)
   - [Configure ESP32S3 to Connect to Network](#configure-esp32s3-to-connect-to-network)
   - [Build Embedded Web Page](#build-embedded-web-page)
   - [Submit Questions to ChatGPT](#submit-questions-to-chatgpt)
   - [Get Answers from ChatGPT](#get-answers-from-chatgpt)
9. [Pin Usage](#pin-usage)
   - [Digital Pins](#digital-pins)
   - [Digital PWM](#digital-pwm)
   - [Analog Pins](#analog-pins)
   - [Serial Communication](#serial-communication)
     - [Hardware Serial](#hardware-serial)
     - [Software Serial](#software-serial)
10. [Micropython](#micropython)
11. [Learning Resources](#learning-resources)
12. [Troubleshooting](#troubleshooting)
13. [Purchase Links](#purchase-links)

---

## Introduction

ESP32S3SuperMini is a mini IoT development board based on the Espressif ESP32-S3 WiFi/Bluetooth dual-mode chip. The ESP32-S3 is a 32-bit RISC-V CPU with FPU (floating-point unit) capable of 32-bit single-precision operations, offering strong computational capabilities. It has excellent RF performance, supporting IEEE 802.11 b/g/n WiFi and Bluetooth 5 (LE) protocol. The board comes with an external antenna to enhance signal strength for wireless applications. It features a compact and exquisite design with a single-sided surface mount structure. It features rich interfaces including 11 digital I/O pins usable as PWM and 4 analog I/O pins usable as ADC. It supports four serial interfaces: UART, I2C, and SPI. The board also includes a small reset button and a bootloader mode button.

In summary, ESP32S3SuperMini is a high-performance, low-power, cost-effective IoT mini development board suitable for low-power IoT applications and wireless wearable devices.

![ESP32S3-SuperMini](data/esp32/esp32s3supermini/esp32s3.jpg)

---

## Hardware Description

### Product Specifications

- **CPU:** ESP32-S3, 32-bit RISC-V single-core processor, clock speed up to 160 MHz
- **WiFi:** 802.11 b/g/n protocol, 2.4 GHz, supports Station mode, SoftAP mode, SoftAP+Station mode, promiscuous mode
- **Bluetooth:** Bluetooth 5.0
- **Ultra-low power:** Deep sleep current approximately 43μA
- **Board resources:** 400KB SRAM, 384KB ROM, built-in 4M flash
- **Chip model:** ESP32S3FH4R2
- **Ultra-small size:** Finger-sized (22.52 x 18mm) classic design, suitable for wearable devices and small projects
- **Security:** Hardware accelerator supporting AES-128/256, hash, RSA, HMAC, digital signatures, and secure boot encryption
- **Interfaces:** 1x I2C, 1x SPI, 2x UART, 11x GPIO (PWM), 4x ADC
- **Onboard RGB and blue LED:** Shared on GPIO48 pin

### Pinout Diagram

![Arduino ESP32S3 Dev Module Pinout](data/esp32/esp32s3supermini/esp32s3foot1.jpg)

*Arduino ESP32S3 Dev Module Pinout*

### Dimensions

![Dimensions](data/esp32/esp32s3supermini/dimension.jpg)

### Schematic

![Schematic](data/esp32/esp32s3supermini/1.png)

---

## External Power Supply

For external power supply, connect the positive terminal to B+ on the back of the board and the negative terminal to B- on the back (supports 3.3-6V power supply). USB can be used for battery charging.

> **Note:** Please be careful when soldering to avoid shorting the positive and negative poles, which may damage the battery and device.

---

## WiFi Antenna

If you want to use an external antenna, you can attach an external antenna according to the following picture:

![External Antenna Connection](data/esp32/esp32s3supermini/esp32s31.png)

---

## Getting Started

### Hardware Setup

You need the following items:

- 1 x ESP32S3SuperMini
- 1 computer
- 1 USB Type-C data cable

> **Tip:** Some USB cables can only provide power, not data transfer. If you don't have a USB cable or are unsure whether your USB cable can transfer data, you can purchase a [Type-C data cable](https://item.taobao.com/item.htm?spm=a1z10.3-c-s.w4002-24438210134.9.24866ea30WLxAl&id=679700862802).

**Step 1:** Connect the ESP32S3SuperMini to your computer using a USB Type-C data cable.

![USB Connection](data/esp32/esp32s3supermini/usbconnect.jpg)

### Software Setup

**Step 1:** Download and install the latest version of Arduino IDE based on your operating system.

![Arduino IDE Download](data/arduino/other/ArduinoIDE.png)

If the download is slow, you can download from the Arduino community site in China [ArduinoIDE download address](https://arduino.me/download).

**Step 2:** Start the Arduino application.

**Step 3:** Add the ESP32 board package to Arduino IDE.

Navigate to **File > Preferences**, then enter the following URL in "Additional Boards Manager URL":

```
https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
```

![Add Board URL](data/arduino/other/add_board.png)

Navigate to **Tools > Board > Boards Manager...**, enter "esp32" in the search box, and install the latest version of ESP32.

![Install ESP32 Board](data/esp32/esp32s3supermini/add_esp32s3.png)

Navigate to **Tools > Board > ESP32 Arduino** and select **"ESP32s3 Dev Module"**. The board list is quite long; you need to scroll to the bottom to find it.

![Select Board](data/esp32/esp32s3supermini/selectboard.jpg)

Navigate to **Tools > Port** and select the serial port name of the connected ESP32S3SuperMini. This may be COM3 or higher (COM1 and COM2 are typically reserved for hardware serial ports).

### Blinking LED

**Step 1:** Copy the following code into Arduino IDE:

```cpp
// define led according to pin diagram
int led = 48;

void setup() {
  // initialize digital pin led as an output
  pinMode(led, OUTPUT);
}

void loop() {
  digitalWrite(led, HIGH);   // turn the LED off
  delay(1000);               // wait for a second
  digitalWrite(led, LOW);    // turn the LED on
  delay(1000);               // wait for a second
}
```

After uploading, you will see the LED on the board blinking with a 1-second delay between each blink.

### WiFi Control LED

A simple web server that lets you blink an LED via the web. From the serial monitor, you can see the IP address of the ESP32S3SuperMini. Open that address in a web browser to turn the LED on and off.

```cpp
#include <WiFi.h>

const char* ssid     = "WIFI_NAME";  // Set WiFi name
const char* password = "WIFI_PASSWORD";  // Set WiFi password

WiFiServer server(80);

void setup()
{
    Serial.begin(115200);
    pinMode(8, OUTPUT);      // set the LED pin mode

    delay(10);

    // We start by connecting to a WiFi network

    Serial.println();
    Serial.println();
    Serial.print("Connecting to ");
    Serial.println(ssid);

    WiFi.begin(ssid, password);

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }

    Serial.println();
    Serial.println("WiFi connected.");
    Serial.println("IP address: ");
    Serial.println(WiFi.localIP());
    
    server.begin();
}

void loop(){
  WiFiClient client = server.available();   // listen for incoming clients

  if (client) {                             // if you get a client,
    Serial.println("New Client.");           // print a message out the serial port
    String currentLine = "";                 // make a String to hold incoming data from the client
    while (client.connected()) {             // loop while the client's connected
      if (client.available()) {             // if there's bytes to read from the client,
        char c = client.read();             // read a byte, then
        Serial.write(c);                    // print it out the serial monitor
        if (c == '\n') {                    // if the byte is a newline character

          // if the current line is blank, you got two newline characters in a row.
          // that's the end of the client HTTP request, so send a response:
          if (currentLine.length() == 0) {
            // HTTP headers always start with a response code (e.g. HTTP/1.1 200 OK)
            // and a content-type so the client knows what's coming, then a blank line:
            client.println("HTTP/1.1 200 OK");
            client.println("Content-type:text/html");
            client.println();

            // the content of the HTTP response follows the header:
            client.print("Click <a href=\"/H\">here</a> to turn the LED on pin 8 on.<br>");
            client.print("Click <a href=\"/L\">here</a> to turn the LED on pin 8 off.<br>");

            // The HTTP response ends with another blank line:
            client.println();
            // break out of the while loop:
            break;
          } else {    // if you got a newline, then clear currentLine:
            currentLine = "";
          }
        } else if (c != '\r') {  // if you got anything else but a carriage return character,
          currentLine += c;      // add it to the end of the currentLine
        }

        // Check to see if the client request was "GET /H" or "GET /L":
        if (currentLine.endsWith("GET /H")) {
          digitalWrite(8, HIGH);               // GET /H turns the LED on
        }
        if (currentLine.endsWith("GET /L")) {
          digitalWrite(8, LOW);                // GET /L turns the LED off
        }
      }
    }
    // close the connection:
    client.stop();
    Serial.println("Client Disconnected.");
  }
}
```

---

## WiFi Usage

### Scan WiFi Networks (Station Mode)

Use the ESP32S3SuperMini to scan available WiFi networks around it. The board is configured as Station (STA) mode.

**Step 1:** Copy and paste the following code into Arduino IDE:

```cpp
#include "WiFi.h"

void setup()
{
    Serial.begin(115200);

    // Set WiFi to station mode and disconnect from an AP if it was previously connected
    WiFi.mode(WIFI_STA);
    WiFi.disconnect();
    delay(100);

    Serial.println("Setup done");
}

void loop()
{
    Serial.println("scan start");

    // WiFi.scanNetworks will return the number of networks found
    int n = WiFi.scanNetworks();
    Serial.println("scan done");
    if (n == 0) {
        Serial.println("no networks found");
    } else {
        Serial.print(n);
        Serial.println(" networks found");
        for (int i = 0; i < n; ++i) {
            // Print SSID and RSSI for each network found
            Serial.print(i + 1);
            Serial.print(": ");
            Serial.print(WiFi.SSID(i));
            Serial.print(" (");
            Serial.print(WiFi.RSSI(i));
            Serial.print(")");
            Serial.println((WiFi.encryptionType(i) == WIFI_AUTH_OPEN)?" ":"*");
            delay(10);
        }
    }
    Serial.println("");

    // Wait a bit before scanning again
    delay(5000);
}
```

**Step 2:** Upload the code and open the Serial Monitor to start scanning WiFi networks.

### Connect to WiFi Network

This example connects the ESP32S3SuperMini to a WiFi network.

**Step 1:** Copy and paste the following code into Arduino IDE:

```cpp
#include <WiFi.h>

const char* ssid     = "your-ssid"; // Your WiFi name
const char* password = "your-password";   // Your WiFi password

void setup()
{
    Serial.begin(115200);
    delay(10);

    // We start by connecting to a WiFi network

    Serial.println();
    Serial.println();
    Serial.print("Connecting to ");
    Serial.println(ssid);

    WiFi.begin(ssid, password);

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }

    Serial.println();
    Serial.println("WiFi connected");
    Serial.println("IP address: ");
    Serial.println(WiFi.localIP());
}

void loop()
{
}
```

**Step 2:** Upload the code and open the Serial Monitor to check if the board has connected to the WiFi network.

### WiFi Hotspot (SoftAP)

In this example, the ESP32S3SuperMini acts as a WiFi access point that other devices can connect to, similar to a phone's WiFi hotspot feature.

**Step 1:** Copy and paste the following code into Arduino IDE:

```cpp
#include "WiFi.h"
void setup()
{
  Serial.begin(115200);
  WiFi.softAP("ESP_AP", "12345678");
}

void loop()
{
  Serial.print("Host Name:");
  Serial.println(WiFi.softAPgetHostname());
  Serial.print("Host IP:");
  Serial.println(WiFi.softAPIP());
  Serial.print("Host IPV6:");
  Serial.println(WiFi.softAPIPv6());
  Serial.print("Host SSID:");
  Serial.println(WiFi.SSID());
  Serial.print("Host Broadcast IP:");
  Serial.println(WiFi.softAPBroadcastIP());
  Serial.print("Host mac Address:");
  Serial.println(WiFi.softAPmacAddress());
  Serial.print("Number of Host Connections:");
  Serial.println(WiFi.softAPgetStationNum());
  Serial.print("Host Network ID:");
  Serial.println(WiFi.softAPNetworkID());
  Serial.print("Host Status:");
  Serial.println(WiFi.status());
  delay(1000);
}
```

**Step 2:** Upload the code and open the Serial Monitor to view detailed information about the WiFi access point.

---

## Bluetooth Usage

### Scan Bluetooth Devices

Use the ESP32S3SuperMini to scan for nearby Bluetooth devices.

**Step 1:** Copy and paste the following code into Arduino IDE:

```cpp
#include <BLEDevice.h>
#include <BLEUtils.h>
#include <BLEScan.h>
#include <BLEAdvertisedDevice.h>

int scanTime = 5; // In seconds
BLEScan* pBLEScan;

class MyAdvertisedDeviceCallbacks: public BLEAdvertisedDeviceCallbacks {
    void onResult(BLEAdvertisedDevice advertisedDevice) {
      Serial.printf("Advertised Device: %s \n", advertisedDevice.toString().c_str());
    }
};

void setup() {
  Serial.begin(115200);
  Serial.println("Scanning...");

  BLEDevice::init("");
  pBLEScan = BLEDevice::getScan(); // create new scan
  pBLEScan->setAdvertisedDeviceCallbacks(new MyAdvertisedDeviceCallbacks());
  pBLEScan->setActiveScan(true); // active scan uses more power, but get results faster
  pBLEScan->setInterval(100);
  pBLEScan->setWindow(99);  // less or equal setInterval value
}

void loop() {
  // put your main code here, to run repeatedly:
  BLEScanResults foundDevices = pBLEScan->start(scanTime, false);
  Serial.print("Devices found: ");
  Serial.println(foundDevices.getCount());
  Serial.println("Scan done!");
  pBLEScan->clearResults();   // delete results from BLEScan buffer to release memory
  delay(2000);
}
```

**Step 2:** Upload the code and open the Serial Monitor to start scanning Bluetooth devices.

### As Bluetooth Server

In this example, the ESP32S3SuperMini acts as a Bluetooth server. Use a smartphone to search for the ESP32S3SuperMini board and send a string to display in the Serial Monitor.

**Step 1:** Copy and paste the following code into Arduino IDE:

```cpp
#include <BLEDevice.h>
#include <BLEUtils.h>
#include <BLEServer.h>

// See the following for generating UUIDs:
// https://www.uuidgenerator.net/

#define SERVICE_UUID        "4fafc201-1fb5-459e-8fcc-c5c9c331914b"
#define CHARACTERISTIC_UUID "beb5483e-36e1-4688-b7f5-ea07361b26a8"


class MyCallbacks: public BLECharacteristicCallbacks {
    void onWrite(BLECharacteristic *pCharacteristic) {
      std::string value = pCharacteristic->getValue();

      if (value.length() > 0) {
        Serial.println("*********");
        Serial.print("New value: ");
        for (int i = 0; i < value.length(); i++)
          Serial.print(value[i]);

        Serial.println();
        Serial.println("*********");
      }
    }
};

void setup() {
  Serial.begin(115200);

  BLEDevice::init("MyESP32");
  BLEServer *pServer = BLEDevice::createServer();

  BLEService *pService = pServer->createService(SERVICE_UUID);

  BLECharacteristic *pCharacteristic = pService->createCharacteristic(
                                         CHARACTERISTIC_UUID,
                                         BLECharacteristic::PROPERTY_READ |
                                         BLECharacteristic::PROPERTY_WRITE
                                       );

  pCharacteristic->setCallbacks(new MyCallbacks());

  pCharacteristic->setValue("Hello World");
  pService->start();

  BLEAdvertising *pAdvertising = pServer->getAdvertising();
  pAdvertising->start();
}

void loop() {
  // put your main code here, to run repeatedly:
  delay(2000);
}
```

**Step 2:** Upload the code and open the Serial Monitor.

**Step 3:** On your smartphone, download and install the LightBlue app.

- [LightBlue App (Android)](https://play.google.com/store/apps/details?id=com.punchthrough.lightblueexplorer&hl=en_US&gl=US&pli=1)
- [LightBlue App (iOS)](https://apps.apple.com/us/app/lightblue/id557428110)

**Step 4:** Turn on phone Bluetooth, bring the phone close to ESP32S3SuperMini, scan for devices, and connect to the MyESP32 device.

**Step 5:** Open the LightBlue app and click the **Bonded** tab.

**Step 6:** Click **CONNECT** next to MyESP32.

**Step 7:** Click the section at the bottom that shows "Readable" / "Writable".

**Step 8:** In the data format dropdown, select **UTF-8 String**.

**Step 9:** Type "Hello" in the **WRITTEN VALUES** field, then click **WRITE**.

You will see the text string "Hello" output in the Arduino IDE Serial Monitor.

---

## ChatGPT Integration

### Overview

ChatGPT is a chatbot model released by OpenAI on November 30, 2022. It is an AI-powered natural language processing tool that can converse and interact based on chat context. In embedded systems, ChatGPT can be a helpful assistant for writing simple programs, checking, and fixing bugs.

OpenAI provides official API interfaces for calling the GPT-3.5 model, enabling deployment of this model on embedded systems. The ESP32S3SuperMini, with its WiFi/Bluetooth dual-mode capability, can perfectly support Arduino's WiFi Client and HTTP Client libraries to call the OpenAI ChatGPT API and create a custom Q&A web page.

This tutorial guides users through using ESP32S3 WiFiClient and HTTPClient libraries, connecting to networks, publishing web pages, and the basics of HTTP GET and POST to call OpenAI ChatGPT and create a Q&A website.

The tutorial is divided into four main steps:

1. **Configure ESP32S3SuperMini to connect to the network** — Learn basic WiFi configuration, network setup, connecting to network services, and obtaining the IP address.
2. **Build an embedded web page** — Use the WiFi client library's GET and POST functions to write and deploy a Q&A web page on the ESP32S3SuperMini.
3. **Submit questions via the built-in web page** — Learn how to use HTTP Client's POST method to send questions to OpenAI API according to the OpenAI API standard.
4. **Get answers from ChatGPT** — Use HTTP Client POST to retrieve responses and parse the answer from the returned JSON.

### Prerequisites

You need the following items:

- 1 x ESP32S3SuperMini
- 1 computer
- 1 USB Type-C data cable
- 1 ChatGPT API key (contact the group owner if you don't have one)

> **Tip:** Some USB cables can only provide power, not data transfer. If you don't have a USB cable or are unsure whether your USB cable can transfer data, you can purchase a [Type-C data cable](https://item.taobao.com/item.htm?spm=a1z10.3-c-s.w4002-24438210134.9.24866ea30WLxAl&id=679700862802).

**Step 1:** Connect the ESP32S3SuperMini to your computer using a USB Type-C data cable.

**Step 2:** Download and install the latest version of Arduino IDE.

**Step 3:** Add the ESP32 board package to Arduino IDE.

Navigate to **File > Preferences**, then enter the following URL in "Additional Boards Manager URL":

```
https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
```

Navigate to **Tools > Board > Boards Manager...**, enter "esp32" in the search box, and install the latest version of ESP32.

Navigate to **Tools > Board > ESP32 Arduino** and select **"ESP32s3 Dev Module"**.

Navigate to **Tools > Port** and select the serial port name of the connected ESP32S3SuperMini.

### Configure ESP32S3 to Connect to Network

The ESP32S3SuperMini WiFi usage tutorial has detailed instructions.

When ESP32 is set to WiFi mode, it can connect to other networks (e.g., a router). In this case, the router assigns a unique IP address to the ESP32 development board.

To use ESP32 WiFi functionality, the first thing to do is include the WiFi.h library:

```cpp
#include <WiFi.h>
```

To connect ESP32 to a specific WiFi network, you must know its SSID and password. The network must be within ESP32 WiFi range.

First, set the WiFi mode. If ESP32 will connect to another network (access point/hotspot), it must be in station mode:

```cpp
WiFi.mode(WIFI_STA);
```

Then use `WiFi.begin()` to connect to the network. You must pass the network SSID and password as parameters.

Connecting to a WiFi network may take some time, so we typically add a while loop that continuously checks whether the connection has been established using `WiFi.status()`. When the connection is successfully established, it returns `WL_CONNECTED`.

When ESP32 is set to WiFi station mode, it can connect to other networks (e.g., a router). In this case, the router assigns a unique IP address to the ESP32 board. To get the board's IP address, call `WiFi.localIP()` after establishing the network connection.

```cpp
void WiFiConnect(void){
    WiFi.begin(ssid, password);
    Serial.print("Connecting to ");
    Serial.println(ssid);

    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }

    Serial.println();
    Serial.println("WiFi connected!");
    Serial.print("IP address: ");
    Serial.println(WiFi.localIP());
}
```

The variables `password` and `ssid` store the password and SSID of the network you want to connect to.

```cpp
// Replace with your network credentials
const char* ssid = "REPLACE_WITH_YOUR_SSID";
const char* password = "REPLACE_WITH_YOUR_PASSWORD";
```

This is a very simple WiFi connection program. Upload the program to ESP32S3SuperMini, then open the serial assistant and set the baud rate to 115200. If the connection is successful, you will see the printed IP address.

### Build Embedded Web Page

The ESP32 WiFi library integrates many useful WiFiClient functions that allow designing and developing embedded web pages without additional libraries.

Create a new WiFiServer object to control the IoT server established by the ESP32S3SuperMini:

```cpp
WiFiServer server(80);
WiFiClient client1;
```

After the ESP32S3SuperMini connects to WiFi successfully, you can obtain the current IP address from the Serial Monitor. The ESP32S3SuperMini has successfully built a Web server. You can access this Web server through the ESP32S3SuperMini's IP address.

Assuming your ESP32S3SuperMini's IP address is `192.168.7.152`, you can enter this IP address in a browser.

After entering the IP address, you may only see a blank page. This is because the page content has not been published yet.

Now use C to create an array to store the page content you want to layout, i.e., HTML code:

```cpp
const char html_page[] PROGMEM = {
    "HTTP/1.1 200 OK\r\n"
    "Content-Type: text/html\r\n"
    "Connection: close\r\n"
    "\r\n"
    "<!DOCTYPE HTML>\r\n"
    "<html>\r\n"
    "<head>\r\n"
      "<meta charset=\"UTF-8\">\r\n"
      "<title>Cloud Printer: ChatGPT</title>\r\n"
      "<link rel=\"icon\" href=\"https://files.seeedstudio.com/wiki/xiaoesp32c3-chatgpt/chatgpt-logo.png\" type=\"image/x-icon\">\r\n"
    "</head>\r\n"
    "<body>\r\n"
    "<p style=\"text-align:center;\">\r\n"
    "<img alt=\"ChatGPT\" src=\"https://files.seeedstudio.com/wiki/xiaoesp32c3-chatgpt/chatgpt-logo.png\" height=\"200\" width=\"200\">\r\n"
    "<h1 align=\"center\">Cloud Printer</h1>\r\n" 
    "<h1 align=\"center\">OpenAI ChatGPT</h1>\r\n" 
    "<div style=\"text-align:center;vertical-align:middle;\">"
    "<form action=\"/\" method=\"post\">"
    "<input type=\"text\" placeholder=\"Please enter your question\" size=\"35\" name=\"chatgpttext\" required=\"required\"/>\r\n"
    "<input type=\"submit\" value=\"Submit\" style=\"height:30px; width:80px;\"/>"
    "</form>"
    "</div>"
    "</p>\r\n"
    "</body>\r\n"
    "<html>\r\n"
};
```

> **Tip:** HTML syntax is beyond the scope of this tutorial. You can learn HTML yourself or use existing code generation tools. We recommend using an [HTML generator](https://webcode.tools/). Note that in C programs, `\` and `"` are special characters; if you want to preserve their special functions in the program, you need to add a backslash before them.

`client1` refers to the Socket client after the Web server is established. The following code is the Web server processing flow:

```cpp
client1 = server.available();
if (client1){
    Serial.println("New Client.");
    boolean currentLineIsBlank = true;    
    while (client1.connected()){
        if (client1.available()){
            char c = client1.read();
            json_String += c;
            if (c == '\n' && currentLineIsBlank) {                                 
                dataStr = json_String.substring(0, 4);
                Serial.println(dataStr);
                if(dataStr == "GET "){
                    client1.print(html_page);  // Send the response body to the client
                }         
                else if(dataStr == "POST"){
                    json_String = "";
                    while(client1.available()){
                        json_String += (char)client1.read();
                    }
                    Serial.println(json_String); 
                    dataStart = json_String.indexOf("chatgpttext=") + strlen("chatgpttext=");
                    chatgpt_Q = json_String.substring(dataStart, json_String.length());                    
                    client1.print(html_page);        
                    delay(10);
                    client1.stop();       
                }
                json_String = "";
                break;
            }
            if (c == '\n') {
                currentLineIsBlank = true;
            }
            else if (c != '\r') {
                currentLineIsBlank = false;
            }
        }
    }
}
```

In the above example program, you need to use `server.begin()` to start the IoT server. This statement should be placed in the `setup()` function:

```cpp
void setup()
{
    Serial.begin(115200);
 
    WiFi.mode(WIFI_STA);
    WiFi.disconnect();
    while(!Serial);

    Serial.println("WiFi Setup done!");
    
    WiFiConnect();
    server.begin();
}
```

Once the above program is running and you enter the ESP32S3SuperMini's IP address in a browser (provided your host is on the same WiFi as the ESP32S3SuperMini), the WiFiClient GET step will begin execution. The client print method sends the HTML page content.

```cpp
if(dataStr == "GET "){
    client1.print(html_page);
}
```

The input box on the page is where users enter their questions. After the user enters content and clicks the submit button, the webpage captures the button state and stores the entered question in the string variable `chatgpt_Q`.

### Submit Questions to ChatGPT

On the previous page, there is an input box. The task is to get this question and send it out via the OpenAI API.

**Step 1:** Register an OpenAI account. (Registration may not work normally from within China; search Baidu for alternatives.)

You can click [here](https://platform.openai.com/signup) to go to the OpenAI registration page. If you have already registered, skip this step.

**Step 2:** Get the OpenAI API key.

Log in to the [OpenAI website](https://platform.openai.com/overview), click your account avatar in the top right corner, and select "View API Keys".

In the new popup page, click **+ Create new secret key**, then copy your key and save it.

In the program, create a string variable and copy the key here:

```cpp
char chatgpt_token[] = "sk**********Rj9DYWSDFNA";
```

**Step 3:** Write the program according to OpenAI's HTTP request format.

OpenAI provides detailed [API usage documentation](https://platform.openai.com/docs/api-reference/making-requests) for users to call ChatGPT using their own API key.

According to the ChatGPT documentation, the format for sending requests is:

```
curl https://api.openai.com/v1/completions \
-H "Content-Type: application/json" \
-H "Authorization: Bearer YOUR_API_KEY" \
-d '{"model": "text-davinci-003", "prompt": "Say this is a test", "temperature": 0, "max_tokens": 7}'
```

HTTP (HyperText Transfer Protocol) is the request-response protocol used between client and server. GET is used to request data from a specified resource, typically for retrieving values from APIs. POST is used to send data to the server to create/update resources. ESP32 can issue HTTP POST requests with three types of body formats: URL-encoded, JSON object, or plain text. These are the most common methods and should work with most APIs or web services.

First, import the HTTPClient library:

```cpp
#include <HTTPClient.h>
```

You also need to enter the OpenAI domain so that ESP32S3 can post the question to ChatGPT. Do not forget the OpenAI API key:

```cpp
HTTPClient https;
const char* chatgpt_token = "YOUR_API_KEY";
char chatgpt_server[] = "https://api.openai.com/v1/completions";
```

We need to use a JSON object to issue an HTTP POST request:

```cpp
if (https.begin(chatgpt_server)) {  // HTTPS
    https.addHeader("Content-Type", "application/json"); 
    String token_key = String("Bearer ") + chatgpt_token;
    https.addHeader("Authorization", token_key);
    String payload = String("{\"model\": \"text-davinci-003\", \"prompt\": \"") + chatgpt_Q + String("\", \"temperature\": 0, \"max_tokens\": 100}");
    httpCode = https.POST(payload);   // start connection and send HTTP header
    payload = "";
}
else {
    Serial.println("[HTTPS] Unable to connect");
    delay(1000);
}
```

In the program, the `POST()` method sends the `payload` to the server. `chatgpt_Q` is the question content to be sent to ChatGPT, which is provided by the "Get Question" page.

For more information on ESP32S3 HTTPClient functionality, we recommend reading [ESP32 HTTP GET and HTTP POST with Arduino IDE](https://randomnerdtutorials.com/esp32-http-get-post-arduino/).

### Get Answers from ChatGPT

This is the final step of the entire tutorial: how to get the ChatGPT answer and record it.

Continue reading the OpenAI API documentation to understand the structure of the messages returned by ChatGPT. This will enable writing a program to parse the content we need.

The API returns a JSON structure like this:

```json
{
    "id": "cmpl-GERzeJQ4lvqPk8SkZu4XMIuR",
    "object": "text_completion",
    "created": 1586839808,
    "model": "text-davinci:003",
    "choices": [
        {
            "text": "\n\nThis is indeed a test",
            "index": 0,
            "logprobs": null,
            "finish_reason": "length"
        }
    ],
    "usage": {
        "prompt_tokens": 5,
        "completion_tokens": 7,
        "total_tokens": 12
    }
}
```

From the OpenAI reference documentation, we know the answer to the question is located at `{"choices":[{"text":"\n\nxxxxxxx",}]}`.

So we can determine that the "answer" we need starts with `\n\n` and ends with `","`. In the program, we can use the `indexOf()` method to find the start and end positions of the text, and store the returned answer content:

```cpp
dataStart = payload.indexOf("\\n\\n") + strlen("\\n\\n");
dataEnd = payload.indexOf("\",\"", dataStart); 
chatgpt_A = payload.substring(dataStart, dataEnd);
```

In summary, we can use a `switch` statement combined with the program's current state to determine which step of the program to execute:

```cpp
typedef enum 
{
  do_webserver_index,
  send_chatgpt_request,
  get_chatgpt_list,
} STATE_;

STATE_ currentState;

switch(currentState){
    case do_webserver_index:
        // Handle web server requests
    case send_chatgpt_request:
        // Send question to ChatGPT
    case get_chatgpt_list:
        // Get ChatGPT answer
}
```

The complete integrated code is as follows. Do not rush to upload the program; you need to change the `ssid`, `password`, and `chatgpt_token` in the program to your own values.

```cpp
#include "WiFi.h"
#include <HTTPClient.h>

// Replace with your network credentials
const char* ssid     = "WIFI_ACCOUNT";
const char* password = "WIFI_PASSWORD"; 
// chatgpt api
const char* chatgpt_token = "YOUR_OPENAI_API_KEY";

char chatgpt_server[] = "https://api.openai.com/v1/completions";

// Set web server port number to 80
WiFiServer server(80);
WiFiClient client1;

HTTPClient https;

String chatgpt_Q;
String chatgpt_A;
String json_String;
uint16_t dataStart = 0;
uint16_t dataEnd = 0;
String dataStr;
int httpCode = 0;

typedef enum 
{
  do_webserver_index,
  send_chatgpt_request,
  get_chatgpt_list,
} STATE_;

STATE_ currentState;

void WiFiConnect(void){
    WiFi.begin(ssid, password);
    Serial.print("Connecting to ");
    Serial.println(ssid);
    while (WiFi.status() != WL_CONNECTED) {
        delay(500);
        Serial.print(".");
    }
    Serial.println();
    Serial.println("WiFi connected!");
    Serial.print("IP address: ");
    Serial.println(WiFi.localIP());
    currentState = do_webserver_index;
}

const char html_page[] PROGMEM = {
    "HTTP/1.1 200 OK\r\n"
    "Content-Type: text/html\r\n"
    "Connection: close\r\n"
    "\r\n"
    "<!DOCTYPE HTML>\r\n"
    "<html>\r\n"
    "<head>\r\n"
      "<meta charset=\"UTF-8\">\r\n"
      "<title>Cloud Printer: ChatGPT</title>\r\n"
      "<link rel=\"icon\" href=\"https://files.seeedstudio.com/wiki/xiaoesp32c3-chatgpt/chatgpt-logo.png\" type=\"image/x-icon\">\r\n"
    "</head>\r\n"
    "<body>\r\n"
    "<p style=\"text-align:center;\">\r\n"
    "<img alt=\"ChatGPT\" src=\"https://files.seeedstudio.com/wiki/xiaoesp32c3-chatgpt/chatgpt-logo.png\" height=\"200\" width=\"200\">\r\n"
    "<h1 align=\"center\">Cloud Printer</h1>\r\n" 
    "<h1 align=\"center\">OpenAI ChatGPT</h1>\r\n" 
    "<div style=\"text-align:center;vertical-align:middle;\">"
    "<form action=\"/\" method=\"post\">"
    "<input type=\"text\" placeholder=\"Please enter your question\" size=\"35\" name=\"chatgpttext\" required=\"required\"/>\r\n"
    "<input type=\"submit\" value=\"Submit\" style=\"height:30px; width:80px;\"/>"
    "</form>"
    "</div>"
    "</p>\r\n"
    "</body>\r\n"
    "<html>\r\n"
};
 
void setup()
{
    Serial.begin(115200);
 
    WiFi.mode(WIFI_STA);
    WiFi.disconnect();
    while(!Serial);

    Serial.println("WiFi Setup done!");
    
    WiFiConnect();
    server.begin();
}
 
void loop()
{
  switch(currentState){
    case do_webserver_index:
      Serial.println("Web Production Task Launch");
      client1 = server.available();
      if (client1){
        Serial.println("New Client.");
        boolean currentLineIsBlank = true;    
        while (client1.connected()){
          if (client1.available()){
            char c = client1.read();
            json_String += c;
            if (c == '\n' && currentLineIsBlank) {                                 
              dataStr = json_String.substring(0, 4);
              Serial.println(dataStr);
              if(dataStr == "GET "){
                client1.print(html_page);
              }         
              else if(dataStr == "POST"){
                json_String = "";
                while(client1.available()){
                  json_String += (char)client1.read();
                }
                Serial.println(json_String); 
                dataStart = json_String.indexOf("chatgpttext=") + strlen("chatgpttext=");
                chatgpt_Q = json_String.substring(dataStart, json_String.length());                    
                client1.print(html_page);
                Serial.print("Your Question is: ");
                Serial.println(chatgpt_Q);
                delay(10);
                client1.stop();       
                currentState = send_chatgpt_request;
              }
              json_String = "";
              break;
            }
            if (c == '\n') {
              currentLineIsBlank = true;
            }
            else if (c != '\r') {
              currentLineIsBlank = false;
            }
          }
        }
      }
      delay(1000);
      break;
    case send_chatgpt_request:
      Serial.println("Ask ChatGPT a Question Task Launch");
      if (https.begin(chatgpt_server)) {
        https.addHeader("Content-Type", "application/json"); 
        String token_key = String("Bearer ") + chatgpt_token;
        https.addHeader("Authorization", token_key);
        String payload = String("{\"model\": \"text-davinci-003\", \"prompt\": \"") + chatgpt_Q + String("\", \"temperature\": 0, \"max_tokens\": 100}");
        httpCode = https.POST(payload);
        payload = "";
        currentState = get_chatgpt_list;
      }
      else {
        Serial.println("[HTTPS] Unable to connect");
        delay(1000);
      }
      break;
    case get_chatgpt_list:
      Serial.println("Get ChatGPT Answers Task Launch");
      if (httpCode == HTTP_CODE_OK || httpCode == HTTP_CODE_MOVED_PERMANENTLY) {
        String payload = https.getString();
        dataStart = payload.indexOf("\\n\\n") + strlen("\\n\\n");
        dataEnd = payload.indexOf("\",\"", dataStart); 
        chatgpt_A = payload.substring(dataStart, dataEnd);
        Serial.print("ChatGPT Answer is: ");
        Serial.println(chatgpt_A);
        Serial.println("Wait 10s before next round...");
        currentState = do_webserver_index;
      }
      else {
        Serial.printf("[HTTPS] GET... failed, error: %s\n", https.errorToString(httpCode).c_str());
        while(1);
      }
      https.end();
      delay(10000);
      break;
  }
}
```

This code originates from [the source repository](https://github.com/limengdu/xiaoesp32c3-chatgpt).

---

## Pin Usage

The ESP32S3SuperMini has rich interfaces. It has 11 digital I/O pins usable as PWM pins, 4 analog input pins usable as ADC pins, and supports four serial communication interfaces: UART, I2C, SPI, and I2S. This article helps you understand these interfaces and implement them in your next project.

Regarding pins A0-A5, GPIO0-GPIO10, and pins starting with D: the default board only has GPIO-prefixed pins (0, 10, 20, 21). The A0-A5 pins are a mapping issue — they are used to conveniently tell users whether a pin functions as an analog or digital pin. When selecting the board type in Arduino, choose **ESP32S3 Dev Module** to reference its pin mapping. The pin mapping diagram is as follows:

![Arduino ESP32S3 Dev Module Pin Mapping](data/esp32/esp32s3supermini/esp32s3foot2.png)

*Arduino ESP32S3 Dev Module Pin Mapping*

### Digital Pins

Upload the following code to the board, and the onboard LED will blink on and off every second.

```cpp
// define led according to pin diagram
int led = 8;

void setup() {
  // initialize digital pin led as an output
  pinMode(led, OUTPUT);
}

void loop() {
  digitalWrite(led, HIGH);   // turn the LED on
  delay(1000);               // wait for a second
  digitalWrite(led, LOW);    // turn the LED off
  delay(1000);               // wait for a second
}
```

### Digital PWM

Upload the following code to see the onboard LED gradually dim.

```cpp
int ledPin = 8;    // LED connected to digital pin 8

void setup() {
  // declaring LED pin as output
  pinMode(ledPin, OUTPUT);
}

void loop() {
  // fade in from min to max in increments of 5 points:
  for (int fadeValue = 0 ; fadeValue <= 255; fadeValue += 5) {
    // sets the value (range from 0 to 255):
    analogWrite(ledPin, fadeValue);
    // wait for 30 milliseconds to see the dimming effect
    delay(30);
  }

  // fade out from max to min in increments of 5 points:
  for (int fadeValue = 255 ; fadeValue >= 0; fadeValue -= 5) {
    // sets the value (range from 0 to 255):
    analogWrite(ledPin, fadeValue);
    // wait for 30 milliseconds to see the dimming effect
    delay(30);
  }
}
```

### Analog Pins

Connect a potentiometer to pin A5, then upload the following code to control the LED blink interval by rotating the potentiometer knob.

```cpp
const int sensorPin = A5;
const int ledPin =  8; 

void setup() {
  pinMode(sensorPin, INPUT);  // declare the sensorPin as an INPUT
  pinMode(ledPin, OUTPUT);   // declare the ledPin as an OUTPUT
}

void loop() {
  // read the value from the sensor:
  int sensorValue = analogRead(sensorPin);
  // turn the ledPin on
  digitalWrite(ledPin, HIGH);
  // stop the program for <sensorValue> milliseconds:
  delay(sensorValue);
  // turn the ledPin off:
  digitalWrite(ledPin, LOW);
  // stop the program for <sensorValue> milliseconds:
  delay(sensorValue);
}
```

### Serial Communication

#### Hardware Serial

The board has two hardware serial ports:

- **USB serial** — enabled by default, allowing you to connect the development board to a PC via USB Type-C and open the Serial Monitor in Arduino IDE to view data sent via serial.
- **UART serial** — requires a USB serial adapter to connect the TX pin to the TX pin and the RX pin to the RX pin.

Additionally, you need to set **USB CDC On Boot** in Arduino IDE to **Disabled** when using UART serial.

#### Software Serial

If you need more serial ports, use the `SoftwareSerial` library to create a software serial port.

---

## Micropython

- [ESP32S3SuperMini Micropython Firmware Download](https://micropython.org/download/ESP32_GENERIC_S3/)
- [Micropython Firmware Download Tutorial](https://chat.nologo.tech/d/75)

---

## Learning Resources

- [Open-source Project ESP32 Arduino Tutorial](https://docs.geeksman.com/esp32/#/directory-arduino)
- [Open-source Project ESP32 Micropython Tutorial](https://docs.geeksman.com/esp32/#/directory-micropython)
- [ESP32-S3 Official Learning Resources](https://docs.espressif.com/projects/esp-idf/zh_CN/latest/esp32s3/get-started/index.html)
- [Collected ESP32-S3 Learning Materials](https://pan.baidu.com/s/1OcWRi08R4A-Cxg8MOuoKtA?pwd=8888)

---

## Troubleshooting

### Q1: Cannot recognize Com port in Arduino

Enter download mode:
- **Method 1:** Hold BOOT while powering on.
- **Method 2:** Press and hold the BOOT button on ESP32S3, then press the RESET button, release the RESET button, and then release the BOOT button. The ESP32S3 will enter download mode. (The download mode needs to be re-entered each time you connect. If the port is unstable and disconnects after pressing once, you can determine it by the port recognition sound.)

### Q2: Program does not run after upload

After uploading successfully, press the Reset button to execute the program.

### Q3: Com port not displayed when plugging into computer, shows "JTAG/serial debug unit"

- [Solution for JTAG/serial debug unit display](https://chat.nologo.tech/d/72/3)

### Q4: ESP32S3SuperMini Arduino serial port cannot print

Need to set **USB CDC On Boot** in the toolbar to **Enabled**.

> For more issues and interesting applications, please visit [forum](https://chat.nologo.tech/) or join the QQ technical communication group: **1054780900**

---

## Purchase Links

- [Taobao Purchase](https://item.taobao.com/item.htm?spm=a1z10.3-c-s.w4002-24438210134.9.27bc6ea3Er0bkc&id=707413078834)
- [AliExpress Store](https://www.aliexpress.com/store/1104139274)