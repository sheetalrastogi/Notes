## Detect SSD/USB storage device is connected to an Android device via OTG

# List Mounted Storage Devices
  adb shell df -h

**Look for entries such as**:
/storage/emulated
/storage/XXXX-XXXX
/mnt/media_rw/XXXX-XXXX

**An external SSD typically appears as**:
/storage/1234-5678

## Check USB Devices Attached
  adb shell dumpsys usb

**Look for**
USB Device Attached
Current Functions

## Java code
```java
Process process = Runtime.getRuntime()
        .exec("adb shell sm list-volumes public");

BufferedReader reader =
        new BufferedReader(
                new InputStreamReader(
                        process.getInputStream()));

String line;

boolean ssdAttached = false;

while ((line = reader.readLine()) != null) {

    if (line.contains("mounted")) {
        ssdAttached = true;
        break;
    }
}

System.out.println(
        ssdAttached
                ? "SSD/USB Storage Connected"
                : "SSD/USB Storage Not Connected");
```
