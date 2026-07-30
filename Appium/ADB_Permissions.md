
```text

adb install -r -g myapp.apk
grants all permissions as defined in AndroidManifest.xml

eg.

#1. Contacts
<uses-permission android:name="android.permission.READ_CONTACTS"/>
<uses-permission android:name="android.permission.WRITE_CONTACTS"/>
<uses-permission android:name="android.permission.GET_ACCOUNTS"/>

Allows:
Read contacts
Update contacts
Access account information

#2. Camera
<uses-permission android:name="android.permission.CAMERA"/>

Allows:
Capture photos
Record video

#3. Microphone
<uses-permission android:name="android.permission.RECORD_AUDIO"/>

Allows:
Voice recording
Audio capture

#4. Location
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>


Allows:
GPS location
Approximate location


#5. Storage (Older Android Versions)
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>


Allows:
Read files
Write files


#6. Phone
<uses-permission android:name="android.permission.READ_PHONE_STATE"/>
<uses-permission android:name="android.permission.CALL_PHONE"/>


Allows:
Read device phone state
Initiate calls

#7. SMS
<uses-permission android:name="android.permission.READ_SMS"/>
<uses-permission android:name="android.permission.SEND_SMS"/>
<uses-permission android:name="android.permission.RECEIVE_SMS"/>


Allows:
Read SMS
Send SMS
Receive SMS

#8. Calendar
<uses-permission android:name="android.permission.READ_CALENDAR"/>
<uses-permission android:name="android.permission.WRITE_CALENDAR"/>


Allows:
Read calendar entries
Create calendar events

#9. Nearby Devices / Bluetooth
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT"/>
<uses-permission android:name="android.permission.BLUETOOTH_SCAN"/>


Allows:
Connect to Bluetooth devices
Discover nearby devices


## Permissions NOT Granted by -g

The following require special handling and will not be automatically granted:

android.permission.SYSTEM_ALERT_WINDOW
android.permission.WRITE_SETTINGS
android.permission.PACKAGE_USAGE_STATS
android.permission.MANAGE_EXTERNAL_STORAGE
android.permission.BIND_ACCESSIBILITY_SERVICE
android.permission.WRITE_SECURE_SETTINGS


These are special/system permissions and usually require:
- Manual Settings configuration
- Root access
- ADB pm grant
- Device owner privileges
- System app privileges


## How to Verify Granted Permissions

Check all permissions for an installed app:
	adb shell dumpsys package com.mycompany.myapp

	Or:
	adb shell dumpsys package com.mycompany.myapp | findstr permission


# Check a specific permission:

adb shell pm check-permission \
com.mycompany.myapp \
android.permission.CAMERA


Benefits:

No permission popups during tests
Camera permission already granted
Location permission already granted
Notification permission granted (where applicable)
Faster and more stable Appium execution
```








