# Step 8 — Cloudflare Tunnel & Zero Trust

## Prerequisites

- Domain with DNS managed by Cloudflare (free)
- Cloudflare account (free)

## The Thinking Behind the Access Policy

Before the technical setup — the policy design took more thought than the rest of this step combined.

My users are friends. They're not developers. The requirements were:

- Give them a clean URL, nothing to install
- No shared password I have to manage or rotate
- Secure enough that a stranger who finds the URL can't get in
- Still require Navidrome credentials so access is per-person, not all-or-nothing

Email OTP hits all of these. They visit the URL, enter their email, get a code, done — session lasts until the JWT expires. No app, no VPN client, no password manager entry. On my side I control the allowlist — adding or revoking someone is one change in the dashboard.

The two-wall approach (Cloudflare OTP → Navidrome login) means even if someone's email gets compromised, they still can't access the library without the Navidrome password. Each friend has their own Navidrome account, so I can revoke individually without affecting anyone else.

---

## Dashboard Setup

Everything except the final Termux command is done in the Cloudflare dashboard — no config files, no CLI tunnel creation.

**Create the tunnel**

[one.dash.cloudflare.com](https://one.dash.cloudflare.com) → Networks → Tunnels → Create a tunnel → Cloudflared → name it → save.

Cloudflare generates a token for the tunnel. Copy it — you'll need it in Termux.

**Configure the public hostname**

Inside the tunnel settings → Public Hostnames → Add:

- Subdomain: `music`
- Domain: `yourdomain.com`
- Service: `http://127.0.0.1:4533`

Cloudflare handles the CNAME record automatically.

**Set up Zero Trust Access Policy**

Access → Applications → Add Application → Self-hosted.

- Application domain: `music.yourdomain.com`
- Policy action: **Allow**
- Identity provider: **One-time PIN**
- Include → Emails → add each person's email individually

Anyone not on the list gets rejected at Cloudflare's edge before the request touches the tunnel.

---

## Termux Side

Install cloudflared:

```bash
pkg install cloudflared -y
```

Run the tunnel using the token from the dashboard:

```bash
cloudflared tunnel run --token <your-token-here>
```

That's it. No config yml, no `tunnel login`, no manual DNS commands — the dashboard handled all of it. The token connects this Termux instance to the tunnel you already configured.

---

## Verify End-to-End

Test from mobile data — not home Wi-Fi — so the request actually goes through Cloudflare:

1. Visit `https://music.yourdomain.com`
2. Cloudflare intercepts → enter an allowlisted email → OTP arrives → enter it
3. Navidrome login screen appears → enter credentials
4. Music library loads

If you get through both walls on mobile data, the whole stack is working.