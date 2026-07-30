```text
## Using ADB Commands to Minimize all open Apps

Send HOME:	adb shell input keyevent KEYCODE_HOME


Open Recents:	adb shell input keyevent KEYCODE_APP_SWITCH


Force Stop an App:	adb shell am force-stop com.example.app


# Example:

	adb shell am force-stop com.android.chrome


## Execute ADB from Selenium/Appium Java

```java

ProcessBuilder pb = new ProcessBuilder("cmd.exe", "/c", "adb shell input keyevent KEYCODE_HOME");

pb.start().waitFor();
```

## Best Practice Before Starting an Appium Session

If your goal is to ensure the emulator is in a clean state:
```java

ProcessBuilder pb = new ProcessBuilder(
        "cmd.exe",
        "/c",
        "adb shell input keyevent KEYCODE_HOME");

pb.start().waitFor();
```

# or after driver creation:
```java

AndroidDriver driver = new AndroidDriver(serverUrl, options);

driver.pressKey(new KeyEvent(AndroidKey.HOME));

```




Other ADB Commands:

1. Clear App Data and Cache (Settings → Apps → Storage → Clear Data)
	adb shell pm clear com.example.app

	adb shell pm clear com.android.chrome


2. Force Stop an App (Kills the application process.)
	adb shell am force-stop com.example.app

	adb shell am force-stop com.android.chrome

3. Restart an App (force stop and launch again)
	adb shell am force-stop com.example.app
	adb shell monkey -p com.example.app -c android.intent.category.LAUNCHER 1


	adb shell am force-stop com.android.chrome
	adb shell monkey -p com.android.chrome -c android.intent.category.LAUNCHER 1
	

4. Start an App Using Activity Name
	adb shell am start -n package/activity

	adb shell am start -n com.android.settings/.Settings

5. Kill All Background Processes
	adb shell am kill-all


6. Send App to Background (Home Screen)
	adb shell input keyevent KEYCODE_HOME

	or
	adb shell input keyevent 3


7. Open Recent Apps
	adb shell input keyevent KEYCODE_APP_SWITCH

	or
	adb shell input keyevent 187

8. Wake Up Device
	adb shell input keyevent KEYCODE_WAKEUP

	or
	adb shell input keyevent 224

9. Unlock Device
	adb shell input keyevent 82


10. Reboot Emulator
	adb reboot


11. Get Installed Packages
	adb shell pm list packages

	Filter for a specific app:
	adb shell pm list packages | findstr chrome


12. Get App Activity Information
	adb shell dumpsys package com.example.app

	Find launcher activity:
	adb shell cmd package resolve-activity --brief com.example.app

13. Uninstall and Reinstall App

Uninstall:		adb uninstall com.example.app
Install:		adb install app.apk
Replace existing:	adb install -r app.apk


14. Reset App State (Recommended for Appium)

	adb shell am force-stop com.example.app
	adb shell pm clear com.example.app
	adb shell monkey -p com.example.app -c android.intent.category.LAUNCHER 1


15. Discover Current Foreground App
	adb shell dumpsys window | findstr mCurrentFocus

	or
	adb shell dumpsys activity activities | findstr mResumedActivity


16. Get Launchable Activity for Any App
	adb shell cmd package resolve-activity --brief com.android.chrome

	Output:
	com.android.chrome/com.google.android.apps.chrome.Main

and launch app:
	Then launch:  adb shell am start -n com.android.chrome/com.google.android.apps.chrome.Main


Appium:		driver.activateApp("com.android.chrome");  //brings an app to front


17. ADB to Get Running Apps
	adb shell dumpsys activity processes

	Or:
	adb shell dumpsys activity activities


18. Check Developer Options Status:
	adb shell settings get global development_settings_enabled

	Output:  1
	1 = Developer Options Enabled
	0 = Developer Options Disabled


19. Check USB Debugging Status
	adb shell settings get global adb_enabled

	Output: 0
	1 = USB Debugging Enabled
	0 = USB Debugging Disabled


20. Stay Awake
	adb shell settings get global stay_on_while_plugged_in


21. List of ADB Commands to Check Before Appium Execution
	adb devices
	adb shell settings get global development_settings_enabled
	adb shell settings get global adb_enabled
	adb shell getprop ro.build.version.release


These commands verify:

- Device connected
- Developer Options enabled
- USB Debugging enabled
- Android version available for testing.


22. Get device android version
	adb shell getprop ro.build.version.release 


23. App install
	adb -e install path/to/app.apk

	where:
	-d                        - directs command to the only connected USB device...
	-e                        - directs command to the only running emulator...
	-s <serial number>        ...
	-p <product name or path> …



24. Install the given app on all connected devices
	adb devices | tail -n +2 | cut -sf 1 | xargs -IX adb -s X install -r com.myAppPackage


25. Uninstalling app from device
	adb uninstall com.myAppPackage

	adb uninstall <app .apk name>

	adb uninstall -k <app .apk name> -> "Uninstall .apk withour deleting data"

	Clear User Data:  
	adb shell pm clear com.mycompany.myapp

26. Most Common Mobile Automation Pattern

Fresh install every test run:
	adb uninstall com.mycompany.myapp
	adb install -r -g MyApp.apk

or in Appium:
	driver.removeApp("com.mycompany.myapp");
	driver.installApp(apkPath);


27. To clear both cache and user data for an Android app using ADB, use:
	adb shell pm clear com.mycompany.myapp


pm clear removes:
- App cache
- Shared Preferences
- SQLite databases
- Internal storage files
- Login sessions/tokens
- App settings
- User-generated data stored in the app sandbox

It is equivalent to:
	Settings → Apps → <App> → Storage → Clear Storage / Clear Data


28. Stop App Before Clearing Data
	adb shell am force-stop com.mycompany.myapp
	adb shell pm clear com.mycompany.myapp

29. Clear Data and Restart App
	adb shell am force-stop com.mycompany.myapp
	adb shell pm clear com.mycompany.myapp
	adb shell monkey -p com.mycompany.myapp -c android.intent.category.LAUNCHER 1

30. Verify Data Was Cleared

	Check app state:
	adb shell dumpsys package com.mycompany.myapp

31. Difference Between pm clear and uninstall
	adb shell pm clear com.mycompany.myapp

	- Keeps app installed
	- Deletes cache and user data

	adb uninstall com.mycompany.myapp

	- Removes app
	- Removes cache and user data
	- Requires reinstallation before use

32. Prevent Device Sleep During Test Execution
	adb shell svc power stayon true

	Restore back:
	adb shell svc power stayon false

33. Turn screen on
	adb shell input keyevent 26


34. Simulate User Activity Periodically
	adb shell input keyevent KEYCODE_HOME

	or
	adb shell input tap 100 100

35. ADB Command to configure device before testing:
(Complete Pre-Test Device Preparation)

adb shell svc power stayon true
adb shell settings put system screen_off_timeout 86400000
adb shell settings put global window_animation_scale 0
adb shell settings put global transition_animation_scale 0
adb shell settings put global animator_duration_scale 0
adb shell input keyevent KEYCODE_WAKEUP
adb shell input keyevent 82


Instead of ADB, Appium can keep the device awake:

UiAutomator2Options options = new UiAutomator2Options();

options.setCapability("autoGrantPermissions", true);
options.setCapability("disableWindowAnimation", true);
options.setCapability("unlockType", "pin");


36. Network connectivity checks:

1. Check Device IP Address
adb shell ip addr show wlan0


or

adb shell ifconfig wlan0


Example output:

inet 192.168.1.105/24

2. Get Only the IP Address
adb shell ip route


Example:

192.168.1.0/24 dev wlan0 proto kernel scope link src 192.168.1.105


The value after src is the device IP.

3. Check Active Network Interfaces
adb shell ip link show


Typical interfaces:

wlan0   --> Wi-Fi
rmnet0  --> Mobile Data
lo      --> Loopback

4. Check Wi-Fi Connection Status
adb shell dumpsys wifi | findstr "Wi-Fi is"


Example:

Wi-Fi is enabled
Wi-Fi is connected

5. Get Connected Wi-Fi Network (SSID)
adb shell dumpsys wifi | findstr SSID


Example:

SSID: MyHomeWifi

6. Check Mobile Data Status
adb shell dumpsys telephony.registry


Look for:

mDataConnectionState=2


Values:

0 = Disconnected
1 = Connecting
2 = Connected
3 = Suspended

7. Check Current Network Type
adb shell getprop gsm.network.type


Example:

LTE


or

NR


(5G)

8. Check Connectivity Manager Status
adb shell dumpsys connectivity


Useful information:

NetworkAgentInfo
NetworkCapabilities
TRANSPORT_WIFI
TRANSPORT_CELLULAR

9. Identify Whether Wi-Fi or Mobile Data Is Active
adb shell dumpsys connectivity | findstr NetworkAgentInfo


Look for:

TRANSPORT_WIFI


or

TRANSPORT_CELLULAR

10. Check Internet Reachability
adb shell ping -c 4 google.com


Example:

64 bytes from ...


If successful, internet connectivity is available.

11. Check DNS Configuration
adb shell getprop | findstr dns


Example:

[net.dns1]: [8.8.8.8]
[net.dns2]: [8.8.4.4]

12. Check Airplane Mode
adb shell settings get global airplane_mode_on


Values:

0 = OFF
1 = ON

13. Check Wi-Fi Enabled/Disabled
adb shell settings get global wifi_on


Values:

0 = Disabled
1 = Enabled

14. Check Mobile Data Enabled
adb shell settings get global mobile_data


Values:

0 = Disabled
1 = Enabled

15. Get Complete Network Diagnostics
adb shell dumpsys connectivity


This is the most comprehensive command and provides:

Active network
Network type (Wi-Fi/Mobile)
DNS servers
Routes
Network capabilities
Connectivity state
Useful Appium Pre-Check

To verify the device has a network connection before executing tests:

adb shell ip route
adb shell dumpsys connectivity
adb shell ping -c 1 google.com


Or in Java:

Process process = Runtime.getRuntime()
    .exec("adb shell ping -c 1 google.com");

int exitCode = process.waitFor();

if (exitCode == 0) {
    System.out.println("Internet Available");
} else {
    System.out.println("Internet Not Available");
}

---------------------------------

37. Appium frameworks to validate network availability before running API-dependent mobile tests.

Process process = Runtime.getRuntime().exec("adb shell ping -c 1 google.com");

int exitCode = process.waitFor();

if (exitCode == 0) {
    System.out.println("Internet Available");
} else {
    System.out.println("Internet Not Available");
}


38. Get Complete Network Diagnostics via ADB

	adb shell dumpsys connectivity

This is the most comprehensive command and provides:
- Active network
- Network type (Wi-Fi/Mobile)
- DNS servers
- Routes
- Network capabilities
- Connectivity state


39. list all user-installed (third-party) apps on an Android device/emulator:
	adb shell pm list packages -3

    Show User Apps with APK Path
	adb shell pm list packages -3 -f

    Count User-Installed Apps
	adb shell pm list packages -3 | find /c "package:"

    List System Apps
	adb shell pm list packages -s

    List Disabled Apps
	adb shell pm list packages -d

    List Enabled Apps
	adb shell pm list packages -e

    Get currently foreground app:
	adb shell dumpsys activity activities | findstr mResumedActivity

    Bring your AUT to foreground:
	adb shell monkey -p com.mycompany.myapp -c android.intent.category.LAUNCHER 1

40. Power on / off a device / emulator:
	adb shell input keyevent 26

	or
	// Press POWER button
	driver.pressKey(new KeyEvent(AndroidKey.POWER));

41. Better Way to Wake Up Device
    Instead of toggling the power button, check the screen state first:

	if (!driver.isDeviceLocked()) {
    		System.out.println("Already unlocked");
	} else {
    		driver.pressKey(new KeyEvent(AndroidKey.POWER));
    		driver.unlockDevice();
	}


42. Frequently Used Android Keys:

	import io.appium.java_client.android.nativekey.AndroidKey;
	import io.appium.java_client.android.nativekey.KeyEvent;

	driver.pressKey(new KeyEvent(AndroidKey.APP_SWITCH)); // Recent Apps
	driver.pressKey(new KeyEvent(AndroidKey.NOTIFICATION)); // Notification Panel
	driver.pressKey(new KeyEvent(AndroidKey.SEARCH)); // Search
	driver.pressKey(new KeyEvent(AndroidKey.VOLUME_UP)); // Volume Up
	driver.pressKey(new KeyEvent(AndroidKey.VOLUME_DOWN)); // Volume Down
	driver.pressKey(new KeyEvent(AndroidKey.MUTE)); // Mute
	driver.pressKey(new KeyEvent(AndroidKey.MENU)); // Menu
	driver.pressKey(new KeyEvent(AndroidKey.WAKEUP)); // Wake Device
	driver.pressKey(new KeyEvent(AndroidKey.ENTER));  //Enter
	driver.pressKey(new KeyEvent(AndroidKey.DEL));
	driver.pressKey(new KeyEvent(AndroidKey.BRIGHTNESS_DOWN));
	driver.pressKey(new KeyEvent(AndroidKey.BRIGHTNESS_UP));


43. Appium code to check an app status and bring it to front:

```java
import io.appium.java_client.android.AndroidDriver;
import io.appium.java_client.appmanagement.ApplicationState;

