# Phonak Audéo Sphere for Home Assistant

Custom integration for Phonak Audéo Sphere (i90) hearing aids. One entry covers
the pair; each aid is its own device with battery, volume (per ear), program
(switches both aids), mute and auto-on (both aids), tap control, and a
Bluetooth-streaming switch per paired phone.

## How it works
Home Assistant doesn't talk to the aids. The **Phonak Bridge** Android app (in
`android/`, built APK in `dist/`) does, from the phone that is already paired
with them:

```
hearing aids ⇄ (Bluetooth LE, Sonova protocol) ⇄ Phonak Bridge on the phone ⇄ (HTTPS webhook) ⇄ Home Assistant
```

- Every 30 s the app POSTs to the integration's webhook. The reply carries the
  changes made in Home Assistant since the last check-in; the app runs them on
  the aids, reads them again and reports straight back. So a change takes up
  to one check-in interval to arrive.
- Every 5 minutes (or when you press **Read hearing aids now**) the app reads
  both aids and sends the values.
- It works wherever the phone is, as long as the phone can reach Home Assistant
  (use your external URL, e.g. Nabu Casa, for outside your home network).

The Pi's own Bluetooth isn't used any more: the aids only advertise rarely while
the phone is connected, and the Pi can't resolve their private addresses
reliably. LE bonds made on the Pi earlier are harmless and can stay.

## Install
1. Copy `dist/custom_components/phonak` to `/config/custom_components/phonak`
   and restart Home Assistant. Entries from 0.3 and earlier (Bluetooth) don't
   load any more: delete them.
2. Add **Phonak Hearing Aids**. The dialog that follows shows the webhook URL
   (also under **Configure** later). If it shows only `/api/webhook/…`, put your
   Home Assistant URL in front, e.g. `https://xxxx.ui.nabu.casa/api/webhook/…`.
3. Install `dist/PhonakBridge-0.1.2.apk` on the phone (copy it over and open it,
   or `adb install`). Allow nearby devices (Bluetooth) and notifications.
4. In the app: paste the webhook URL, check the hearing aids picked ("R-Phonak
   hearing aid" / "L-Phonak hearing aid"), tap **Save & start**, then
   **Allow running in the background**. On Samsung also check Settings → Apps →
   Phonak Bridge → Battery → **Unrestricted**.

The app keeps a notification while it runs and restarts after a reboot.

## myPhonak
myPhonak keeps its own session with the aids open while it runs, even in the
background, and the aids then ignore the bridge ("Hearing aid did not accept
the session; is myPhonak running?"). Close myPhonak when you're done with it
(Recent apps → Close all, or App info → Force stop). To keep it from staying
connected in the background, set its battery use to **Restricted**. Its links
come back as soon as you open it again.

## Testing the app without Home Assistant
`tools/bridge_test_server.py` is a stand-in webhook on the PC that prints the
reports and can hand out commands:

```
py -3.14 tools/bridge_test_server.py --port 8765
adb reverse tcp:8765 tcp:8765
# webhook URL in the app: http://127.0.0.1:8765/api/webhook/test
adb logcat -s PhonakBridge GattLink AidSession
```

## Building the app
JDK 17 and the Android SDK (platform 36, build-tools 37) in
`%LOCALAPPDATA%\Android\Sdk`; `android/local.properties` points there. Then
`cd android` and `gradlew assembleRelease testDebugUnitTest`. The protocol code
(`Protocol.kt`) is a port of `tools/protocol.py`; `ProtocolTest` checks it
against vectors from the Python version and the myPhonak capture.
