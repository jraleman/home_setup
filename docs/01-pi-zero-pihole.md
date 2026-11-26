# 01 · Pi-hole on the Pi Zero

Goal: a headless Pi Zero running [Pi-hole](https://pi-hole.net) as the DNS server
for the whole network. Every device, including the Kodi box, gets ads and trackers
blocked without installing anything on it.

## Step 1: Flash Raspberry Pi OS Lite

1. Open **Raspberry Pi Imager** on your computer.
2. **Device:** choose your Pi Zero model.
3. **OS:** *Raspberry Pi OS (other)* → **Raspberry Pi OS Lite**.
   - Zero 2 W: pick the **64-bit** Lite image.
   - Zero / Zero W: pick the **32-bit** Lite image (these boards can't run 64-bit).
4. **Storage:** select the microSD card.
5. When asked to apply **OS customisation**, fill in:
   - Hostname: `pihole`
   - Username and a strong password (avoid the old `pi` / `raspberry` default)
   - Wi-Fi SSID, password and **country** (skip if using an Ethernet adapter)
   - Locale / timezone
   - **Services → Enable SSH**. Prefer *public-key authentication* and paste
     your `~/.ssh/id_ed25519.pub`.
6. Write the card, put it in the Pi Zero and power it on. The first boot takes
   a few minutes on a Zero.

## Step 2: Reserve its IP address

1. In your router's admin page, find the device named `pihole` in the DHCP client list.
2. Create a **DHCP reservation** for it (e.g. `192.168.1.2`).
3. Reboot the Pi Zero so it picks up the reserved address:

   ```bash
   ssh <user>@pihole.local
   sudo reboot
   ```

If `pihole.local` doesn't resolve, use the IP from the router's client list.

## Step 3: Update the system

```bash
ssh <user>@192.168.1.2
sudo apt update && sudo apt full-upgrade -y
sudo reboot
```

Optional but recommended: install automatic security updates.

```bash
sudo apt install -y unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades   # answer "Yes"
```

## Step 4: Install Pi-hole

Pi-hole's official installer is a script piped to bash. Read it first if you
prefer:

```bash
curl -sSL https://install.pi-hole.net -o basic-install.sh
less basic-install.sh          # review it
sudo bash basic-install.sh
```

During the installer:

| Prompt | Choose |
|---|---|
| Static IP warning | Continue. You already reserved the IP on the router. |
| Upstream DNS provider | Any. Cloudflare or Quad9 are good defaults. This can be changed later. |
| Blocklists | Accept the default list (StevenBlack). |
| Admin web interface / web server | **Yes** |
| Query logging | **Yes** (turn it off later if you want more privacy or less SD wear) |
| Privacy mode | "Show everything" for now, which makes troubleshooting easier |

At the end it shows the admin URL and a generated password. Set your own
password:

```bash
sudo pihole setpassword
```

The dashboard is at **http://192.168.1.2/admin** (or `http://pi.hole/admin` once
DNS points at it).

## Step 5: Point the network at Pi-hole

In the router's **LAN / DHCP settings**:

1. Set **Primary DNS** to the Pi-hole's IP (`192.168.1.2`).
2. Leave **Secondary DNS empty**, or set it to the same IP.
   > Don't put `8.8.8.8` or similar as the secondary. Devices will use it
   > at random and bypass the blocking.
3. Save. Then reconnect devices (toggle Wi-Fi, or reboot) so they get the new
   DNS server from DHCP.

**If your router won't let you change the DNS server it hands out** (common on
ISP-supplied routers): disable DHCP on the router and enable Pi-hole's built-in
DHCP server instead (*Settings → DHCP* in the Pi-hole dashboard). Only do one
or the other. Two DHCP servers on one network cause chaos.

## Step 6: Verify it works

From any computer on the network:

```bash
# Should return 0.0.0.0 (blocked)
nslookup doubleclick.net 192.168.1.2

# Should return a real address (allowed)
nslookup raspberrypi.com 192.168.1.2
```

Then open the Pi-hole dashboard. The **Total queries** counter should climb as
devices browse.

## Step 7: Tune it for a media household

- **Streaming apps acting up?** Some services break when their telemetry
  domains are blocked. Open *Query Log*, filter by the TV/Kodi box's IP, find
  the blocked domain right when the problem happened, and click **Allow**.
- **Add local DNS names:** *Settings → Local DNS Records*, add `kodi → 192.168.1.3`
  and `nas → 192.168.1.4`, so you can type `http://kodi:8080` instead of IPs.
- **Reduce SD card wear (optional):** in *Settings → All settings*, lower how
  many days of queries are kept in the long-term database
  (`database.maxDBdays`), or turn query logging off.
- **Keep it updated:**

  ```bash
  pihole -up          # update Pi-hole
  pihole -g           # refresh blocklists
  ```

## Safety net: if the Pi-hole dies

All DNS on the network goes through this one tiny board. If it's unplugged or
the SD card fails, **every device loses internet name resolution**. Keep a note
(e.g. taped inside the router cupboard) with the quick fix:

> Router admin → LAN/DHCP → set DNS back to "automatic" / the router's own IP.

Back up the configuration with **Settings → Teleporter → Export** after any
significant change. See [06 · Maintenance](06-maintenance-troubleshooting.md).

---

← [00 · Overview](00-overview.md) · Next: [02 · LibreELEC on the Pi 5 →](02-pi5-libreelec.md)