public class AppSwitchExample {

    public static void main(String[] args) {

        String packageName = "com.mycompany.myapp";

        ApplicationState state =
                driver.queryAppState(packageName);

        if (state == ApplicationState.RUNNING_IN_BACKGROUND
                || state == ApplicationState.RUNNING_IN_BACKGROUND_SUSPENDED) {

            driver.activateApp(packageName);

            System.out.println("App brought to foreground");
        }
    }
}

```

Alternative Using ADB:	adb shell monkey -p com.mycompany.myapp -c android.intent.category.LAUNCHER 1


Useful Appium Methods:
- driver.activateApp(packageName);      // Foreground app
- driver.terminateApp(packageName);     // Kill app
- driver.installApp(apkPath);           // Install app
- driver.removeApp(packageName);        // Uninstall app
- driver.queryAppState(packageName);    // Check app state
- driver.runAppInBackground(Duration.ofSeconds(10));



## Get a list of active apps in Appium and close all except com.mycompany.myapp in Appium Java




Above sequence of commands:
- Stops the app
- Clears data/cache
- Launches a fresh instance


Java Utility Method
public static void restartApp(String packageName) {

    try {

        Runtime.getRuntime().exec(
                "adb shell am force-stop " + packageName)
                .waitFor();

        Runtime.getRuntime().exec(
                "adb shell pm clear " + packageName)
                .waitFor();

        Runtime.getRuntime().exec(
                "adb shell monkey -p "
                        + packageName
                        + " -c android.intent.category.LAUNCHER 1")
                .waitFor();

    } catch (Exception e) {
        e.printStackTrace();
    }
}


Usage:

restartApp("com.android.chrome");


Useful ADB Commands:
adb devices
adb devices -l
adb shell getprop ro.build.version.release
adb shell getprop ro.product.model
adb shell dumpsys battery
adb shell screencap -p /sdcard/screenshot.png
adb pull /sdcard/screenshot.png
adb logcat
adb logcat > device.log
adb bugreport
adb shell settings put global animator_duration_scale 0
adb shell settings put global transition_animation_scale 0
adb shell settings put global window_animation_scale 0




```
