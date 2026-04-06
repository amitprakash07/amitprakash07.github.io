---
layout: post
title: "Project coursework: Installing Android device driver / ADB interface driver on Windows"
date: 2015-03-08
tags:
  - android
  - adb
  - device-driver
  - google-usb-driver
  - android-sdk
  - windows
source_wix: "https://amitprakash07.wixsite.com/home/post/2015/03/08/installing-android-device-driveradb-interface-driver-on-windows"
redirect_to_wix: true
---

**Android Debug Bridge (ADB)** is part of the Android SDK and is used for many common device and debugging tasks. Sometimes, even after installing the SDK, `adb` does not work until **drivers** and **PATH** are set correctly.

> **Note (2026):** Install paths and doc URLs have changed since this was written. Prefer installing **[Android Studio](https://developer.android.com/studio)** and use its SDK Manager; `platform-tools` (where `adb` lives) is still the same idea. Official references: **[OEM USB drivers](https://developer.android.com/studio/run/oem-usb)** · **[Google USB Driver (Windows)](https://developer.android.com/studio/run/win-usb)**.

First, install the Android SDK / Android Studio from Google’s developer site (originally linked from [Android SDK](http://developer.android.com/sdk/index.html) — use [Android Studio](https://developer.android.com/studio) today).

Add the folder that contains **`adb.exe`** to your Windows **PATH** (often `…/platform-tools` under the SDK, e.g. `C:\Users\<username>\AppData\Local\Android\sdk\platform-tools\`). Verify in **Command Prompt**:

```text
adb version
```

![adb version in Command Prompt]({{ '/assets/img/blog/2015/android-adb-driver-windows/01-adb-cmd.png' | relative_url }})

Connect your device and list devices:

```text
adb devices
```

![adb devices]({{ '/assets/img/blog/2015/android-adb-driver-windows/02-adb-devices.png' | relative_url }})

If the device does not appear, install the right **USB driver**: **Google USB Driver** for Nexus/Pixel-style dev devices, or your **manufacturer’s OEM driver**. See Google’s **[OEM USB drivers](https://developer.android.com/studio/run/oem-usb)** (older page: [OEM USB #InstallingDriver](http://developer.android.com/tools/extras/oem-usb.html#InstallingDriver)).

Sometimes the device shows in **Device Manager** but still not in `adb devices`:

![Device Manager — Galaxy Nexus]({{ '/assets/img/blog/2015/android-adb-driver-windows/03-galaxy-nexus-device-manager.jpg' | relative_url }})

Download the **Google USB Driver** package ([historical zip page](http://developer.android.com/sdk/win-usb.html) — current: [Install OEM USB drivers](https://developer.android.com/studio/run/win-usb)), unzip it somewhere on disk, then:

### Step 1

Open **Device Manager**, find your device, right‑click → **Update driver software**.

![Device Manager — update driver]({{ '/assets/img/blog/2015/android-adb-driver-windows/04-device-manager.png' | relative_url }})

### Step 2

Choose **Browse my computer for driver software**.

![Browse my computer for driver software]({{ '/assets/img/blog/2015/android-adb-driver-windows/05-driver-source-browse.png' | relative_url }})

### Step 3

Choose **Let me pick from a list of device drivers on my computer**.

![Let me pick from a list…]({{ '/assets/img/blog/2015/android-adb-driver-windows/06-driver-source-pick-list.png' | relative_url }})

### Step 4

Select **Android Composite ADB Interface** (or equivalent), then **Have Disk…**.

![Select Android Composite ADB Interface — Have Disk]({{ '/assets/img/blog/2015/android-adb-driver-windows/07-select-adb-composite.png' | relative_url }})

### Step 5

Point to the extracted Google USB driver folder and select **`android_winusb.inf`** (on older packages the name may match this pattern).

![Locate android_winusb.inf (1)]({{ '/assets/img/blog/2015/android-adb-driver-windows/08-locate-file-1.png' | relative_url }})

![Locate android_winusb.inf (2)]({{ '/assets/img/blog/2015/android-adb-driver-windows/09-locate-file-2.png' | relative_url }})

### Step 6

Click **Next**. If Windows shows a warning, confirm you want to install. Afterward, the device should appear in Device Manager and in **`adb devices`**.

![Installed — adb devices listing]({{ '/assets/img/blog/2015/android-adb-driver-windows/10-installed-device.png' | relative_url }})

`#AndroidDevices` `#DeviceDriver` `#GoogleUSBDriver` `#AndroidSDK`
