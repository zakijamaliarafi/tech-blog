---
heroImage: '/android-adb-advanced-tricks.svg'
title: 'Unlocking the Power of ADB: Advanced Android Tricks'
description: 'Discover advanced Android Debug Bridge (ADB) commands and tricks to control, debloat, and customize your Android device like a pro.'
pubDate: 'May 18 2026'
---

The Android Debug Bridge, almost universally known by its acronym **ADB**, is a remarkably versatile command-line tool that acts as a vital bridge of communication between your computer and your Android device. It operates as a client-server program, consisting of three main components: a client running on your computer that sends commands, a daemon (adbd) running as a background process on your Android device that executes the commands, and a server running on your computer that manages the communication between the client and the daemon. 

While developers rely on ADB extensively for installing applications, debugging code, and examining system logs during the app creation process, its utility extends far beyond the realm of software development. For Android power users, enthusiasts, and tinkerers, ADB represents a master key—a gateway to unlock hidden features, perform deep system modifications, remove carrier-installed bloatware, and execute advanced customizations that are simply impossible through the standard Android graphical user interface.

In this comprehensive guide, we will delve deep into the world of the Android Debug Bridge. We will explore how to properly set up the environment, and more importantly, we will uncover a wide array of advanced ADB tricks and commands that will empower you to take complete control over your Android device without necessarily needing root access.

## Setting Up Your ADB Environment

Before you can unleash the power of ADB, you must configure both your computer and your Android device to communicate with each other.

### 1. Enabling Developer Options and USB Debugging

The first step is to enable Developer Options on your Android device. This menu is hidden by default to prevent average users from accidentally altering critical system behaviors.
1. Open your device's **Settings** app.
2. Scroll down to **About phone** (or **About tablet**).
3. Locate the **Build number** entry.
4. Tap the **Build number** rapidly seven times in a row. You will see a small toast notification indicating that "You are now a developer!"
5. Go back to the main Settings menu, and navigate to **System** > **Developer options**.
6. Scroll down and toggle the switch for **USB debugging** to the "On" position.

### 2. Installing ADB Platform Tools on Your Computer

You do not need to install the massive Android Studio IDE just to use ADB. Google provides the standalone "SDK Platform-Tools" package for Windows, macOS, and Linux.

1. Download the SDK Platform-Tools ZIP file from the official Android developer website.
2. Extract the contents of the ZIP file to a convenient location on your computer (e.g., `C:\platform-tools` on Windows or `~/platform-tools` on macOS/Linux).
3. Open a terminal or command prompt window and navigate to the extracted directory.
4. Connect your Android device to your computer using a high-quality USB cable.
5. In the terminal, type the following command to start the ADB server and check for connected devices:
   ```bash
   adb devices
   ```
6. Look at your Android device's screen. You should see a prompt asking to "Allow USB debugging" from your computer's RSA key fingerprint. Check the box to "Always allow from this computer" and tap OK.
7. Run the `adb devices` command again. Your device should now be listed with the status `device`. If it says `unauthorized`, double-check the prompt on your phone's screen.

With the environment properly configured, you are now ready to execute advanced ADB commands.

## 1. Ultimate System Debloating (No Root Required)

When you purchase a new Android device, especially one tied to a specific cellular carrier or a manufacturer with a heavy custom skin (like Samsung's One UI or Xiaomi's MIUI), it often comes pre-loaded with dozens of unwanted applications. These "bloatware" apps consume valuable storage space, run unnecessary background processes, drain battery life, and clutter your app drawer. 

Normally, the Android system prevents you from uninstalling system apps. The "Uninstall" button is completely grayed out or hidden. However, using ADB, you can uninstall these packages for the current user, effectively erasing them from your day-to-day experience without requiring root privileges.

### Identifying the Target Package Name

To remove an app, you first need to know its exact package name (e.g., `com.facebook.system`, `com.samsung.android.bixby.agent`). You can list all installed packages using the package manager (`pm`) tool via the ADB shell:

