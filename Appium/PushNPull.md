Assume the following Appium project structure:
```text
MobileAutomationFramework
│
├── apps
│   ├── MyInsuranceApp.apk
│   └── testdata.json
│
├── resources
│   ├── images
│   │   └── claim.jpg
│   └── documents
│       └── policy.pdf
│
├── reports
│
└── src
```
## 1. Push APK from Project Folder to Device (Copy APK from local project to device):
  adb push apps\MyInsuranceApp.apk /sdcard/Download/

Verify:
  adb shell ls /sdcard/Download/

output:
  MyInsuranceApp.apk

## 2. Push Test Data File to Device
  adb push apps\testdata.json /sdcard/TestData/

  Output:
    /sdcard/TestData/testdata.json

## 3. Push Image for Upload Testing
  adb push resources\images\claim.jpg /sdcard/Pictures/

  Verify:
    adb shell ls /sdcard/Pictures/


## 4. Push PDF for Document Upload Testing
  adb push resources\documents\policy.pdf /sdcard/Documents/

## 5. Pull Screenshot from Device to Project Reports Folder
Suppose Appium captured a screenshot on device:
  adb pull /sdcard/screenshots/failure.png reports\

Result:
  reports\failure.png

 ## 6. Pull Entire Folder
  adb pull /sdcard/screenshots reports\screenshots

## 7. Execute from Java

**Push**:
```java
String localFile =
        System.getProperty("user.dir")
        + "/resources/images/claim.jpg";

Runtime.getRuntime()
       .exec("adb push " + localFile
             + " /sdcard/Pictures/")
       .waitFor();
```

**Pull**
```java
Runtime.getRuntime()
       .exec("adb pull /sdcard/app_logs/app.log "
             + System.getProperty("user.dir")
             + "/reports/")
       .waitFor();
```

## Summary
---

- Upload Image to Device
  adb push resources\images\claim.jpg /sdcard/Pictures/

- Upload PDF to Device
  adb push resources\documents\policy.pdf /sdcard/Documents/

- Copy Device Screenshot to Framework Report Folder
  adb pull /sdcard/Download/screenshot.png reports\

- Copy Device Logcat Output
  adb pull /sdcard/logcat.txt reports\




