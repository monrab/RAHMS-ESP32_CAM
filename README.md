# RAHMS-ESP32_CAM
This repo contains the .ino sketch and the base64 library for the ESP32-CAM module. This sketch creates a server with a website to take photos, and the photos are also uploaded to Google Drive every hour.

## Prerequisites
The Google Apps [Script](https://github.com/gsampallo/esp32cam-gdrive) needs to be set up in the Google Drive and deployed. 
Tutorial how to use Google Apps Script [here](https://www.benlcollins.com/apps-script/google-apps-script-beginner-guide/)

```java
function doPost(e) {
  var data = Utilities.base64Decode(e.parameters.data);
  var folderName = Utilities.formatDate(new Date(), "GMT-3", "yyyyMMdd_HHmmss")+".jpg";
  var blob = Utilities.newBlob(data, e.parameters.mimetype, folderName );
  
  
   // Save the photo to Google Drive
  var folder, folders = DriveApp.getFoldersByName("RAHMS");
  if (folders.hasNext()) {
    folder = folders.next();
  } else {
    folder = DriveApp.createFolder("RAHMS");
  }
  var file = folder.createFile(blob); 
  return ContentService.createTextOutput('Complete')
}
```
Then the scriptID needs to be added to the main sketch and the ssid and password of the network needs to be set.
```c++
const char *ssid = "********";     //your network SSID
const char *password = "********"; //your network password
const char *myDomain = "script.google.com";
String myScript = "/macros/s/xxxxxxxxxxxx/exec"; //Replace with your own url
String myFilename = "filename=ESP32-CAM.jpg";
String mimeType = "&mimetype=image/jpeg";
String myImage = "&data=";
```

## loop()
Picture is taken everytime takeNewPhoto is true (user clicks on the website to take a photo).
On first run of the program initial picture is taken and stored in SPIFFS.
This loop also uploads a newly taken photo to Google Drive every hour (3600000ms).
```c++
void loop()
{
  if (takeNewPhoto)
  {
    capturePhotoSaveSpiffs();
    takeNewPhoto = false;
    delay(1000);
  }
  if (run_once)
  {
    sendCapturedImage();
    run_once = false;
    delay(1000);
  }
  if ((millis() - timer) > 3600000)
  {
    sendCapturedImage();
    timer = millis();
    delay(1000);
  }
  delay(100);
}
```


### Sources
[Server](https://randomnerdtutorials.com/esp32-cam-take-photo-display-web-server/), [Google drive](https://github.com/gsampallo/esp32cam-gdrive)
