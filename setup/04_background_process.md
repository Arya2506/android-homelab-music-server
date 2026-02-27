# Step 4 — Kill the Phantom Process Killer

## The Problem

Android 12+ has a mechanism called the Phantom Process Killer that aggressively terminates child processes of apps not in the foreground. Termux runs Navidrome and cloudflared as child processes — without this fix, Android kills them unpredictably the moment the screen turns off or Termux loses focus. Your server just silently dies.

## The Fix

Needs ADB from a PC. Enable Wireless Debugging on the phone first:

Settings → Developer Options → Wireless Debugging → enable → note the IP and port shown.

Then on PC:

```bash
adb connect <phone-ip>:<port>

adb shell "/system/bin/device_config set_sync_disabled_for_tests persistent"
adb shell "/system/bin/device_config put activity_manager max_phantom_processes 2147483647"
```

The second command sets the process limit to `INT_MAX` — effectively disabling the killer.

I ran these without fully understanding the internals at the time — they worked first try. If you want to go deeper on what `device_config` is actually doing, the AOSP source is the right place to look.

## Verify

Start something in Termux, lock the screen, leave it for 10+ minutes. Come back — it should still be running. If it is, you're good.

