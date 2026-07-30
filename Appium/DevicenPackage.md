## Device onformation
----------------
- adb get-statе   (print device state)
- adb get-serialno (get the serial number)
- adb shell dumpsys iphonesybinfo (get the IMEI)
- adb shell netstat (list TCP connectivity)
- adb shell pwd (print current working directory)
- adb shell dumpsys battery (battery status)
- adb shell pm list features (list phone features)
- adb shell service list (list all services)
- adb shell dumpsys activity <package>/<activity> (activity info)
- adb shell ps (print process status)
- adb shell wm size (displays the current screen resolution)
- dumpsys window windows | grep -E 'mCurrentFocus|mFocusedApp' (print current app's opened activity)

## Package info
---------
- adb shell list packages (list package names)
- adb shell list packages -r (list package name + path to apks)
- adb shell list packages -3 (list third party package names)
- adb shell list packages -s (list only system packages)
- adb shell list packages -u (list package names + uninstalled)
- adb shell dumpsys package packages (list info on all apps)
- adb shell dump <name> (list info on one package)
- adb shell path <package> (path to the apk file)
