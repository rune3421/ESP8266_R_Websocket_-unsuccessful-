# ESP8266_R_Websocket_-unsuccessful-

This is an attempt to get R to connect directly to an ESP8266 via websocket for live updates. 

Hello, This is my second try at using the ESP8266 Thing Dev to build a websocket-based transmitter for the Baby Boot project. 

It turns out, after some research, I burned up the original chip by running more than 1V through the ADC when trying to troubleshoot some of the software challenges I had while learning about the websocket stack. So, I finished a related side project (reading Arduino Serial into RStudio) while waiting for the replacement ESP chip I ordered. I went ahead and got the pin headers for this one, so that I don’t have to risk damaging this one by soldering it myself. 

I’ve confirmed it runs and uploads code, so now it’s time to use this tutorial to find out how to run websockets well. I have a feeling it’ll be a good idea to go at this from the beginning, since my first try at getting websockets to run got me lost, fast. I’m going to start by running and modifying something in each tutorial step, to make sure I know how to modify the code without breaking it, and then I’ll go back and try to re-route the outputs on a few select exercises to connect to RStudio, so that I know by the end of this project that I’m connecting the ESP to RStudio in a stable way, and maybe have a few extra tools when it comes time for transforming and analyzing the live data in a dashboard. 

First step is reading some of the reference documentation for the ESP, and browsing through the exercises that are available. I’m glad I did this part, because there actually is a faster wifi protocol available than websockets that I might want to try: Open Sound Control. It seems like this tutorial intends to demonstrate code for that, and it would work since it’s designed for low latency transfer of ints and floats, which is just what we want for a high fidelity, streaming data source from the boot. 

Now that I have an overall goal for this project (to get an Open Sound Control type connection between the ESP and RStudio, and if that’s not possible, to get a Websocket connection instead), it’s time to start the first part of the tutorial; the Ping. 

#include <ESP8266WiFi.h>        // Include the Wi-Fi library

const char* ssid     = "NETGEAR76";         // The SSID (name) of the Wi-Fi network you want to connect to
const char* password = "elegantplanet085";     // The password of the Wi-Fi network

void setup() {
  Serial.begin(115200);         // Start the Serial communication to send messages to the computer
  delay(10);
  Serial.println('\n');
  
  WiFi.begin(ssid, password);             // Connect to the network
  Serial.print("Connecting to ");
  Serial.print(ssid); Serial.println(" ...");

  int i = 0;
  while (WiFi.status() != WL_CONNECTED) { // Wait for the Wi-Fi to connect
    delay(1000);
    Serial.print(++i); Serial.print(' ');
  }

  Serial.println('\n');
  Serial.println("Connection established!");  
  Serial.print("IP address:\t");
  Serial.println(WiFi.localIP());         // Send the IP address of the ESP8266 to the computer
}

void loop() { }


The Serial monitor says it’s connected, let’s check on the laptop. Time to ping 192.168.1.13 in the Command Prompt. Ping worked! The ESP can connect. Now it’s time to modify…
In the header I’m inserting
WifiClient client;

And in the void loop I’m inserting 

void loop() {
  client.print(analogRead(A0));}

 just to see if the call client.print goes anywhere, and to see if I get what I expect; that there’s no voltage reading into the A0 port because there’s no wires connected yet. 
But, to read the output, I need to set up a web server/client relationship…which is part of a later lesson, so, rather than being able to edit, I think it’ll be easier to learn how by continuing the tutorial. So…so far no confirmation of the edit!

The next step was to setup WifiMulti, where the ESP remembers multiple possible networks and connects to whichever is strongest, but the code isn’t that different than the original, other than including the Wifimulti library and including multiple slots for router SSID’s and passwords. So, we’ll skip to the Access Point exercise, and use that as a jumping off point for adding in RStudio connectivity. We’ll keep the basic WifiConnection as a backup if the AP sketch doesn’t play nice. 

Here’s the basic sketch: 

#include <ESP8266WiFi.h>        // Include the Wi-Fi library

const char *ssid = "ESP8266 Access Point"; // The name of the Wi-Fi network that will be created
const char *password = "thereisnospoon";   // The password required to connect to it, leave blank for an open network

void setup() {
  Serial.begin(115200);
  delay(10);
  Serial.println('\n');

  WiFi.softAP(ssid, password);             // Start the access point
  Serial.print("Access Point \"");
  Serial.print(ssid);
  Serial.println("\" started");

  Serial.print("IP address:\t");
  Serial.println(WiFi.softAPIP());         // Send the IP address of the ESP8266 to the computer
}

void loop() { }

Checking on my iPhone, the access point does become available, and does connect. 
Now to check if my laptop can connect, before trying to ping it through RStudio. 
Ping results:

Pinging 192.168.4.1 with 32 bytes of data:
Reply from 192.168.4.1: bytes=32 time=15ms TTL=255
Reply from 192.168.4.1: bytes=32 time=5ms TTL=255
Reply from 192.168.4.1: bytes=32 time=21ms TTL=255
Reply from 192.168.4.1: bytes=32 time=90ms TTL=255

