---
heroImage: '/android-bluetooth-low-energy-ble.svg'
title: 'Exploring Android Bluetooth Low Energy (BLE) APIs'
description: 'A deep dive into integrating hardware using Android Bluetooth Low Energy (BLE). Learn about scanning, GATT connections, and handling characteristics.'
pubDate: 'May 17 2026'
---

Bluetooth Low Energy (BLE) is the standard for modern IoT devices, wearables, and sensors. Integrating BLE in Android requires navigating a complex asynchronous API and stringent permission requirements.

## 1. Navigating BLE Permissions

Starting in Android 12 (API 31), BLE permissions became significantly stricter to protect user privacy.

You must declare the following in your `AndroidManifest.xml`:

```xml
<!-- For Android 12+ -->
<uses-permission android:name="android.permission.BLUETOOTH_SCAN"
                 android:usesPermissionFlags="neverForLocation" />
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />

<!-- Legacy permissions -->
<uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
<uses-permission android:name="android.permission.BLUETOOTH" />
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
```
*Note: If you don't use BLE scanning to derive physical location, use `neverForLocation`.*

You must request these permissions at runtime before attempting any BLE operation.

## 2. Scanning for Devices

To find BLE devices, use the `BluetoothLeScanner`. It is highly recommended to scan with a filter (e.g., a specific Service UUID) to save battery.

```kotlin
val bluetoothManager = context.getSystemService(Context.BLUETOOTH_SERVICE) as BluetoothManager
val scanner = bluetoothManager.adapter.bluetoothLeScanner

val filter = ScanFilter.Builder()
    .setServiceUuid(ParcelUuid(YOUR_SERVICE_UUID))
    .build()

val settings = ScanSettings.Builder()
    .setScanMode(ScanSettings.SCAN_MODE_LOW_LATENCY)
    .build()

val scanCallback = object : ScanCallback() {
    override fun onScanResult(callbackType: Int, result: ScanResult) {
        val device = result.device
        Log.d("BLE", "Found: ${device.name} - ${device.address}")
    }
}

// Start Scan
scanner.startScan(listOf(filter), settings, scanCallback)
```

## 3. Connecting to the GATT Server

Once a device is found, connect to its Generic Attribute Profile (GATT) server to read/write data.

```kotlin
val gatt = device.connectGatt(context, false, gattCallback)
```

All BLE operations are asynchronous. You communicate via the `BluetoothGattCallback`.

```kotlin
val gattCallback = object : BluetoothGattCallback() {
    override fun onConnectionStateChange(gatt: BluetoothGatt, status: Int, newState: Int) {
        if (newState == BluetoothProfile.STATE_CONNECTED) {
            // Connected! Now discover services.
            gatt.discoverServices()
        }
    }

    override fun onServicesDiscovered(gatt: BluetoothGatt, status: Int) {
        if (status == BluetoothGatt.GATT_SUCCESS) {
            // Services found! Now you can interact with Characteristics.
            val service = gatt.getService(YOUR_SERVICE_UUID)
            val charac = service.getCharacteristic(YOUR_CHARACTERISTIC_UUID)
            
            // Example: Read a value
            gatt.readCharacteristic(charac)
        }
    }

    override fun onCharacteristicRead(gatt: BluetoothGatt, charac: BluetoothGattCharacteristic, status: Int) {
        if (status == BluetoothGatt.GATT_SUCCESS) {
            val value = charac.value
            // Process sensor data...
        }
    }
}
```

## Best Practices

1. **Queuing Operations:** The Android BLE stack can only handle one operation at a time. If you write to a characteristic, you *must* wait for the `onCharacteristicWrite` callback before executing a read operation. Using a queue or a Kotlin Coroutine wrapper library is highly recommended.
2. **Timeouts:** Hardware is flaky. Always implement timeouts for connections and reads.
3. **Threading:** Callbacks run on a Binder thread. Ensure you push UI updates to the Main Thread.

Building BLE apps requires patience, but understanding the GATT asynchronous flow is the key to reliable hardware integration.
