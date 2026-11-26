# 06 · Maintenance and troubleshooting

## Backups

| What | How | When |
|---|---|---|
| **Pi-hole config** (lists, allow/deny, local DNS, settings) | Dashboard → *Settings → Teleporter → Export*. Restore with *Import* on a fresh install. | After any change |
| **LibreELEC + Kodi** (settings, add-ons, library DB, controller maps) | *Settings → LibreELEC → System → Backup → Create System and Kodi Backup*. Copy the `.tar` from `/storage/backup/` off the device (Samba share `Backup`). | Monthly, and before every major update |
| **Docker data** | Stop containers, then copy `/storage/docker/` off the device. (It's also inside the LibreELEC backup if that backup includes `/storage`.) | Monthly |
| **Game saves / save states** | Inside the LibreELEC backup. Or sync `/storage/.kodi/userdata/addon_data/game.libretro.*` with Syncthing. | — |
| **Whole SD cards** | Optional: image each card with `dd` or Raspberry Pi Imager's backup tools. | Before risky changes |

A good habit: keep at least one copy **off** both Pis (NAS or laptop).

## Monthly routine (10 minutes)

1. **Pi-hole:** `ssh` in → `sudo apt update && sudo apt full-upgrade -y` →
   `pihole -up` → glance at the dashboard for anything odd.
2. **LibreELEC:** check *Settings → LibreELEC → Updates*, read the release notes,
   and update if it's a point release.
3. **Kodi add-ons:** *Settings → Add-ons → My add-ons*. Updates normally install
   automatically. Check that Jellyfin/streaming add-ons still work.
4. **Docker:** pull new images and recreate containers (see
   [05 · Step 6](05-docker-pi5.md#step-6-day-to-day-commands)), then `docker image prune -f`.
5. **Controllers:** charge them. Check for firmware updates every few months.

## Troubleshooting

### Network / Pi-hole

| Symptom | Likely cause | Fix |
|---|---|---|
| Nothing on the network loads any website | Pi-hole is down | Power-cycle the Pi Zero. If it doesn't recover, set the router's DNS back to automatic (see [01 · Safety net](01-pi-zero-pihole.md#safety-net-if-the-pi-hole-dies)). |
| One app/site broken, everything else fine | A needed domain is blocked | Pi-hole *Query Log* → filter by that device → **Allow** the blocked domain. |
| Ads still showing on some devices | Device uses hard-coded DNS or DNS-over-HTTPS | Turn off "Secure DNS"/"Private DNS" in that browser or phone, or block outbound port 53 except from the Pi-hole on the router (if supported). |
| Dashboard shows queries only from the router's IP | Router forwards DNS for clients instead of handing out the Pi-hole's IP | Set the Pi-hole IP in the router's **DHCP** DNS setting (not its WAN DNS), or use Pi-hole's DHCP server. |

### Kodi / LibreELEC

| Symptom | Likely cause | Fix |
|---|---|---|
| No picture / "no signal" | Wrong HDMI port | Use **HDMI0** (next to USB-C power). Try another cable. |
| Lightning bolt / undervoltage warning | Weak PSU or cable | Official 27 W supply. Power hungry USB drives from a powered hub. |
| Stuttering on 4K files | H.264 4K (CPU-decoded) or Wi-Fi bandwidth | Use HEVC for 4K. Use Ethernet. |
| Film judder | Refresh rate not switching | *Settings → Player → Videos → Adjust display refresh rate: On start/stop* and set the whitelist ([02 · Step 5](02-pi5-libreelec.md#step-5-picture-settings)). |
| TV remote doesn't work | CEC off on TV | Enable CEC on the TV, and check *Peripherals → CEC Adapter* in Kodi. |
| NAS source "couldn't connect" | SMB version / credentials / NAS asleep | Set SMB min version to v2. Re-enter credentials. Check the NAS IP reservation. |
| Movies scraped as the wrong title | File/folder naming | Rename to `Title (Year)/Title (Year).ext`. Use *Refresh* on the item and search manually. |

### RetroPlayer / controllers

| Symptom | Likely cause | Fix |
|---|---|---|
| Controller pairs but buttons do nothing | Not mapped for that profile | *Settings → System → Input → Configure attached controllers*. Map **Kodi Default** and the system profile. |
| Controller won't pair | Wrong mode / old firmware | Put it in the right mode (D-input for 8BitDo Bluetooth). Update firmware from a PC. Remove the old pairing in LibreELEC Bluetooth settings and pair again. |
| Laggy input | Bluetooth interference | Use wired or the 2.4 GHz dongle. Use Ethernet so Wi-Fi isn't competing with Bluetooth. |
| PS1 games fail to start | Missing/misnamed BIOS | Put the correctly named BIOS in the core's `system` folder ([04 · BIOS](04-retroplay-controllers.md#bios-files-ps1-and-some-others)). |
| N64 game slow | Heavy title / high internal resolution | Lower the resolution in the Mupen64Plus-Next add-on settings. Check the Pi's temperature. |
| Can't exit a game | — | **Home/Guide** button or `Esc` opens the in-game menu → *Stop*. |

### Docker

| Symptom | Likely cause | Fix |
|---|---|---|
| Container data gone after reboot | Volume outside `/storage` | Move volumes under `/storage/docker/…`. |
| Jellyfin can't see media | NAS mount not up | `systemctl status storage-nas-media.mount`. Check `What=`/`Options=`. |
| Jellyfin playback buffers on phones | Transcoding on CPU | Enable direct play in the client. Raise its bitrate limit on the LAN. Use compatible formats. |
| Kodi stutters while containers run | Resource contention | `docker stats`. Stop or remove heavy containers, and schedule scans/updates at night. |

## Useful paths and addresses

| What | Where |
|---|---|
| Pi-hole dashboard | `http://192.168.1.2/admin` |
| Kodi web/remote | `http://192.168.1.3:8080` |
| Jellyfin | `http://192.168.1.3:8096` |
| Kodi userdata (LibreELEC) | `/storage/.kodi/userdata/` |
| LibreELEC backups | `/storage/backup/` |
| Systemd mount units | `/storage/.config/system.d/` |
| Docker app data | `/storage/docker/<app>/` |
| Pi-hole config (v6) | `/etc/pihole/pihole.toml` |

---

← [05 · Docker](05-docker-pi5.md) · Back to [00 · Overview](00-overview.md)
