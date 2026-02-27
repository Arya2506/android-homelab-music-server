# Step 9 — Script to Start Server

This script starts both Navidrome and the Cloudflare tunnel with a single command.

In Termux, create the file:

```bash
nano start-services.sh
```

Paste the script and save — `Ctrl+O`, `Enter`, `Ctrl+X`.

```bash
#!/data/data/com.termux/files/usr/bin/bash

# Start Navidrome music server in background
navidrome --configfile ~/navidrome/navidrome.toml > /dev/null 2>&1 &

# Start Cloudflare tunnel in background
# Replace <your-token-here> with your tunnel token from Cloudflare dashboard
cloudflared tunnel run --token <your-token-here> > /dev/null 2>&1 &
```

Make it executable and run:

```bash
chmod +x start-services.sh
bash start-services.sh
```