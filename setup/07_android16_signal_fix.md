# Step 7 — Android 16 Signal 31 Fix

## What Happened

Even after switching to the `pkg` install, Navidrome was crashing on startup with no useful output. Just killed. Running it with verbose logging showed `signal 31` — which isn't a Navidrome error, it's the kernel terminating the process.

Signal 31 is `SIGSYS` — sent by the kernel when a process makes a syscall that's blocked by a `seccomp` filter. Android uses seccomp to sandbox apps. Android 16 tightened the filter compared to earlier versions, and Navidrome's default network binding behavior triggered a syscall path that hit that new restriction.

The specific trigger: Navidrome binding to `0.0.0.0` (all interfaces) on startup. That particular bind path makes a syscall that Android 16's seccomp filter now blocks. The kernel sends SIGSYS, Navidrome dies.

## The Fix

Change the bind address in `navidrome.toml` from `0.0.0.0` to `127.0.0.1`:

```toml
# Crashes on Android 16 — seccomp blocks the bind syscall path
# Address = "0.0.0.0"

# Fix
Address = "127.0.0.1"
```

This works cleanly for this setup because `cloudflared` connects to `localhost:4533` anyway — it doesn't matter which interface Navidrome binds to from the tunnel's perspective. Restricting to loopback is actually more correct here than binding to all interfaces.

## LAN Access After the Fix

Binding to `127.0.0.1` means direct LAN access (`192.168.x.x:4533`) no longer works out of the box. Options:

- **Reverse proxy** — run nginx or caddy in Termux, bind it to the LAN interface, forward to Navidrome on loopback. Most flexible but adds a moving part.
- **Specific interface bind** — some builds allow binding to the LAN IP directly instead of loopback. Test if `Address = "192.168.x.x"` works on your setup without triggering SIGSYS.
- **Tunnel for everything** — skip LAN direct access entirely, always go through the Cloudflare tunnel even on home Wi-Fi. Slightly higher latency but simpler.

I went with the reverse proxy approach for LAN access. For most use cases, tunnel-for-everything is the simplest solution.

## Verify

```bash
navidrome --configfile ~/.config/navidrome/navidrome.toml
```

Should start and stay running. Confirm with:

```bash
curl http://127.0.0.1:4533/ping
```

A response means it's up. No more signal 31.

