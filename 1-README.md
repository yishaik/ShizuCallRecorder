# Yishai Call Recorder

A Pixel-focused, non-root call recorder based on [ShizuCallRecorder](https://github.com/kitsumed/ShizuCallRecorder). It runs the bundled `scrcpy-server` as Android's shell user through Shizuku, captures the `voice-call` audio source, and stores recordings locally on the phone.

This fork defaults to:

- automatic recording for incoming and outgoing carrier calls;
- AAC audio in an MPEG-4 container with the `.m4a` extension;
- on-device storage chosen during setup;
- filenames in the form `{date}_{time}_{direction}_{phone_number}.m4a`;
- no automatic deletion.

Existing installs keep their saved settings. To apply these defaults to an upgraded install, turn on both automatic-recording switches and select AAC in Settings, or clear the app's data and run setup again.

## How it works

A normal Android app cannot directly capture the protected call-audio mix. This app uses Shizuku to start the bundled `scrcpy-server` with Android's shell privileges. The app asks scrcpy for the `voice-call` source and muxes its AAC stream into `.m4a` with Android's `MediaMuxer`. Call state is detected with Android's phone-state or `InCallService` paths, then the recording service starts and stops automatically.

This is intended for carrier phone calls. Third-party calling apps are not the main target.

## Pixel 9 Pro setup

### 1. Install and start Shizuku

1. Install Shizuku. The upstream project currently recommends [thedjchi's Shizuku fork](https://github.com/thedjchi/Shizuku), although the standard Shizuku app may also work.
2. On the Pixel, open **Settings > About phone** and tap **Build number** seven times to enable Developer options.
3. Open **Settings > System > Developer options**.
4. Turn on **Wireless debugging**. The phone must be connected to Wi-Fi for Android's wireless-debugging pairing flow.
5. Open Shizuku and choose **Start via Wireless debugging**.
6. Choose **Pairing**, then **Developer options**. Under Wireless debugging, choose **Pair device with pairing code**.
7. Enter the pairing code in Shizuku, return to Shizuku, and press **Start** if it has not started automatically.
8. Confirm Shizuku reports that its service is running.

After a reboot, Shizuku normally has to be started again. The app cannot record through Shizuku while its service is stopped.

### 2. Install and configure this app

1. Build and install the debug APK, or download an APK produced by this repository's GitHub Actions.
2. Open the app and accept the legal notice only after checking that call recording is lawful where you are and for the people you call.
3. When Shizuku asks, allow this app to use Shizuku.
4. Grant the requested **Phone**, **Call logs**, **Contacts**, and **Notifications** permissions. Contacts are used for names and call filtering; call logs/phone state help identify calls and direction.
5. Choose a local recording folder with Android's folder picker. A practical choice is `Music/Call Recordings`. This grants the app ongoing access to that folder through Android's Storage Access Framework.
6. In app settings, confirm:
   - **Automatically record incoming calls** is on.
   - **Automatically record outgoing calls** is on.
   - **Audio source** is `voice-call`.
   - **Audio codec** is **AAC (.m4a)**.
   - no contact or anonymous-number filter is excluding calls.
7. On the app's system settings page, set battery use to **Unrestricted** if Android offers that choice. Pixel background limits can otherwise delay call detection or stop a recording service.

### 3. Test before relying on it

1. Keep Shizuku running.
2. Make one outgoing test call and answer one incoming test call.
3. Test the routes you use: earpiece, speakerphone, and Bluetooth.
4. Open the recordings screen, play both files, and confirm that both people are audible.
5. Reboot once, restart Shizuku, and repeat a test call.

## Build

The project is Kotlin with Gradle Kotlin DSL and Jetpack Compose. It requires Android Studio / an Android SDK and JDK 17.

```bash
# The upstream source snapshot currently does not include gradlew/gradlew.bat.
# Use Android Studio's Gradle integration or a locally installed compatible Gradle.
gradle assembleDebug
```

The APK is written under `app/build/outputs/apk/` when the build succeeds.

## Important limitations

- **Device testing is required.** This fork has been code-reviewed but has not been installed or call-tested on a Pixel 9 Pro in the environment that produced these changes.
- The app depends on hidden Android APIs, Shizuku, scrcpy-server, Telecom behavior, and OEM audio routing. A monthly Pixel update or a new Android release can break any of those points.
- Shizuku must be running. Android typically stops its ADB-backed service after reboot.
- Automatic detection can be affected by permissions, battery restrictions, and Android background-execution rules.
- Android may redact or deliver the phone number late, so a filename can contain an unknown/anonymous value even when audio recording works.
- Bluetooth/headset behavior must be checked on the actual device. The upstream design claims both-side capture, but no route should be assumed reliable until a playback test confirms it.
- This is not a covert recorder. Follow local notice and consent laws.

## Changes in this fork

- 2026-09-22: changed fresh-install defaults to auto-record incoming and outgoing calls, AAC/M4A, and date/time/direction/number filenames; replaced setup documentation with a Pixel-oriented flow.

## License and attribution

This fork is licensed under **GNU GPL v3 or later**, including the upstream project's Section 7 additional terms. Modified versions must keep copyright and license notices, identify modifications, provide corresponding source to recipients of binaries, and remain under the same GPL terms. See [LICENSE](LICENSE).

Copyright (C) 2026-present kitsumed (Med) and contributors. Fork modifications copyright (C) 2026 Yishai Kaminsky.

The bundled scrcpy-server remains subject to its own upstream license and notices.
