---
title: Android Applications Reverse Engineering Cheat Sheet
updated: 2017-25-02 13:37
---

# Android Applications Reverse Engineering Cheatsheet

## Android Reversing 101

### Developer Options and USB Debugging

On an Android Device, the developer options can be activated by clicking 7 times on the *Build Version* located in the *Settings* menu.

Now that the developer options are available, USB debugging can be enabled via this menu.

**Note:** The following steps are likely to require USB debugging.

### Find/Pull/Install/Remove an APK on an Android Device

Listing the packages installed on the device:

```bash
adb shell pm list packages -f
```

Pulling an APK from the device:

```bash
adb pull /data/app/package-name/application.apk
```

Uninstalling an APK on the device:

```bash
adb uninstall mobi.skred.app
```

Installing an APK on the device:

```bash
adb install application.apk
```

### Make an APK Debuggable 

Unless configured otherwise, the release version of an APK is not debuggable. `apktool` ([https://github.com/iBotPeaches/Apktool](https://github.com/iBotPeaches/Apktool)) can be used to change that.

First the APK needs to be decoded:

```bash
apktool d -o app_directory application.apk
``` 

Once we have access to the files contained inside it, we will add `android:debuggable="true"` to the `application` tag in the `AndroidManifest.xml̀ file. Then the APK can be repacked:

```bash
apktool b -o application_debug.apk app_directory
``` 

### Sign an APK 

In order to make the device accept the newly built APK, we need to sign it by running the following commands.

```bash
keytool -genkey -v -keystore android_keystore.ks -alias alias_name -keyalg RSA -keysize 2048 -validity 10000
jarsigner -verbose -sigalg SHA1withRSA -digestalg SHA1 -keystore android_keystore.ks application_debug.apk alias_name
```

## Tools

 * **apktool** [https://github.com/iBotPeaches/Apktool](https://github.com/iBotPeaches/Apktool)
 * **keytool**
 * **jarsigner**

## Common errors 

### Insufficient permissions for device: udev requires plugdev group membership

```bash 

panda@bamboo ~/ % adb shell

error: insufficient permissions for device: udev requires plugdev group membership.
See [http://developer.android.com/tools/device.html] for more information.
```

This error occurs when USB debugging is not enabled on the device and/or if the USB connection is configured as "Charge this device". Enabling USB debugging and setting the USB connection to "Transfer files" fixed the issue. 

### No resource identifier found for attribute "*" in package 'android'

```bash
panda@bamboo ~/skred/test % java -jar apktool_2.3.1.jar b -o skred.apk skred

I: Using Apktool 2.3.1
I: Checking whether sources has changed...
I: Checking whether resources has changed...
I: Building resources...
W: /home/panda/skred/test/skred/res/layout-v26/abc_screen_toolbar.xml:5: error: No resource identifier found for attribute 'keyboardNavigationCluster' in package 'android'
W: 
Exception in thread "main" brut.androlib.AndrolibException: brut.androlib.AndrolibException: brut.common.BrutException: could not exec (exit code = 1): [/tmp/brut_util_Jar_6619145415774124321.tmp, p, --forced-package-id, 127, --min-sdk-version, 14, --target-sdk-version, 26, --version-code, 49, --version-name, 0.4.9, --no-version-vectors, -F, /tmp/APKTOOL4166809953551643161.tmp, -0, arsc, -0, arsc, -I, /home/panda/.local/share/apktool/framework/1.apk, -S, /home/panda/skred/test/skred/res, -M, /home/panda/skred/test/skred/AndroidManifest.xml]
	at brut.androlib.Androlib.buildResourcesFull(Androlib.java:492)
	at brut.androlib.Androlib.buildResources(Androlib.java:426)
	at brut.androlib.Androlib.build(Androlib.java:305)
	at brut.androlib.Androlib.build(Androlib.java:270)
	at brut.apktool.Main.cmdBuild(Main.java:227)
	at brut.apktool.Main.main(Main.java:75)
Caused by: brut.androlib.AndrolibException: brut.common.BrutException: could not exec (exit code = 1): [/tmp/brut_util_Jar_6619145415774124321.tmp, p, --forced-package-id, 127, --min-sdk-version, 14, --target-sdk-version, 26, --version-code, 49, --version-name, 0.4.9, --no-version-vectors, -F, /tmp/APKTOOL4166809953551643161.tmp, -0, arsc, -0, arsc, -I, /home/panda/.local/share/apktool/framework/1.apk, -S, /home/panda/skred/test/skred/res, -M, /home/panda/skred/test/skred/AndroidManifest.xml]
	at brut.androlib.res.AndrolibResources.aaptPackage(AndrolibResources.java:456)
	at brut.androlib.Androlib.buildResourcesFull(Androlib.java:478)
	... 5 more
Caused by: brut.common.BrutException: could not exec (exit code = 1): [/tmp/brut_util_Jar_6619145415774124321.tmp, p, --forced-package-id, 127, --min-sdk-version, 14, --target-sdk-version, 26, --version-code, 49, --version-name, 0.4.9, --no-version-vectors, -F, /tmp/APKTOOL4166809953551643161.tmp, -0, arsc, -0, arsc, -I, /home/panda/.local/share/apktool/framework/1.apk, -S, /home/panda/skred/test/skred/res, -M, /home/panda/skred/test/skred/AndroidManifest.xml]
	at brut.util.OS.exec(OS.java:95)
	at brut.androlib.res.AndrolibResources.aaptPackage(AndrolibResources.java:450)
	... 6 more

```

This error occured because of the file `/home/panda/.local/share/apktool/framework/1.apk` generated by `apktool`. Removing it fixed the issue.