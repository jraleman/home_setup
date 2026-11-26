# home_setup

Your guide to KODI and a proper home setup :)

A step-by-step guide to building a living-room entertainment system out of two
Raspberry Pis: a **Kodi media center** that also plays retro games, and a tiny
**network-wide ad blocker**.

## What you end up with

- **Kodi on the TV**, booting straight from a Raspberry Pi 5 running
  [LibreELEC](https://libreelec.tv), controlled with the TV remote (HDMI-CEC) or your phone.
- **One media library** from your NAS, a self-hosted [Jellyfin](https://jellyfin.org)
  server, and legitimate streaming add-ons.
- **Retro gaming** with Kodi's built-in RetroPlayer: NES, SNES, Genesis, Game Boy,
  PS1 and N64, using the controllers you already own.
- **No ads or trackers** on any device in the house, thanks to
  [Pi-hole](https://pi-hole.net) on a Raspberry Pi Zero.
- **Room to grow** with Docker containers on the Pi 5.

## The setup

| Device | Role | OS |
|---|---|---|
| Raspberry Pi 5 | Kodi, RetroPlayer, Docker host (Jellyfin) | LibreELEC |
| Raspberry Pi Zero (bare-bones, headless) | Pi-hole DNS ad blocking | Raspberry Pi OS Lite |
| NAS / file server | Stores movies, shows, music and games | Whatever it runs |

```
Internet ─ Router ─┬─ Pi Zero  (Pi-hole DNS for the whole network)
                   ├─ Pi 5     (Kodi + RetroPlayer + Docker) ─HDMI─► TV
                   └─ NAS      (SMB / NFS media shares)
```

**Controllers** (wired and wireless):

- 8BitDo SNES-style controller
- Retro Fighters Tribute64 controller
- Sony DualShock 4 (PS4) controller
- Nintendo Switch GameCube-style controller

## Step-by-step guide

Work through the parts in order. Each one assumes the previous one is done.

| # | Part | What you'll do |
|---|---|---|
| 00 | [Overview and shopping list](docs/00-overview.md) | Hardware to buy, network layout, IP address plan |
| 01 | [Pi-hole on the Pi Zero](docs/01-pi-zero-pihole.md) | Headless Raspberry Pi OS Lite, Pi-hole install, point the router at it |
| 02 | [LibreELEC on the Pi 5](docs/02-pi5-libreelec.md) | Install Kodi, picture and sound settings, TV and phone remotes |
| 03 | [Media sources](docs/03-media-sources.md) | NAS shares, Jellyfin client, streaming add-ons |
| 04 | [RetroPlayer and controllers](docs/04-retroplay-controllers.md) | Emulators, game library, pairing and mapping all four gamepads |
| 05 | [Docker on the Pi 5](docs/05-docker-pi5.md) | LibreELEC Docker add-on, NAS mounts, Jellyfin server |
| 06 | [Maintenance and troubleshooting](docs/06-maintenance-troubleshooting.md) | Backups, monthly routine, common fixes |

## Before you start

You'll need:

- A computer with an SD card reader and
  [Raspberry Pi Imager](https://www.raspberrypi.com/software/)
- Admin access to your router (to set DNS and reserve IP addresses)
- About an afternoon for parts 00 to 02, and another for 03 to 05

The guide uses these example addresses. Swap in your own, using the plan in
[part 00](docs/00-overview.md#plan-your-addresses).

| Device | Hostname | Example IP |
|---|---|---|
| Router | — | `192.168.1.1` |
| Pi Zero (Pi-hole) | `pihole` | `192.168.1.2` |
| Pi 5 (LibreELEC) | `kodi` | `192.168.1.3` |
| NAS | `nas` | `192.168.1.4` |

## Quick links once it's running

| Service | Address |
|---|---|
| Pi-hole dashboard | `http://192.168.1.2/admin` |
| Kodi web remote | `http://192.168.1.3:8080` |
| Jellyfin | `http://192.168.1.3:8096` |
| SSH into the Pi 5 | `ssh root@192.168.1.3` |
| SSH into the Pi Zero | `ssh <user>@192.168.1.2` |

## A note on content

This guide only covers legal sources: your own media, official or well-known
add-ons that use your own subscriptions, and games and BIOS files dumped from
hardware you own. It doesn't cover piracy "builds" or ROM download sites.
