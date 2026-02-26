# Architecture — Engineering Case Study

## Problem Statement

I wanted a personal music server accessible from anywhere. Spotify's free tier has ads, and premium is ₹119/month that's ₹1,428/year for something I don't own. I had an old Xiaomi phone sitting unused. The question was: can I turn that into a production-grade (for personal use) streaming server with real security and near-zero running cost?

The answer is yes, but it took more problem-solving than expected.

---

## Constraints

- **Hardware:** one spare Xiaomi phone — no budget for new hardware
- **Cost:** as close to ₹0/month as possible
- **Security:** not just "it works" — proper auth, no exposed home IP
- **Accessibility:** reachable from outside the home network, any device, no VPN client installs
- **Reliability:** must survive phone screen off, background process killers, and Android's aggressive memory management

These constraints ruled out a lot of obvious approaches early.

---

## Why Not the Obvious Alternatives

**Raspberry Pi** — ₹6,000–8,000 for new hardware I didn't need when the phone was already sitting there. The phone has faster compute than a Pi 3, a built-in UPS (the battery), and eMMC storage instead of an SD card that might die in 6 months.

**VPS (DigitalOcean, etc.)** — reintroduces a monthly bill. Also means uploading my entire music library to someone else's server and keeping it synced. Defeats the point.

**Port forwarding** — technically works but exposes the home IP, requires DDNS for dynamic IPs (most Indian ISPs), and puts the service directly on the internet for bots to scan. Not acceptable.

**Tailscale** — solid option for purely personal use, but requires the Tailscale client on every device. Cloudflare Zero Trust gives a browser-native OTP flow with no client installation needed.

---

## Architecture

```
Client (anywhere)
    │
    ▼
Cloudflare Zero Trust  ← email allowlist + OTP
    │
    ▼
Cloudflare Tunnel      ← encrypted, outbound-only, no open ports
    │
    ▼
Xiaomi Phone (static LAN IP)
└── LineageOS (Android 16)
    └── Termux
        └── cloudflared  →  Navidrome :4533
                               └── /sdcard/Music
```

The key insight: `cloudflared` opens an **outbound** persistent connection to Cloudflare's edge. The phone never accepts an inbound connection. The home router has zero port forwarding rules. The home IP never appears in any DNS record.

All traffic enters through Cloudflare, gets checked against the access policy, then gets forwarded down the tunnel to Navidrome on localhost.

---

## Security Model

Two reasons I didn't just slap Navidrome on a port and call it done — bots constantly scan the internet for exposed services, and I didn't want my home IP in a DNS record.

**Layer 1 — No open ports.**
Nothing to scan, nothing to hit. The tunnel is outbound-only.

**Layer 2 — Cloudflare Zero Trust.**
Even if someone finds the domain, they hit an email OTP wall before touching the application. Only allowlisted emails get the OTP. This runs at Cloudflare's edge — Navidrome never sees unauthenticated requests.

**Layer 3 — Navidrome credentials.**
Valid username + password still required after passing Cloudflare.

**Layer 4 — Registration disabled.**
`EnableUserRegistration = false`. No self-signup. Accounts are admin-created only.

A bot finds nothing open. A person who finds the domain gets stopped at OTP. Someone on the allowlist without Navidrome credentials hits another wall.

---

## The Problems I Actually Had to Solve

These weren't in any tutorial. This is the part that took real time.

### Android's Phantom Process Killer

Android 12+ introduced aggressive background process killing that targets child processes of apps not in the foreground. Termux spawns Navidrome and cloudflared as child processes — Android was killing them unpredictably, sometimes within minutes of the screen turning off.

### Signal 31 (SIGSYS) on Android 16

This one took the most debugging. Navidrome kept crashing immediately on startup with no useful error — just `signal: killed`. Tracing it down: Android 16 tightened its `seccomp` filter (the kernel-level syscall allowlist). Navidrome binding to `0.0.0.0` triggered a syscall path that the new filter blocked, causing the kernel to send `SIGSYS`.

Fix: bind to `127.0.0.1` instead of `0.0.0.0` in the config. Since `cloudflared` connects to localhost anyway, this loses nothing for the tunnel flow. LAN access is handled separately via a specific interface bind.

### MIUI Battery Optimization

Even with developer mode and manual battery exemptions, MIUI would eventually kill Termux sessions. This isn't a settings problem — it's MIUI's architecture. The real fix was flashing LineageOS, which removes the entire aggressive battery management layer. Standard Android battery optimization is easy to exempt from after that.

---

## Deployment

No CI/CD, no containers — this is a phone. Deployment is:

1. Phone boots → static DHCP reservation gives it the same LAN IP every time
2. Termux:Boot runs `~/.termux/boot/start-services.sh`
3. Script starts Navidrome then cloudflared with a short delay between them
4. Cloudflared establishes the tunnel, Navidrome scans the music folder
5. Server is live at `music.yourdomain.com`

Updates to Navidrome: `pkg upgrade navidrome` in Termux, restart the process.

---

## Limitations

These are real — not disclaimers, just honest constraints of the setup.

- **Uptime tied to home internet.** If the ISP drops, the server is unreachable. Not a problem in practice for personal use, but worth knowing.
- **Throughput ceiling.** Home upload bandwidth limits concurrent streams. Fine for 1–2 users, not for more.
- **Android-specific complexity.** Signal 31, phantom process killer, battery management — none of these exist on a Pi or VPS. They cost real debugging time.
- **Cloudflare dependency.** Zero Trust and tunnels are free tier today. If that changes, the networking layer needs rethinking.
- **Storage bound to phone.** USB OTG works for expansion but adds fragility.

---

## Cost

At just ₹100, this project is 14x cheaper than Spotify Premium and over 60x cheaper than a VPS or Raspberry Pi setup in its first year. By leveraging your existing Wi-Fi and power, it eliminates the heavy entry costs and hardware fees of traditional alternatives.

---

## Lessons Learned

- **Android is not Linux.** It looks like Linux, Termux makes it feel like Linux, but seccomp filters and process management will remind you it isn't.
- **Cloudflare's free tier is genuinely powerful.** Zero Trust with email OTP, tunnels with automatic reconnection, DNS management — all free. Worth understanding properly.
- **Static DHCP reservation > static IP on device.** Let the router manage the IP space; don't hardcode it on the Android side where network config is awkward.
- **Outbound tunnel > port forwarding** for anything home-hosted. The security difference is significant and the complexity difference is minimal once you understand how `cloudflared` works.

---

## Future Improvements

- **Charge limiting** — keep the battery at 80% instead of 100% to reduce long-term degradation. Some LineageOS builds support this natively.
- **Automated health check + restart** — a cron job in Termux that pings Navidrome and restarts if it's down. Currently manual.
- **Music sync pipeline** — right now adding music means copying files manually. A lightweight rsync or Syncthing setup would make this cleaner.
- **Offline-first clients** — Symfonium with local sync would make the library accessible even when home internet is down.