# Mi Fitness NoTrack (`mifitness-notrack`)

An Xposed/LSPosed/Vector module that completely neutralizes Xiaomi's `OneTrack` analytics framework and third-party telemetry inside Xiaomi Mi Fitness (`com.xiaomi.wearable`).

## Why This Exists
While hosts-based blockers (e.g. AdAway, NextDNS, Re-Malwack) prevent telemetry packets from leaving the device over the wire, the app continues to run heavy tracking routines in the background:
* Compiling hardware identifiers and app usage patterns.
* Encrypting payloads with AES-ECB.
* Storing events in local SQLite databases (`onetrack.db`).
* Waking CPU locks to retry failed network connections.

`mifitness-notrack` stops tracking **at the code level**, saving CPU and battery while keeping all watch-syncing, health monitoring, and notification mirroring functionality 100% operational.

## Update-Resilient Architecture
This module does not rely on fragile, obfuscated internal methods that break with every app update:
1. **Public SDK Kill Switch:** On initialization, calls Xiaomi's built-in `OneTrack.setDisable(true)` switch and forces `OneTrack.isDisable()` to return `true`.
2. **In-Process Drop:** Replaces `OneTrack.track(...)` and `EventManager` serialization methods with `DO_NOTHING`.
3. **Android Framework Choke Point (100% Version-Agnostic):** Hooks `android.database.sqlite.SQLiteDatabase.insertWithOnConflict` to drop any inserts into `events` or `monitor` tables. Because this is an Android OS framework class, app updates cannot rename or bypass it.
4. **Network Host Choke Point:** Hooks `java.net.InetAddress.getAllByName` to immediately throw `UnknownHostException` on Xiaomi tracking domains, aborting connection threads before socket creation.
5. **Third-Party Telemetry:** Stubs Google Firebase Analytics and Facebook AppEventsLogger.

## Installation
1. Download the latest APK from the [Releases](https://github.com/bgwastu/mifitness-notrack/releases) or GitHub Actions build artifacts.
2. Install the APK on your device:
   ```bash
   adb install -r mifitness-notrack.apk
   ```
3. Open your Xposed manager (Vector, LSPosed, etc.).
4. Enable the module. The scope is automatically set to `com.xiaomi.wearable`.
5. Force-stop Mi Fitness (`am force-stop com.xiaomi.wearable`) and relaunch.

## Scope
* Target package: `com.xiaomi.wearable` (Mi Fitness / Xiaomi Wear)

## License
MIT
