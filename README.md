 **ESP32-CAM Projects: Firebase, Google Drive, and Object Detection_**

This repository contains three distinct projects for the ESP32-CAM (AI Thinker) module, each demonstrating different functionalities: connecting to Firebase for data storage, uploading images to Google Drive, and performing object detection with an OLED display. All projects use the WiFiManager library for easy WiFi configuration and are designed to run on the AI Thinker ESP32-CAM with a 0.96-inch SSD1306 OLED display (128x64, I2C on GPIO 15 for SDA, GPIO 14 for SCL).

**Contents**

- Hardware Requirements
- Software Requirements
- WiFiManager Configuration
- Project 1: ESP32-CAM Connected to Firebase
- Project 2: ESP32-CAM Connected to Google Drive
- Project 3: ESP32-CAM with Object Detection and OLED Display
- Debugging Tips

**Hardware Requirements**

- ESP32-CAM (AI Thinker): Equipped with an OV2640 camera module.
- 0.96-inch SSD1306 OLED Display (128x64, I2C)
- FTDI Programmer: For Serial communication and code upload.
- Power Supply: 5V/external supply (USB may be insufficient).
- Computer with Arduino IDE: For uploading code and monitoring Serial output.

**Software Requirements**

- Arduino IDE (version 1.8.19 or later):
- Install the esp32 board support (version 2.0.4 or later) via Boards Manager.

*Board Settings:*
- Board: ESP32-CAM (Tools > Board > ESP32 Arduino > ESP32-CAM).
- Partition Scheme: Huge APP (3MB No OTA/1MB SPIFFS).
- Serial Monitor: 115200 baud.


**Libraries (install via Arduino IDE: Sketch > Include Library > Manage Libraries):** 

- WiFiManager (by tzapu, version ~0.15 or later): For WiFi configuration.
- Adafruit_GFX: For OLED graphics.
- Adafruit_SSD1306: For OLED control.
- For Firebase project: Firebase_ESP_Client (by Mobizt).
- For Google Drive project: WiFi and HTTPClient (included with ESP32 board support).
- Google Apps Script (for Google Drive project): Deployed web app URL.
- Firebase Project (for Firebase project): Database URL and API key.

*WiFiManager Configuration*

All projects use the WiFiManager library to simplify WiFi setup. Users can configure WiFi credentials (SSID and password) via a web-based configuration portal:

**First Boot or No Saved Credentials:**

The ESP32-CAM creates an access point (AP) named “ESP32-CAM-Config”.

OLED displays: “Connect to AP: ESP32-CAM-Config” and “IP: 192.168.4.1”.

Connect a device (phone/laptop) to “ESP32-CAM-Config” (no password).

Open a browser and navigate to http://192.168.4.1.

Enter your WiFi SSID and password, then save.

Credentials are saved to non-volatile storage for future connections.


*Subsequent Boots:*

Automatically connects to the saved WiFi network.

OLED shows “WiFi: Connected” and the IP address (e.g., “192.168.69.145”).

**Alternative Flashing with Carenuity:**
Users can flash their ESP32-CAM using Carenuity’s Solution Builder platform (visit Carenuity: ).
After flashing, edit WiFi credentials via the WiFiManager portal as described above.

*Reset WiFi Credentials:*
Add a reset button or include in code:wifiManager.resetSettings();
ESP.restart();

Press the ESP32-CAM’s reset button to restart and re-enter config mode if connection fails.


*Note:*

Ensure your WiFi network is 2.4GHz (ESP32 does not support 5GHz).
WiFiManager has a 3-minute timeout; if no connection is made, the ESP32 restarts.



##**Project 1: ESP32-CAM Connected to Firebase**
**Description**
This project captures images with the ESP32-CAM and uploads them to Firebase Realtime Database as base64-encoded strings. The OLED displays WiFi and upload status.

**Setup**

*Firebase Configuration:*

Create a Firebase project at console.firebase.google.com.