```bash
# List all installed packages
adb shell pm list packages

# Search for a specific manufacturer's packages using grep
adb shell pm list packages | grep samsung
adb shell pm list packages | grep amazon
```

Alternatively, you can install an app like "App Inspector" from the Google Play Store to easily view the package name of any app currently installed on your device.

### Executing the Uninstall Command

Once you have identified the precise package name, you can issue the uninstall command. The crucial flags here are `-k` (which keeps the data and cache directories, though this is optional) and `--user 0` (which specifies that the app should be uninstalled for the primary user profile).

```bash
# Uninstall Bixby completely for the current user
adb shell pm uninstall -k --user 0 com.samsung.android.bixby.agent
adb shell pm uninstall -k --user 0 com.samsung.android.bixby.vision
```

*Crucial Warning: Exercise extreme caution when debloating system packages. While you cannot permanently delete the underlying APK files from the read-only system partition without root, removing critical framework components or core services (like the System UI, Package Installer, or Keyboard) can cause severe system instability, continuous crashing, or even trap your device in an unrecoverable boot loop requiring a factory reset.*

If you accidentally uninstall a critical app, you can reinstall it using the following command:
```bash
adb shell cmd package install-existing <package_name>
```

## 2. Granting Elevated and Hidden Permissions

Modern Android versions employ a strict permission model to protect user privacy. Apps must request permission to access the camera, location, or contacts, and the user must explicitly grant them via a pop-up dialog. 

However, there is an entire category of powerful, low-level system permissions that are deliberately hidden from the standard settings UI. Applications designed for automation (like Tasker, MacroDroid, or Automate), advanced battery monitoring (like BetterBatteryStats or GSAM Battery Monitor), or deep system customization often require these elevated permissions to function correctly. Without root access, the only way to grant these permissions is via ADB.

### Common Elevated Permissions

Here are a few examples of common elevated permissions and how to grant them to third-party applications:

**WRITE_SECURE_SETTINGS:**
This permission allows an application to modify hidden system settings, such as toggling airplane mode, changing animation scales, switching location modes, or modifying network preferences without user interaction.
```bash
adb shell pm grant com.joaomgcd.tasker android.permission.WRITE_SECURE_SETTINGS
```

**DUMP and PACKAGE_USAGE_STATS:**
These permissions are essential for battery monitoring apps. They allow the app to read low-level system logs, analyze CPU wakelocks, and determine exactly which background processes are draining the battery.
```bash
adb shell pm grant com.asksven.betterbatterystats android.permission.DUMP
adb shell pm grant com.asksven.betterbatterystats android.permission.PACKAGE_USAGE_STATS
```

**SYSTEM_ALERT_WINDOW:**
While this permission (Draw over other apps) is usually accessible via the settings menu, some OEM skins hide it for certain apps. ADB can force the grant.
```bash
adb shell pm grant net.dinglisch.android.taskerm android.permission.SYSTEM_ALERT_WINDOW
```

## 3. Manipulating Display Resolution and Density (DPI)

Android is designed to run on a vast array of screen sizes and resolutions. The Window Manager (`wm`) is the system service responsible for handling display properties. Using ADB, you can manually override the device's default resolution and display density (DPI).

### Modifying Screen Density (DPI)

The DPI (Dots Per Inch) determines the physical size of UI elements on the screen. Lowering the DPI value makes everything (text, icons, buttons) smaller, allowing you to fit significantly more content on the screen—a trick often referred to as "tablet mode" on larger phones. Conversely, increasing the DPI makes everything larger and easier to read.

```bash
# Check the current screen density
adb shell wm density

# Change DPI to 320 (making UI elements smaller)
adb shell wm density 320

# Reset DPI to the factory default
adb shell wm density reset
```

### Modifying Screen Resolution

You can also alter the actual rendering resolution of the device. This can be useful for developers testing how their UI scales, or for gamers trying to improve performance by rendering a demanding game at a lower resolution.

