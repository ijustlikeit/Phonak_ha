# Phonak for Home Assistant

Custom integration for Phonak hearing aids. One entry covers
the pair; each aid is its own device with battery, volume (per ear), program
(switches both aids), mute and auto-on (both aids), tap control, and a
Bluetooth-streaming switch per paired phone.

## How it works
Home Assistant doesn't talk to the aids. The **Phonak Bridge** Android app built APK in `dist/`) does, from the phone that is already paired
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

The Pi's (or whatever you use for Home assistant) own Bluetooth isn't used any more: the aids only advertise rarely while
the phone is connected, and the Pi (again whatever you use for Home Assistant) can't resolve their private addresses
reliably. 

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
   **Allow running in the background**. On Samsung (or other Android)  phone also check Settings → Apps →
   Phonak Bridge → Battery → **Unrestricted**.

The app keeps a notification while it runs and restarts after a reboot.

## myPhonak
myPhonak keeps its own session with the aids open while it runs, even in the
background, and the aids then ignore the bridge ("Hearing aid did not accept
the session; is myPhonak running?"). Close myPhonak when you're done with it
(Recent apps → Close all, or App info → Force stop). To keep it from staying
connected in the background, set its battery use to **Restricted**. Its links
come back as soon as you open it again.
`Note:` Do not open or use myPhonak app while using this integration, I've had loss of connection and issues reconnecting.

## Tested with
`Phonak Audéo i90 Sphere by Sonova AG`
`Samsung Android phone S26`
`myPhonak app available on Google play`

My best guess is that it should work with any of the newer Sphere or Lumity lines of hearing aids.
I have no way of testing any other Phonak hearing aids nor any other Android phones with this integration. 

## Background
I used this approach (Phonak bridge on Android phone) because you can use this integration at all times when you have your phone with you.  
If you went the design route of having the integration handle the bluetooth connections you would need to be within range of your Home assistant bluetooth
controller. This would limit the usefulness severely.   
