# Hardware

## The Device

An old Xiaomi phone I already owned. That's the whole hardware story — there was no purchase decision, just a question of whether it was capable enough.

It is. The phone has a faster CPU than a Raspberry Pi 3, eMMC storage (more reliable than the microSD that Pi setups typically use), built-in Wi-Fi, and a battery that acts as a UPS for short power cuts. The only real trade-off versus dedicated server hardware is Android's process management — which is a software problem, not a hardware one.

---

## Why This Beat Buying a Raspberry Pi

A Pi 4 costs ₹6,000–8,000 in India, plus microSD, power supply, and case. For a use case that an already-owned phone handles fine, that's an unnecessary spend. The Pi wins if you want native Linux and a bigger community for troubleshooting. The phone wins if it's sitting unused in a drawer.

One underrated advantage: the battery. A Pi with no UPS goes down on any power cut, even a brief one. The phone keeps running and `cloudflared` reconnects automatically when the internet comes back.

---

## What to Check If You're Replicating This

Not every Android phone will work. The things that actually matter:

- **Unlockable bootloader** — Xiaomi requires linking a MI account and waiting (72h to 30 days depending on model). Check your specific device before assuming it's unlockable.
- **LineageOS support** — verify at [lineageos.org/devices](https://lineageos.org/devices) before starting. No LineageOS build = stuck on MIUI = background process hell.
- **Android 16 target** — on Android 16, Navidrome installs directly via `pkg`. On older Android versions you'd need to download and run the binary manually, which is more fragile.
- **2GB+ RAM** — Navidrome sits around 80–100MB at rest. Termux adds overhead. 2GB is comfortable; 1GB will be tight.

---

## Battery

The phone runs plugged in 24/7, which degrades lithium batteries over time. In practice this doesn't matter much — even a degraded battery still provides short-term UPS, and the phone runs fine on wall power alone. If you want to slow degradation, some LineageOS builds support charge limiting (capping at 80%). Worth enabling if your build supports it, but not critical.

---

## Storage

Internal storage holds the music library. For most personal libraries this is fine — FLAC albums average around 300MB, so even 32GB gives you room for ~100 albums with space left over. If you need more, USB OTG works but adds another thing that can fail. I kept it simple with internal storage.