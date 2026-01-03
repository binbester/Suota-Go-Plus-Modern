# Suota Go Plus - Updated for Modern Android

This is an updated fork of the original [Suota-Go-Plus](https://github.com/Jesus805/Suota-Go-Plus) by **Jesus805**.

## Credits

All credit for the original application goes to **[Jesus805](https://github.com/Jesus805)**. This repository contains only the Client code with updates to make it compile and run on modern Android devices.

For the full project including firmware and documentation, please visit the original repository:
- **Original Repository**: https://github.com/Jesus805/Suota-Go-Plus
- **Reference Article**: https://tinyhack.com/2018/11/21/reverse-engineering-pokemon-go-plus/

## What This App Does

This Android application performs SUOTA (Software Update Over The Air) on Pokemon GO Plus devices using Dialog Semiconductor's DA14580 chip. It allows you to:

1. Patch the Pokemon GO Plus firmware
2. Extract device keys (Device Key and Blob)
3. Restore the original firmware

## Why This Fork Exists

The original project was built for older Android SDK versions and would not compile on modern development environments. This fork updates the project to:

- Target Android 13 (API 33)
- Use AndroidX libraries instead of deprecated Android Support libraries
- Add required Bluetooth permissions for Android 12+
- Fix SUOTA timing issues on newer Android devices

## Changes Made

### SDK & Framework Updates

| Component | Original | Updated |
|-----------|----------|---------|
| Target Android SDK | 27 (Android 8.1) | 33 (Android 13) |
| Target Framework | v8.1 | v13.0 |
| Xamarin.Forms | 3.4.0 | 5.0.0.2662 |
| Support Libraries | Xamarin.Android.Support v27 | Xamarin.AndroidX |

### AndroidManifest.xml Changes

Added new Bluetooth permissions required for Android 12+:
```xml
<uses-permission android:name="android.permission.BLUETOOTH_SCAN" />
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
```

Updated target SDK:
```xml
<uses-sdk android:minSdkVersion="21" android:targetSdkVersion="33" />
```

### Package Updates

Migrated from Android Support to AndroidX:
- `Xamarin.Google.Android.Material` (1.9.0.2)
- `Xamarin.AndroidX.AppCompat` (1.6.1.3)
- `Xamarin.AndroidX.Legacy.Support.V4` (1.0.0.21)
- `Xamarin.AndroidX.CardView` (1.0.0.21)
- `Xamarin.AndroidX.MediaRouter` (1.3.0)

### SUOTA Timing Fix

**Problem**: On some Android devices (tested on Samsung Galaxy S8+), the SUOTA process would fail partway through the firmware transfer (around block 69-82 out of 83 blocks).

**Root Cause**: The BLE write timing was slightly too slow, causing the Pokemon GO Plus to disconnect due to what appears to be a connection supervision timeout. The original 10ms delay between writes was on the edge of acceptable timing.

**Solution**: Reduced the delay constant from 10ms to 7ms:

```csharp
// Constants.cs
// Original:
public const int DelayMS = 10;

// Fixed:
public const int DelayMS = 7;
```

This minor timing adjustment allows the SUOTA process to complete successfully on modern Android devices.

### Interface Update

Added `skipPreDelay` parameter to `IBleManager.WriteCharacteristic` for potential future optimizations:

```csharp
Task WriteCharacteristic(GoPlus device, Guid characteristic, byte[] value, bool noResponse = false, bool skipPreDelay = false);
```

## Building the Project

### Requirements

- Visual Studio 2022 with Xamarin/MAUI workload
- Android SDK 33 or higher
- .NET Standard 2.0

### Steps

1. Clone this repository
2. Open `suota_pgp.sln` in Visual Studio
3. Restore NuGet packages
4. Build and deploy to your Android device

## Usage

1. Pair your Pokemon GO Plus with the Pokemon GO app first
2. Open this app and grant all required permissions
3. Select your paired Pokemon GO Plus device
4. Choose the firmware file (patch.img) from the PgpExtractor folder
5. Start the SUOTA process
6. Wait for completion - the process takes about 1-2 minutes

## Troubleshooting

### SUOTA fails at a specific block

If SUOTA consistently fails at a certain block number:

- **Fails early (blocks 1-30)**: The timing might be too fast. Try increasing `DelayMS` in `Constants.cs`
- **Fails late (blocks 60-82)**: The timing might be too slow. Try decreasing `DelayMS` in `Constants.cs`
- **Try these phone settings**:
  - Turn OFF WiFi (shares 2.4GHz antenna with BLE)
  - Keep phone plugged in (prevents BLE throttling)
  - Clear Bluetooth cache in Settings
  - Developer Options: Disable Bluetooth A2DP Hardware Offload (if available)

### App won't compile

Make sure you have:
- Latest Visual Studio 2022 updates
- Android SDK 33 installed
- All NuGet packages restored

## License

This project maintains the same license as the original repository. Please refer to the original [Suota-Go-Plus](https://github.com/Jesus805/Suota-Go-Plus) repository for license details.

## Disclaimer

This software is provided for educational and research purposes. Use at your own risk. Modifying device firmware may void warranties or cause permanent damage to your device.
