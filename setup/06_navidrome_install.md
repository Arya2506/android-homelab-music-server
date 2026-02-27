# Step 6 — Navidrome Installation

## What I Tried First (And Why It Failed)

My first approach was the standard one — download the Navidrome binary from GitHub releases, make it executable, run it. This is how you'd install it on any Linux server.

```bash
# What I tried first
wget https://github.com/navidrome/navidrome/releases/download/v0.x.x/navidrome_linux_arm64.tar.gz
tar -xf navidrome_linux_arm64.tar.gz
chmod +x navidrome
./navidrome
```

It didn't start. The error wasn't obvious — just crashed on execution. After researching I found out why: Android 16 enforces `W^X` (write XOR execute) at the kernel level. A binary you download and mark executable violates that policy — Android blocks it from running without kernel-level changes to allow it. That's not a rabbit hole worth going down for a personal music server.

The cleaner solution: Navidrome is available directly as a Termux package on Android 16. The `pkg` version is already compiled and signed in a way that satisfies Android's execution policy. No binary juggling needed.

## The Actual Install

```bash
pkg install navidrome -y
```

That's it. One command, no manual binary handling.

## Configuration

```bash
mkdir -p ~/.config/navidrome
nano ~/.config/navidrome/navidrome.toml
```

```toml
MusicFolder = "/sdcard/Music"
DataFolder   = "/data/data/com.termux/files/home/.local/share/navidrome"
Port         = 4533
LogLevel     = "info"

# Important — see signal fix step for why 0.0.0.0 crashes on Android 16
Address = "127.0.0.1"

# No self-registration
EnableUserRegistration = false

#you can also add some more config options from navidrome docs
#such as  LogLevel, AuthRequestLimit, AuthWindowLength, DefaultTheme
#https://www.navidrome.org/docs/usage/configuration/options/ 
```

## First Run

```bash
navidrome --configfile ~/.config/navidrome/navidrome.toml
```

From another device on the same Wi-Fi, open `http://192.168.x.x:4533`. First boot prompts you to create an admin account — do that, then verify the music folder is picked up correctly.

If it crashes immediately on startup — that's the Signal 31 issue. Step 7 covers it.

