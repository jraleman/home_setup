# 02 · LibreELEC on the Raspberry Pi 5

Goal: the Pi 5 boots straight into Kodi on the TV. It's controllable with the TV
remote and a phone, and set up for good picture and sound.

[LibreELEC](https://libreelec.tv) ("Just enough OS for Kodi") is a minimal,
appliance-style Linux. The root filesystem is read-only, and everything you
change lives under `/storage`.

## Step 1: Assemble the hardware

1. Fit the **Active Cooler** (or the case with the fan) before anything else.
2. (Optional) Fit the M.2 HAT and NVMe SSD if you're using one.
3. Connect **Ethernet**.
4. Connect the TV/AV receiver to **HDMI0**, the micro-HDMI port closest to the USB-C power port.
5. Don't plug in power yet.

## Step 2: Flash LibreELEC

1. Open **Raspberry Pi Imager**.
2. **Device:** Raspberry Pi 5.
3. **OS:** *Media player OS* → **LibreELEC** → the image for **RPi5**.
   (Alternatively, download the RPi5 image from
   [libreelec.tv/downloads](https://libreelec.tv/downloads) and use *Use custom*.)
4. **Storage:** the microSD card (or the NVMe drive in a USB enclosure).
5. Skip OS customisation. LibreELEC has its own first-run wizard.
6. Write, insert the card, connect power. The first boot resizes the storage
   partition and reboots on its own.

> **Booting from NVMe:** the Pi 5 bootloader can boot NVMe directly. If it
> doesn't, update the bootloader EEPROM (possible from Raspberry Pi OS or via the
> *Bootloader* option in Raspberry Pi Imager's *Misc utility images*) and set the
> boot order to try NVMe first.

## Step 3: First-run wizard

| Screen | Choose |
|---|---|
| Language / region | Your locale and keyboard layout |
| Hostname | `kodi` |
| Networking | Ethernet connects automatically. Skip Wi-Fi if wired. |
| Sharing & remote access | Enable **SSH** (set a password when asked) and **Samba** |

Then in your router, add a **DHCP reservation** for `kodi` (e.g. `192.168.1.3`)
and reboot the Pi 5. Because the router hands out the Pi-hole as DNS, the Pi 5
is already ad-blocked.

Check SSH access from your computer:

```bash
ssh root@192.168.1.3
```

> Set or change the root password under *Settings → LibreELEC → Services → SSH*
> if you didn't set one during the wizard. Never leave SSH on with a default
> password.

## Step 4: Switch Kodi settings to "Expert"

Many of the options below are hidden by default. In **Settings** (gear icon),
click the level toggle at the bottom-left of any settings page until it reads
**Expert**.

## Step 5: Picture settings

*Settings → System → Display*

- **Resolution:** your TV's native resolution (e.g. 3840×2160 or 1920×1080).
- **Refresh rate:** 60 Hz for the menus (50 Hz in PAL regions if you prefer).
- **Whitelist** (Expert): add the resolutions/refresh rates your TV supports,
  e.g. 23.976, 24, 25, 29.97, 50, 59.94, 60 Hz.

*Settings → Player → Videos*

- **Adjust display refresh rate:** *On start/stop*. Films then play at 24 Hz
  without judder.
- **Sync playback to display:** leave off unless you notice judder or audio drift.

HDR10/HLG passthrough works on the Pi 5 when the TV supports it. Dolby Vision
isn't supported.

## Step 6: Sound settings

*Settings → System → Audio*

- **Audio output device:** the HDMI output (it names your TV or receiver).
- **Number of channels:** 2.0 for TV speakers, or match your AV receiver (5.1/7.1).
- **Allow passthrough:** turn on if you have an AV receiver/soundbar that decodes
  Dolby/DTS, then enable the formats it supports (AC3, E-AC3, DTS, TrueHD, DTS-HD).

## Step 7: Remote controls

**TV remote via HDMI-CEC (no extra hardware):**

1. Turn on CEC in your TV's settings. Brands call it Anynet+ (Samsung),
   SimpLink (LG), Bravia Sync (Sony), and so on.
2. In Kodi: *Settings → System → Input → Peripherals → CEC Adapter*. Choose
   whether the TV powers on/off with Kodi and which input Kodi switches to.

**Phone as a remote:**

1. *Settings → Services → Control*: enable **Allow remote control via HTTP**,
   set a username/password, and enable **Allow remote control from applications
   on other systems**.
2. Install the official **Kore** app (Android) or a remote app such as Yatse/Official
   Kodi Remote. Point it at `192.168.1.3`, port `8080`.

## Step 8: Updates

*Settings → LibreELEC → Updates*

- **Automatic updates:** *Notify* is a good middle ground. You'll see a prompt and
  can choose when to update.
- Stay on the stable channel. Major versions (new Kodi releases) can break add-ons
  until those are updated. Read the release notes before jumping.
- The same page offers to update the Pi 5 **bootloader EEPROM** when a new one is
  available.

## Step 9: Quick sanity checks

- Play a test video. Press `O` on a keyboard (or open the info overlay) to see
  whether it's using hardware decode and how many frames are dropped.
- *Settings → System info → Hardware*: check that the CPU temperature stays
  well below 80 °C under load. Higher than that means the cooler isn't working.
- If a lightning-bolt icon or "undervoltage" warning appears, use the official
  27 W power supply.

---

← [01 · Pi-hole](01-pi-zero-pihole.md) · Next: [03 · Media sources →](03-media-sources.md)
