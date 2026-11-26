# 04 · RetroPlayer and controllers

Goal: retro games launched from Kodi's **Games** menu, using Kodi's built-in
**RetroPlayer** with libretro emulator cores. The four controllers are paired
and mapped.

> **Legal note:** only use ROMs dumped from games you own, and BIOS files from
> hardware you own. This guide doesn't link to ROM sites.

## What runs well on a Pi 5

| System | Kodi emulator add-on (libretro core) | Pi 5 performance |
|---|---|---|
| NES | `Nintendo - NES / Famicom (Nestopia UE)` or `FCEUmm` | Perfect |
| SNES | `Nintendo - SNES / SFC (Snes9x)` | Perfect |
| Game Boy / Color / Advance | `Gambatte`, `mGBA` | Perfect |
| Sega Genesis / Mega Drive / Master System | `Genesis Plus GX` | Perfect |
| PlayStation 1 | `PCSX ReARMed` | Very good (needs BIOS for best compatibility) |
| Nintendo 64 | `Mupen64Plus-Next` | Good for most games, some need tweaks |
| GameCube / PS2 | — | Not practical in Kodi on a Pi 5 |

Exact add-on names vary a little between Kodi versions. Search by system name.

## Step 1: Install emulator add-ons

1. *Settings → Add-ons → Install from repository*.
2. Choose **All repositories** (LibreELEC builds its ARM game add-ons into its own
   repository) → **Game add-ons** → **Emulators**.
3. Install the cores you want from the table above.

> Kodi can also install the right emulator automatically the first time you
> launch a ROM, but installing up front avoids surprises.

## Step 2: Add your ROM library

