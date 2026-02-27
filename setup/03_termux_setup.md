# Step 3 — Termux Setup

## Where to Get It

**Not the Play Store.** The Play Store version hasn't been updated in years and will cause package compatibility issues down the line. Two reliable sources:

- **F-Droid:** [f-droid.org/packages/com.termux](https://f-droid.org/packages/com.termux/)
- **GitHub Releases:** [github.com/termux/termux-app/releases](https://github.com/termux/termux-app/releases)

I went directly to the GitHub repo — the releases page lists APKs by architecture (`arm64-v8a` for most modern phones). Pick the right one for your device, sideload it, done. The repo also has solid documentation if you want to understand what you're installing.

## Initial Setup

```bash
pkg update && pkg upgrade -y
pkg install wget curl openssh nano -y
```

## One Thing Worth Knowing

Termux runs as a normal Android app — no root required. But it spawns child processes (your server daemons), and Android 12+ aggressively kills those. The next step deals with that directly.