Set up a Realtime Database and note the Database URL (e.g., https://your-project-id.firebaseio.com).

Get the API Key from Project Settings > General.

Update the code with:#define FIREBASE_HOST "https://your-project-id.firebaseio.com/"

#define FIREBASE_AUTH "your-api-key"




*Code:*

Uses Firebase_ESP_Client to send base64-encoded images to /images in the database.

OLED displays: “WiFi: Connected”, “Uploading...”, “Upload OK”, or “Upload Failed”.

Serial Monitor shows: WiFi status, IP address, and upload results.


*Operation:*

Captures images every 5 seconds (configurable via delay(5000)).

Encodes images to base64 and uploads to Firebase.

Displays status on OLED and Serial Monitor (115200 baud).


Example Output

Serial Monitor:ESP32-CAM Firebase Image Upload
Starting WiFiManager...
WiFi connected
IP Address: 192.168.69.145
Camera initialized
Image uploaded to Firebase


OLED:
Setup: “WiFi: Connecting...”, “WiFi: Connected”, “Camera OK”.
Loop: “Uploading...”, “Upload OK” or “Upload Failed”.



##**Project 2: ESP32-CAM Connected to Google Drive**
**Description**
This project captures images with the ESP32-CAM and uploads them to Google Drive using a Google Apps Script. The OLED displays WiFi and upload status.

Setup

Google Apps Script:
Create a script at script.google.com:function doPost(e) {
  try {
    var data = Utilities.base64Decode(e.parameter.image);
    var folder = DriveApp.getFolderById("YOUR_FOLDER_ID");
    var blob = Utilities.newBlob(data, 'image/jpeg', 'ESP32-CAM_' + new Date().toISOString() + '.jpg');
    var file = folder.createFile(blob);
    return ContentService.createTextOutput("Success").setMimeType(ContentService.MimeType.TEXT);
  } catch (error) {
    return ContentService.createTextOutput("Error: " + error).setMimeType(ContentService.MimeType.TEXT);
  }
}


Replace "YOUR_FOLDER_ID" with your Google Drive folder ID (from the folder’s URL).
Deploy as a web app: Execute as: Me, Who has access: Anyone.
Copy the Web app URL (e.g., https://script.google.com/macros/s/<YOUR_SCRIPT_ID>/exec).


Code:

File: esp32cam_google_drive.ino (provided in previous response).
Update #define GOOGLE_SCRIPT_URL "YOUR_GOOGLE_SCRIPT_URL" with your web app URL.
Captures JPEG images, encodes to base64, and sends via HTTP POST.
OLED displays: “WiFi: Connected”, “Capturing...”, “Upload OK”, or “Upload Failed”.
Serial Monitor shows: WiFi status, IP address, and upload results.


Operation:
Captures images every 5 seconds (configurable via delay(5000)).
Sends base64-encoded images to Google Drive.
Images appear in your Google Drive folder (e.g., ESP32-CAM_2025-07-28T12:45:00Z.jpg).


Example Output

Serial Monitor:ESP32-CAM Image Upload to Google Drive
Starting WiFiManager...
WiFi connected
IP Address: 192.168.69.145
Camera initialized
Image uploaded successfully


OLED:
Setup: “WiFi: Connecting...”, “WiFi: Connected”, “Camera OK”.
Loop: “Capturing...”, “Upload OK” or “Upload Failed”.



##**Project 3: ESP32-CAM with Object Detection and OLED Display**

**Description**
This project uses the ESP32-CAM to perform object detection with an Edge Impulse model, detecting "yellow motor" and "white pods". The OLED displays only the label and confidence (e.g., yellow motor (0.85)), while the Serial Monitor shows full bounding box details.

**Setup**

Edge Impulse Model:
A modelthat has been trained in Edge Impulse with labels “yellow motor” and “white pods”.

Code:
Performs object detection on captured images.
OLED displays: yellow motor (0.85) or white pods (0.92) for high-confidence detections (confidence > 0.7), or “No objects detected”.
Serial Monitor shows: yellow motor (0.85) [x:100, y:50, w:80, h:60].

Operation:
Captures images every 5 seconds (configurable via delay(5000)).
Runs Edge Impulse model to detect objects.
Displays results on OLED (simplified) and Serial Monitor (detailed).



Requirements

Libraries: WiFiManager, Adafruit_GFX, Adafruit_SSD1306, Edge Impulse model library.
Edge Impulse model library (e.g., CynthiaK-project-1_inferencing.h).

Example Output

Serial Monitor:Edge Impulse Object Detection with WiFiManager
Model labels:
  yellow motor
  white pods
Starting WiFiManager...
WiFi connected
IP Address: 192.168.69.145
Camera initialized
Predictions (DSP: 50 ms., Classification: 20 ms., Anomaly: 0 ms.):
  yellow motor (0.85) [x:100, y:50, w:80, h:60]
  white pods (0.92) [x:150, y:70, w:50, h:90]


OLED:
Setup: “WiFi: Connecting...”, “WiFi: Connected”, “Camera OK”.
Loop: “yellow motor (0.85)”, “white pods (0.92)”, or “No objects detected”.



**Debugging Tips**

No Serial Output:
Test Serial:void setup() {
  Serial.begin(115200);
  delay(1000);
  Serial.println("Test Serial Output");
}
void loop() {
  Serial.println("Loop running...");
  delay(1000);
}


**Check FTDI, USB cable, or power supply (5V/500mA+).**


**WiFi Issues:**
If stuck in config mode, connect to “ESP32-CAM-Config” and visit http://192.168.4.1.
Ensure 2.4GHz WiFi network.
Use Carenuity’s Solution Builder platform to flash and configure credentials.


**OLED Issues:**
If blank, verify wiring (SDA → GPIO 15, SCL → GPIO 14).
Run I2C scanner:#include <Wire.h>
void setup() {
  Serial.begin(115200);
  Wire.begin(15, 14);
  Serial.println("I2C Scanner");
  for (uint8_t address = 1; address < 127; address++) {
    Wire.beginTransmission(address);
    if (Wire.endTransmission() == 0) {
      Serial.printf("I2C device found at address 0x%02X\n", address);
    }
  }
}
void loop() {}


Confirm OLED address (0x3C or 0x3D).


**Camera Issues:**
If “Camera init failed”, check OV2640 connections.
Try FRAMESIZE_QQVGA (160x120) if memory issues occur.


**Firebase Issues:**
Verify FIREBASE_HOST and FIREBASE_AUTH.
Check Firebase database rules allow write access.


**Google Drive Issues:**
Ensure Google Apps Script URL is correct and deployed with “Anyone” access.
Verify folder ID and script permissions.


**Object Detection Issues:**
If labels differ, check ei_classifier_inferencing_categories[] in the Edge Impulse library.
Share Serial output for debugging.



