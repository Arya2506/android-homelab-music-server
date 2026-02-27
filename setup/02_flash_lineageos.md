# Step 2 — Flash LineageOS

## Why LineageOS Over MIUI

MIUI's background process killing made running a persistent server impossible — even with every battery exemption enabled, it would eventually kill Termux sessions. LineageOS is a clean AOSP build with none of that. Full developer control, no bloatware, and security patches for devices Xiaomi stopped supporting.

## Before You Start

Two things to get right before touching the phone:

**Know your device codename.** Every Xiaomi device has a codename (e.g. `whyred`, `lavender`, `miatoll`). Settings → About Phone → you'll see it, or search "[your model] codename". You need this to find the correct LineageOS build — downloading the wrong one bricks the process.

**Follow the device-specific guide on LineageOS wiki exactly.** The general steps are the same across devices but the recovery method, partition layout, and flashing order can differ. I cross-referenced the LineageOS wiki with a YouTube walkthrough for my specific device — both pointed to the same steps, which gave me enough confidence to proceed.

## Steps

Boot into recovery (Power + Volume Up, or `adb reboot recovery` — check your device guide), flash the LineageOS zip, wipe data, reboot.

```
lineageos.org/devices → find your device → follow that guide
```

That's genuinely it. The wiki is well-maintained and device-specific. Don't improvise here.

## Verify

Settings → About Phone → should show LineageOS build number and Android 16. If it boots and shows this, you're done.

