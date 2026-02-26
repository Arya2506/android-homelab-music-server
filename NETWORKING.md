# Networking

## The Core Decision — Tunnel vs. Port Forwarding

The traditional approach to exposing a home server is port forwarding: open a port on the router, point it at your device, done. I didn't do that, for a few reasons.

Most Indian ISPs assign dynamic public IPs — meaning your home IP changes, and you need a DDNS service to keep a domain pointed at it. More importantly, an open port is directly reachable by anyone on the internet. Bots scan the entire IPv4 space constantly; within hours of opening a port, you'll see login attempts in your logs. For a personal server that's supposed to be low-maintenance, that's not a starting point I wanted.

Cloudflare Tunnel flips the model. Instead of the internet connecting *in* to your server, your server connects *out* to Cloudflare. The `cloudflared` daemon opens a persistent outbound WebSocket over port 443 (standard HTTPS — nothing to unblock) to Cloudflare's edge. Your home IP never appears in any DNS record. There's no open port. The router is untouched.

---

## How the Tunnel Actually Works

```
Client → Cloudflare Edge → [persistent outbound tunnel] → cloudflared → Navidrome :4533
```

The phone holds one end of that tunnel. Cloudflare holds the other. When a request comes in for `music.yourdomain.com`, Cloudflare routes it down the already-open connection to `cloudflared` on the phone, which forwards it to Navidrome on localhost. The response travels back the same way.

The DNS side is clean too — Cloudflare automatically creates a `CNAME` pointing your subdomain to a `*.cfargotunnel.com` address. Nobody doing a DNS lookup sees a home IP. They see Cloudflare's infrastructure.

---

## Zero Trust — Why OTP Over a Password

Zero Trust Access sits in front of the tunnel. Before a request ever reaches Navidrome, Cloudflare checks the access policy at the edge.

The policy I set up: email allowlist + one-time PIN. Only specific email addresses can even request an OTP. An unknown email gets rejected before anything else happens.

I chose email OTP over a shared password for one reason — there's no password to steal or guess. The security relies on inbox access, which is already protected by the email provider's own auth (including 2FA if enabled). It's also stateless from my side: no credential database to maintain, no password rotation to think about.

After a successful OTP, Cloudflare issues a short-lived JWT cookie. The user doesn't have to re-authenticate every request — just once per session.

---

## Static IP — DHCP Reservation, Not Device-Side Config

The phone needs a consistent LAN IP so the tunnel config and any local references don't break when it reconnects to Wi-Fi.

I did this via DHCP reservation on the router, not by setting a static IP in Android's network settings. The router sees the phone's MAC address and always hands it the same IP. Android thinks it got a normal DHCP lease — no special config needed on the device side. This is cleaner because the router stays in control of the IP space and there's no risk of conflicts if something else grabs that address first.

---

## LAN vs. Internet Access

Two access paths exist:

| Path | Route | Auth |
|---|---|---|
| LAN | `192.168.x.x:4533` direct | Navidrome credentials only |
| Internet | `music.yourdomain.com` via tunnel | Cloudflare OTP → Navidrome credentials |

LAN access bypasses Cloudflare entirely — direct to the phone, lower latency, works even if home internet is down. The Cloudflare layer only applies to external traffic. This is an intentional trade-off: slightly different security posture on LAN vs. internet, but acceptable for a home network.

---

## The Signal 31 Problem and What It Taught Me About Android Networking

This is covered in detail in [ARCHITECTURE.md](./ARCHITECTURE.md#the-problems-i-actually-had-to-solve), but the short version: Navidrome binding to `0.0.0.0` on Android 16 triggered a syscall blocked by the tightened seccomp filter, crashing the process immediately.

The fix — binding to `127.0.0.1` instead — actually clarified something about how the whole stack fits together. `cloudflared` connects to `localhost:4533`. The tunnel doesn't care what interface Navidrome binds to, as long as it's reachable from localhost. Restricting the bind address is technically *more correct* for this use case anyway — Navidrome doesn't need to be reachable on every interface, only on loopback (for the tunnel) and the LAN interface (for local access).