Ping statistics for 192.168.4.1:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 5ms, Maximum = 90ms, Average = 32ms

C:\Users\I'm a Dreamer>

Alright, so now the laptop is connected, and the ping does pong. Let’s get into RStudio and see if we can get the AP to ping pong from here…and after a brief search for ping functions from R, it looks like that’s more complicated than just jumping to a specific call. So once again, moving on in the tutorial without a successful edit. Wow, this is starting off rough. I have a lot to learn. 

Running the mDNS code:
#include <ESP8266WiFi.h>        // Include the Wi-Fi library
#include <ESP8266WiFiMulti.h>   // Include the Wi-Fi-Multi library
#include <ESP8266mDNS.h>        // Include the mDNS library

ESP8266WiFiMulti wifiMulti;     // Create an instance of the ESP8266WiFiMulti class, called 'wifiMulti'

void setup() {
  Serial.begin(115200);         // Start the Serial communication to send messages to the computer
  delay(10);
  Serial.println('\n');

  wifiMulti.addAP("NETGEAR76", "elegantplanet085");   // add Wi-Fi networks you want to connect to
  wifiMulti.addAP("ssid_from_AP_2", "your_password_for_AP_2");
  wifiMulti.addAP("ssid_from_AP_3", "your_password_for_AP_3");

  Serial.println("Connecting ...");
  int i = 0;
  while (wifiMulti.run() != WL_CONNECTED) { // Wait for the Wi-Fi to connect: scan for Wi-Fi networks, and connect to the strongest of the networks above
    delay(1000);
    Serial.print(++i); Serial.print(' ');
  }
  Serial.println('\n');
  Serial.print("Connected to ");
  Serial.println(WiFi.SSID());              // Tell us what network we're connected to
  Serial.print("IP address:\t");
  Serial.println(WiFi.localIP());           // Send the IP address of the ESP8266 to the computer

  if (!MDNS.begin("esp8266")) {             // Start the mDNS responder for esp8266.local
    Serial.println("Error setting up MDNS responder!");
  }
  Serial.println("mDNS responder started");
}

void loop() { }

Got a ping, but not a mDNS response. It could not return a ping using mDNS, but when checking on a webpage, the web page was able to return a “refused to connect”, which is hopeful, meaning it was found, but the request wasn’t formatted yet. Looking in the TaskManager revealed that Bonjour (windows version of mDNS) is running, so I should find out how to format them in a minute. 

Running the Web Server sketch and then looking up the IP address got Hello World to show, but the mDNS still did not resolve, so I’ll need to troubleshoot that. Next thing, since I can see the outputs, is I can edit this part of the code and try to get different outputs on the web server. This opens up options, and this might be the first sketch that I edit successfully, trying to do non-trivial things like printing to Serial. For this one, I’m going to try changing the text/plain to text/html and see what happens. AAAND, I didn’t break the code! It’s still pasting plain text, but it worked. Whew! 

So, I went to work since that last line, and now that I’m back it’s time to see where we’ve been, and where we’re going. It is becoming clear that I’m going to need to set up a server and a client for the websocket, and some of the intervening exercises aren’t going to be that helpful, and on top of that the websocket sketch in this exercise has a lot of extraneous stuff going on, so I’ll get my editing in, through deleting the unneeded code in that exercise, and in setting up the R Studio code to run its side. 

Starting with the Arduino exercise, I did a little editing (cut it down by over half), and studied the HTML and Java codes a bit to see what the R Studio side of the handshake would need to do. Here’s the cut down version:

#include <ESP8266WiFi.h>
#include <ESP8266WiFiMulti.h>
#include <ESP8266WebServer.h>
#include <ESP8266mDNS.h>
#include <FS.h>
#include <WebSocketsServer.h>

ESP8266WiFiMulti wifiMulti;       // Create an instance of the ESP8266WiFiMulti class, called 'wifiMulti'

ESP8266WebServer server(80);       // Create a webserver object that listens for HTTP request on port 80
WebSocketsServer webSocket(81);    // create a websocket server on port 81

File fsUploadFile;                 // a File variable to temporarily store the received file

const char *ssid = "ESP8266 Access Point"; // The name of the Wi-Fi network that will be created
const char *password = "thereisnospoon";   // The password required to connect to it, leave blank for an open network

//const char *OTAName = "ESP8266";           // A name and a password for the OTA service
//const char *OTAPassword = "esp8266";

#define LED_RED     15            // specify the pins with an RGB LED connected
#define LED_GREEN   12
#define LED_BLUE    13

const char* mdnsName = "esp8266"; // Domain name for the mDNS responder

void setup() {
  pinMode(LED_RED, OUTPUT);    // the pins with LEDs connected are outputs
  pinMode(LED_GREEN, OUTPUT);
  pinMode(LED_BLUE, OUTPUT);

  Serial.begin(115200);        // Start the Serial communication to send messages to the computer
  delay(10);
  Serial.println("\r\n");

  startWiFi();                 // Start a Wi-Fi access point, and try to connect to some given access points. Then wait for either an AP or STA connection

  startWebSocket();            // Start a WebSocket server
  
  startMDNS();                 // Start the mDNS responder

  startServer();               // Start a HTTP server with a file read handler and an upload handler
  
}

