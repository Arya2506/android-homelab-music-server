# Step 1 — Bootloader Unlock

## The Annoying Part First

Xiaomi locks their bootloaders and ties the unlock process to a MI account with a mandatory waiting period — anywhere from 72 hours to 30 days depending on your device and region. You can't skip it or work around it. I hit this on day one and it stalled the entire project while I waited.

Check your specific device's waiting period before you start. If it's 7+ days, start this step first and work on something else in the meantime.

---

## The MI Unlock Tool is Windows Only

No Linux, no Mac. If you're not on Windows, you'll need a VM or a friend's machine for this one step. Annoying but unavoidable.

---

## Steps

**1. Enable Developer Options**

Settings → About Phone → tap **MIUI Version** 7 times → "You are now a developer" toast appears.

Then: Settings → Additional Settings → Developer Options → enable **OEM Unlocking** and **USB Debugging**.

**2. Link MI Account and Start the Clock**

In Developer Options → find **Mi Unlock Status** → sign in with your MI account. This starts the mandatory waiting period. There's nothing else to do until it expires — Xiaomi's server tracks it.

**3. After the Wait — Run MI Unlock Tool**

Download [MI Unlock Tool](https://en.miui.com/unlock/download_en.html), boot the phone into Fastboot (Power + Volume Down), connect via USB, sign in with the same MI account, click Unlock.

That's it. The tool flashes the unlocked bootloader and wipes the device.

**4. Verify**

Boot back into Fastboot — you'll see an "unlocked" status on screen. From every normal boot going forward there'll be a warning screen about the unlocked bootloader. Expected, ignore it.

---

## What's Actually Different Now

The bootloader no longer checks that the OS is signed by Xiaomi. That's the only meaningful change — the phone is now ready to accept LineageOS. SafetyNet and Play Integrity will flag the device as modified, but since we're replacing the OS entirely, that doesn't matter.

