# Security

## Threat Model

Before deciding what to build, I had to be honest about what the actual threats are. This is a personal music server, not a payment system. The realistic attack surface is:

- **Bots and scanners** — automated tools that sweep the internet looking for exposed services and known CVEs. This is the most common threat for anything home-hosted.
- **Credential stuffing** — automated login attempts using leaked username/password databases. Navidrome's login page is a target if it's directly reachable.
- **Someone who finds the domain** — unlikely for a personal server, but possible. Without a gate in front of the app, the login screen is the only thing between them and your data.

What I'm not defending against: physical device access (LineageOS encrypts storage by default), Cloudflare infrastructure compromise (accepted dependency), or a sophisticated targeted attacker. The controls I put in place are matched to the actual threat level — not over-engineered, not under-built.

---

## Why Layered Auth

A single auth control is a single point of failure. If Navidrome has a vulnerability in its login flow, and that's the only gate, you're exposed. Layers mean each control is independent — bypassing one doesn't get you anywhere without the next.

The four layers here, in order:

**No open ports.** The router has zero forwarding rules. The phone has no inbound ports open to the internet. A bot scanning my home IP finds nothing — there's literally no TCP handshake to initiate. This is the biggest single security win of the whole setup and it came for free with the Cloudflare Tunnel model.

**Cloudflare Zero Trust — email allowlist + OTP.** This runs at Cloudflare's edge, before the request touches the tunnel. An unknown email is rejected immediately. A known email gets an OTP sent to their inbox — no password to stuff or brute-force. Even if Navidrome had a zero-day in its login page, an unauthenticated attacker can never reach it through this setup.

**Navidrome credentials.** After passing Cloudflare, users still need a valid username and password. This covers the case where someone on the allowlist (say, a shared email) shouldn't have full library access.

**Registration disabled.** `EnableUserRegistration = false` in the config. Nobody can self-register. If someone somehow gets through the first three layers and reaches the Navidrome UI, they can't create their own account.

```
Internet
   │
   ├─ No open ports         ← bot finds nothing to connect to
   ├─ Zero Trust OTP        ← unknown users blocked at edge
   ├─ Navidrome login       ← credentials still required
   └─ Registration closed   ← no self-signup path
```

---

## The Cloudflare Dependency

Layers 1 and 2 rely on Cloudflare. That's worth being clear about — it's a conscious trade-off, not an oversight.

The alternative is running your own auth layer, your own ingress, managing TLS, keeping it updated. That complexity introduces more real-world risk than trusting Cloudflare's edge, which is the same Zero Trust product used by enterprises and is well-audited. If Cloudflare's free tier changes or they have an outage, Layer 3 (Navidrome auth) still holds — the server just becomes unreachable from the internet until the tunnel reconnects, which it does automatically.

---
