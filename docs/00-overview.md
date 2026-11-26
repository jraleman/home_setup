# 00 · Overview and shopping list

This guide builds a living-room entertainment system out of two Raspberry Pis:

| Device | Role | OS |
|---|---|---|
| **Raspberry Pi 5** | Kodi media center, RetroPlayer games, Docker host | [LibreELEC](https://libreelec.tv) |
| **Raspberry Pi Zero** | Network-wide ad blocking with Pi-hole | Raspberry Pi OS Lite, headless ("bare-bones") |

## How it fits together

```
                    Internet
                       │
                 ┌─────┴─────┐
                 │  Router   │  DHCP hands out Pi-hole as the DNS server
                 └─────┬─────┘
        ┌──────────────┼──────────────────────┐
        │              │                      │
 ┌──────┴──────┐ ┌─────┴───────────────┐ ┌────┴──────────┐
 │ Pi Zero     │ │ Pi 5 (LibreELEC)    │ │ NAS / PC      │
 │ Pi-hole     │ │  Kodi + RetroPlayer │ │ SMB/NFS shares│
 │ DNS :53     │ │  Docker: Jellyfin   │ └───────────────┘
 │ Web /admin  │ │  HDMI ─► TV         │
 └─────────────┘ │  BT/USB ◄─ gamepads │
                 └─────────────────────┘
```

Every device on the network asks the Pi-hole for DNS, so ads and trackers are
blocked everywhere, including inside Kodi. The Pi 5 plays media from a NAS, from
a self-hosted media server (Jellyfin in Docker), and from official streaming add-ons.

## Order of work

Follow the parts in this order. Each one assumes the previous one is done.

1. [Pi-hole on the Pi Zero](01-pi-zero-pihole.md): get the network foundation in place first.
2. [LibreELEC on the Pi 5](02-pi5-libreelec.md): install the OS, set up the TV, remote and network.
3. [Media sources](03-media-sources.md): NAS shares, the media server client, streaming add-ons.
4. [RetroPlayer and controllers](04-retroplay-controllers.md): emulators, ROM library, pairing the four gamepads.
5. [Docker on the Pi 5](05-docker-pi5.md): run Jellyfin (and other services) in containers.
6. [Maintenance and troubleshooting](06-maintenance-troubleshooting.md): backups, updates, fixes.

## Shopping list

### Raspberry Pi 5 (media center)

| Item | Notes |
|---|---|
| Raspberry Pi 5, **4 GB or 8 GB** | 4 GB is plenty for Kodi alone. Get 8 GB if you'll also run Docker containers. |
| Official 27 W USB-C power supply | Weaker supplies cause undervoltage warnings and limit USB power for controllers and drives. |
| Active Cooler or a case with a fan | The Pi 5 throttles under load without one, especially during emulation. |
| microSD card, 32 GB+, A2 rated | Or an M.2 HAT + NVMe SSD for faster boot and more durable storage (optional). |
| **Micro-HDMI → HDMI** cable | Use the HDMI0 port, the one closest to the USB-C power port. |
| Ethernet cable | Strongly recommended over Wi-Fi for streaming, and it leaves the Bluetooth radio free for controllers. |
| (Optional) USB SSD/HDD | Only if you want local storage on the Pi 5 itself. |

> The Pi 5 has **no 3.5 mm audio jack** and **no hardware H.264 decoder**. H.264
> is decoded on the CPU, which handles 1080p fine. HEVC/H.265 is hardware-decoded
> up to 4K60. Prefer HEVC files for 4K content.

### Raspberry Pi Zero (Pi-hole)

| Item | Notes |
|---|---|
| Raspberry Pi Zero **W**, or better, **Zero 2 W** | Pi-hole is light, so any Zero works, but a Zero 2 W is noticeably snappier. |
| 5 V / 2.5 A micro-USB power supply | The official Pi Zero supply is ideal. |
| microSD card, 16 GB+ | A quality brand. Pi-hole writes logs constantly. |
| (Optional) micro-USB OTG → Ethernet adapter | More reliable than Wi-Fi for a DNS server. **Required** if you have the original Pi Zero (no Wi-Fi). |
| (Optional) small case | |

### Controllers (already owned)

- 8BitDo SNES-style controller
- Retro Fighters Tribute64 controller
- Sony DualShock 4 (PS4) controller
- Nintendo Switch GameCube-style controller
- USB cables for each (useful for charging, firmware updates and wired play)

### Also needed

- A computer with an SD card reader for flashing images
- [Raspberry Pi Imager](https://www.raspberrypi.com/software/) installed on that computer
- Admin access to your router (to set DNS and reserve IP addresses)

## Plan your addresses

Write these down before you start. You'll need them throughout the guide.
Replace the example values with addresses that fit your router's network.

| Device | Hostname | Example IP |
|---|---|---|
| Router | — | `192.168.1.1` |
| Pi Zero (Pi-hole) | `pihole` | `192.168.1.2` |
| Pi 5 (LibreELEC) | `kodi` | `192.168.1.3` |
| NAS / file server | `nas` | `192.168.1.4` |

Reserve these in the router's **DHCP reservation** (sometimes called "static
lease" or "address reservation") page, keyed to each device's MAC address.
Reserving on the router is simpler and safer than setting static IPs on each
device.

---

Next: [01 · Pi-hole on the Pi Zero →](01-pi-zero-pihole.md)