bool rainbow = false;             // The rainbow effect is turned off on startup

unsigned long prevMillis = millis();
int hue = 0;

void loop() {
  webSocket.loop();                           // constantly check for websocket events
  server.handleClient();                      // run the server
//  ArduinoOTA.handle();                        // listen for OTA events
}

void startWiFi() { // Start a Wi-Fi access point, and try to connect to some given access points. Then wait for either an AP or STA connection
  WiFi.softAP(ssid, password);             // Start the access point
  Serial.print("Access Point \"");
  Serial.print(ssid);
  Serial.println("\" started\r\n");

  wifiMulti.addAP("NETGEAR76", "elegantplanet085");   // add Wi-Fi networks you want to connect to
  wifiMulti.addAP("ssid_from_AP_2", "your_password_for_AP_2");
  wifiMulti.addAP("ssid_from_AP_3", "your_password_for_AP_3");

  Serial.println("Connecting");
  while (wifiMulti.run() != WL_CONNECTED && WiFi.softAPgetStationNum() < 1) {  // Wait for the Wi-Fi to connect
    delay(250);
    Serial.print('.');
  }
  Serial.println("\r\n");
  if(WiFi.softAPgetStationNum() == 0) {      // If the ESP is connected to an AP
    Serial.print("Connected to ");
    Serial.println(WiFi.SSID());             // Tell us what network we're connected to
    Serial.print("IP address:\t");
    Serial.print(WiFi.localIP());            // Send the IP address of the ESP8266 to the computer
  } else {                                   // If a station is connected to the ESP SoftAP
    Serial.print("Station connected to ESP8266 AP");
  }
  Serial.println("\r\n");
}


void startWebSocket() { // Start a WebSocket server
  webSocket.begin();                          // start the websocket server
  webSocket.onEvent(webSocketEvent);          // if there's an incomming websocket message, go to function 'webSocketEvent'
  Serial.println("WebSocket server started.");
}

void startMDNS() { // Start the mDNS responder
  MDNS.begin(mdnsName);                        // start the multicast domain name server
  Serial.print("mDNS responder started: http://");
  Serial.print(mdnsName);
  Serial.println(".local");
}

void startServer() { // Start a HTTP server with a file read handler and an upload handler
  server.on("/edit.html",  HTTP_POST, []() {  // If a POST request is sent to the /edit.html address,
    server.send(200, "text/plain", "Hello Server!"); 
  });                       // print 'Hello Server!'

  server.onNotFound(handleNotFound);          // if someone requests any other file or page, go to function 'handleNotFound'
                                              // and check if the file exists

  server.begin();                             // start the HTTP server
  Serial.println("HTTP server started.");
}

void handleNotFound(){ // if the requested file or page doesn't exist, return a 404 not found error
   ;
    server.send(404, "text/plain", "Oops! You gotta choose what happens with this call");
  }


void webSocketEvent(uint8_t num, WStype_t type, uint8_t * payload, size_t length) { // When a WebSocket message is received
  switch (type) {
    case WStype_DISCONNECTED:             // if the websocket is disconnected
      Serial.printf("[%u] Disconnected!\n", num);
      break;
    case WStype_CONNECTED: {              // if a new websocket connection is established
        IPAddress ip = webSocket.remoteIP(num);
        Serial.printf("[%u] Connected from %d.%d.%d.%d url: %s\n", num, ip[0], ip[1], ip[2], ip[3], payload);
        rainbow = false;                  // Turn rainbow off when a new connection is established
      }
      break;
    case WStype_TEXT:                     // if new text data is received
      Serial.printf("[%u] get Text: %s\n", num, payload);
      if (payload[0] == '#') {            // we get RGB data
        uint32_t rgb = (uint32_t) strtol((const char *) &payload[1], NULL, 16);   // decode rgb data
        int r = ((rgb >> 20) & 0x3FF);                     // 10 bits per color, so R: bits 20-29
        int g = ((rgb >> 10) & 0x3FF);                     // G: bits 10-19
        int b =          rgb & 0x3FF;                      // B: bits  0-9

        analogWrite(LED_RED,   r);                         // write it to the LED output pins
        analogWrite(LED_GREEN, g);
        analogWrite(LED_BLUE,  b);
      } else if (payload[0] == 'R') {                      // the browser sends an R when the rainbow effect is enabled
        rainbow = true;
      } else if (payload[0] == 'N') {                      // the browser sends an N when the rainbow effect is disabled
        rainbow = false;
      }
      break;
  }
}


 And, checking through Serial and the web browser, it does work, though it doesn’t have handles for sending data, or for knowing what to do when R tries to connect. But, the websocket is open and the HTML messages get sent, so we’re halfway there!

Aaand, it was about at this point that my research started revealing that there was no websocket style connection between RStudio and Arduino, so I started looking elsewhere. Someday I’ll get the inclination to build a websocket library for Arduino to RStudio, but that’s not today. 