1. Copy ROMs to the Pi 5 or the NAS, one folder per system:

   ```
   Games/
     SNES/
     N64/
     NES/
     PSX/
   ```

   - To copy to the Pi 5 over the network, use the Samba share enabled in
     [02 · Step 3](02-pi5-libreelec.md#step-3-first-run-wizard):
     `\\kodi\Emulators` (Windows) or `smb://kodi/Emulators` (macOS/Linux).
   - Or put them on the NAS next to your media.

2. In Kodi: main menu → **Games** → *Add games…* → **Browse** → pick the folder
   (local path or the NAS network location, same as in
   [03 · A3](03-media-sources.md#a3-add-the-source)).
3. Launch any game. Kodi picks (or offers to install) a matching emulator.

### BIOS files (PS1 and some others)

Each core looks for BIOS files in its add-on's `system` folder. On LibreELEC
that is under:

```
/storage/.kodi/userdata/addon_data/game.libretro.<core>/system/
```

Copy them via SSH/SCP or the Samba share. BIOS file names must match exactly
what the core expects. Check the core's page on the
[libretro docs](https://docs.libretro.com/).

## Step 3: Connect the controllers

![Controller cheat sheet. 8BitDo SNES-style: USB-C in X-input mode, Bluetooth in D-input mode, or the USB receiver on 2.4G models; map it as Super Nintendo. Tribute64: 2.4 GHz receiver or USB-C, in PC mode; map it as Nintendo 64 with the C-buttons on the right stick. DualShock 4: Bluetooth (hold Share + PS) or a micro-USB data cable; map it as Default. GameCube-style pad for Switch: Bluetooth, USB, or an adapter in PC mode; map it as GameCube or Default. Map Kodi Default on every pad too, for the menus.](images/controllers.svg)

### General advice

- **Wired (or a 2.4 GHz USB dongle) gives the lowest latency.** Bluetooth is
  fine for most games.
- Pair Bluetooth controllers in **Settings → LibreELEC → Bluetooth**. Put the
  controller in pairing mode, wait for it to appear, select it and choose *Pair*.
- Using Ethernet on the Pi 5 helps Bluetooth stability, since Wi-Fi and
  Bluetooth share a radio.
- Update every controller's firmware **from a PC or phone first**. Old
  firmware is the #1 cause of pairing and mapping problems.

### 8BitDo SNES-style controller

8BitDo pads have several "modes" picked by holding a button combo while
powering on. The exact combo depends on the model (SN30 Pro, SN30 Pro+, SN30
2.4G, …), so check the manual or the 8BitDo support page for yours.

- **Wired:** plug in via USB-C. In **X-input** mode it appears as an Xbox
  controller and is the most plug-and-play.
- **Bluetooth:** power on in **D-input / Android** mode (often `Start + B`),
  then hold the **pair** button (often `Select` for 3 s) until the LED blinks
  rapidly. Pair from LibreELEC settings.
- **2.4 GHz models:** plug in the USB receiver. It pairs automatically.
- Firmware: **8BitDo Ultimate Software** / *Firmware Updater* on Windows or macOS.

### Retro Fighters Tribute64

- **Wireless:** plug the **2.4 GHz USB receiver** into the Pi 5 and switch the
  controller to its **PC / X-input** mode (see its manual for the mode switch or
  combo).
- **Wired:** USB-C cable in PC mode.
- It appears as a standard gamepad. The **C-buttons** usually map to the right
  analog stick, which is exactly what N64 cores expect. Map them as C-buttons
  in Step 4.

### Sony DualShock 4 (PS4)

- **Bluetooth:** hold **Share + PS** until the light bar **double-flashes**
  rapidly, then pair from LibreELEC settings. The Linux kernel supports it
  natively.
- **Wired:** micro-USB data cable (some cheap cables are charge-only).
- Tip: the touchpad can show up as a mouse and move a pointer around Kodi. If
  that bothers you, use the controller wired, or ignore the pointer. It
  disappears after a few seconds.

### Nintendo Switch GameCube-style controller

Which kind you have changes how it connects:

| Controller | How to connect |
|---|---|
| **Wireless third-party GameCube-style pad for Switch** (e.g. PowerA) | Usually acts as a *Switch Pro Controller* over Bluetooth. Hold the sync button and pair from LibreELEC settings. Supported by the kernel's `hid-nintendo` driver. |
| **Wired GameCube-style pad for Switch** (USB) | Plug in. Usually detected as a Pro Controller or generic HID gamepad. |
| **Original GameCube controller + USB adapter** (Mayflash / Wii U-style) | Set the adapter's switch to **PC** mode, then plug in. Each port shows as a separate gamepad. |
| **Nintendo's official GameCube controller for Switch 2** | Newer hardware. Linux support depends on the kernel in your LibreELEC version. Try USB first, then Bluetooth. If it doesn't show up under *Configure attached controllers*, it isn't supported yet on your build. |

GameCube games themselves aren't practical on the Pi 5, but this pad is
comfortable for SNES, N64 and Genesis games.

## Step 4: Map each controller in Kodi

Kodi stores a button map per physical controller, so do this once per pad:

1. *Settings → System → Input → **Configure attached controllers***.
2. Pick the **controller profile** on the left that matches how you'll use it:
   - **Kodi Default** (or "Default"): for navigating Kodi's menus. Map this for
     every pad.
   - **Super Nintendo** for the 8BitDo.
   - **Nintendo 64** for the Tribute64 (map C-buttons on the right stick).
   - **GameCube** / **Default** for the GameCube-style pad.
   - **Default** (it uses a PlayStation layout) for the DualShock 4.
3. Click **Get started** and press each button as Kodi highlights it. Press
   nothing for 5 s, or use *Skip*, for buttons the pad doesn't have.
4. Repeat for every attached controller.

Per-game options, including video filters, aspect ratio, save/load states and
the button layout, are in the **in-game menu**. Open it with the controller's
**Home/Guide** button, or `Esc` on a keyboard. **Settings → Games** has more
options, such as auto-save on exit.

## Step 5: Recommended game settings

- **Settings → Games → General → Enable auto-save:** on.
- In-game menu → **Video settings:** choose *Nearest neighbour* or a
  scanline/CRT shader for sharp 8/16-bit pixels. *Stretch mode: Original
  (aspect-correct)*.
- N64: if a game runs slowly, open its emulator add-on's settings and lower the
  internal resolution (`Mupen64Plus-Next` → *RDP plugin / resolution factor*).

---

← [03 · Media sources](03-media-sources.md) · Next: [05 · Docker on the Pi 5 →](05-docker-pi5.md)