```bash
# Check the current physical and overridden screen size
adb shell wm size

# Change resolution to 1080x1920 (Width x Height)
adb shell wm size 1080x1920

# Reset resolution to the factory default
adb shell wm size reset
```
*Note: Changing the resolution dramatically can sometimes cause graphical glitches or misaligned touch input in certain applications.*

## 4. Capturing High-Quality Screen Recordings

While modern Android versions come with a built-in screen recorder accessible from the Quick Settings panel, the built-in tool often offers limited control over the recording parameters, defaulting to compressed bitrates and standard resolutions. 

ADB provides a powerful native screen recording utility (`screenrecord`) that grants you fine-grained control over the output, making it perfect for creating high-quality app demonstrations, tutorial videos, or capturing precise gameplay footage.

### Advanced Screen Recording Commands

```bash
# Record the screen at a high bitrate (8 Mbps) for a maximum of 3 minutes
adb shell screenrecord --bit-rate 8000000 --time-limit 180 /sdcard/demo.mp4

# Record the screen at a specific custom resolution
adb shell screenrecord --size 1280x720 /sdcard/demo.mp4

# Pull the recorded video file from the Android device to your computer
adb pull /sdcard/demo.mp4
```

*Note: The native `screenrecord` tool does not capture internal audio on older Android versions, though it may on newer builds depending on OEM implementations.*

## 5. Wireless ADB: Untethering Your Workflow

Historically, utilizing ADB required maintaining a physical USB connection between your phone and your computer. This could be cumbersome, especially if you were debugging an app that required physical movement or if you simply wanted to charge your phone via a wall outlet while working.

### Native Wireless Debugging (Android 11 and Above)

Starting with Android 11, Google introduced native, robust Wireless Debugging, completely eliminating the need for a USB cable even for the initial setup.

1. Ensure both your computer and your Android device are connected to the exact same Wi-Fi network.
2. On your Android device, go to **Developer options** and enable **Wireless debugging**.
3. Tap on the **Wireless debugging** text to open its specific menu.
4. Tap **Pair device with pairing code**. A pop-up will display a 6-digit Wi-Fi pairing code, along with an IP address and a specific port number.
5. On your computer's terminal, enter the pair command:
   ```bash
   adb pair <IP_ADDRESS>:<PORT>
   ```
6. When prompted, enter the 6-digit pairing code shown on your phone.
7. Once paired successfully, note the different IP address and port listed under the "IP address & Port" section in the Wireless debugging menu.
8. Connect to the device using that specific port:
   ```bash
   adb connect <IP_ADDRESS>:<PORT>
   ```

### Legacy Wireless TCP/IP Mode (Android 10 and Below)

For older devices running Android 10 or below, you must still use a USB cable to initiate the wireless connection.

1. Connect your device via USB cable and ensure standard USB debugging is working (`adb devices`).
2. Instruct the ADB daemon on the phone to restart and listen for TCP/IP connections on port 5555:
   ```bash
   adb tcpip 5555
   ```
3. Disconnect the physical USB cable.
4. Find your device's local IP address (usually found in Settings > Network & internet > Wi-Fi > network properties).
5. Connect wirelessly from your computer:
   ```bash
   adb connect <DEVICE_IP_ADDRESS>:5555
   ```

## Conclusion

The Android Debug Bridge is a phenomenally powerful utility that unlocks the true potential of the Android operating system. By mastering these advanced ADB commands, power users can transcend the limitations of the standard user interface. Whether you are ruthlessly debloating your system to reclaim battery life and storage, granting critical permissions to automation apps to streamline your daily workflows, tweaking the display density for a customized visual experience, or executing high-quality screen recordings over a wireless connection, ADB provides the absolute control necessary to truly make your Android device your own. Always proceed with caution, maintain backups of important data, and enjoy the unparalleled freedom of an unlocked Android experience.
