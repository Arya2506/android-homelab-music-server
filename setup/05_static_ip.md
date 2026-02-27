# Step 5 — Static IP

## Why It Matters

The phone needs a consistent LAN IP. If DHCP assigns a new one after reconnect, the Cloudflare tunnel config breaks and any saved LAN shortcuts stop working. The fix is a DHCP reservation on the router — the router always hands the same IP to the phone's MAC address.

## The Practical Problem

To make the reservation, you need the phone's current IP — visible in Settings → About Phone → Wi-Fi, or in Termux:

```bash
ip addr show wlan0
```

The catch: the moment you leave the Settings screen to open the router admin panel in a browser, Android can reassign the IP. You need both visible at the same time.

**How I solved it: scrcpy + split screen.**

Trying to do this on the phone alone was painful — small screen, swapping windows loses the IP, typing the router admin URL while memorising a number. Instead I connected the phone to my PC via USB and used [scrcpy](https://github.com/Genymobile/scrcpy) to mirror and control the phone from the PC screen. Router admin panel open in a browser on one side, phone screen on the other. No swapping, no losing the IP mid-task.

```bash
# On PC (scrcpy installed)
scrcpy
```

That's it — phone screen appears on PC, fully controllable with mouse and keyboard.

## The Reservation

Once you can see both simultaneously:

1. Open your router admin panel (`192.168.1.1` or `192.168.0.1` — check your router)
2. Find DHCP settings — labelled differently per router: "DHCP Reservation", "Static Lease", "Address Reservation"
3. Find the phone in the connected devices list, reserve its current IP against its MAC address
4. Save

From this point the phone always gets the same IP. Verify by reconnecting Wi-Fi and checking the IP hasn't changed.

