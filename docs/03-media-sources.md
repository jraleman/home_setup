# 03 · Media sources

Kodi on the Pi 5 pulls from three places:

| Source | Best for | Section |
|---|---|---|
| **NAS / network share** (SMB or NFS) | Your own library, watched mainly on this TV | [A](#a-nas--network-shares) |
| **Self-hosted media server** (Jellyfin) | One library shared to Kodi *and* phones, laptops, other TVs | [B](#b-self-hosted-media-server) |
| **Streaming add-ons** | Legitimate online services (YouTube, etc.) | [C](#c-streaming-add-ons) |

You can use all three at once. Pick one *primary* way to browse your own
library, either Kodi's library (A) or Jellyfin (B). Otherwise you'll have two
copies of the same movies in the menus.

![Three ways media reaches Kodi on the Pi 5. A: Kodi reads the NAS share over SMB or NFS into its own library. B: Jellyfin, running in Docker on the Pi 5, reads the same NAS files, and Kodi connects to it with Jellyfin for Kodi or JellyCon; Jellyfin also serves phones, laptops and other TVs. C: streaming add-ons from the official Kodi repository. Using A and B for the same files shows every movie twice.](images/media-sources.svg)

## Before you start: organise your files

Kodi's scrapers (and Jellyfin's) match files by name. A predictable layout saves
hours of fixing mismatches:

```
Movies/
  Blade Runner (1982)/
    Blade Runner (1982).mkv
TV Shows/
  The Expanse/
    Season 01/
      The Expanse S01E01.mkv
Music/
  Artist/
    Album (Year)/
      01 - Track.flac
```

Keep movies and TV shows in **separate top-level folders**. Each folder gets
its own content type.

---

## A. NAS / network shares

### A1. Prepare the share on the NAS

- Create a read-only user (e.g. `kodi`) on the NAS with access to the media shares.
- **NFS** is the most efficient for Kodi on Linux (no auth overhead). Allow the
  Pi 5's IP (`192.168.1.3`) read access to the export.
- **SMB** is easier to set up and works fine. Use SMB2 or SMB3. Disable SMB1 on
  the NAS.

### A2. Tell Kodi about SMB minimum version (SMB only)

*Settings → Services → SMB client*

- **Minimum protocol version:** `SMBv2`
- **Maximum protocol version:** `SMBv3`

### A3. Add the source

1. Main menu → **Movies** → *Enter files section* → **Add videos…**
2. **Browse** → *Add network location…*
   - Protocol: **Network File System (NFS)** or **Windows network (SMB)**
   - Server name: `192.168.1.4` (or `nas`)
   - Shared folder: e.g. `/volume1/media/Movies` (NFS) or `media/Movies` (SMB)
   - Username/password: the `kodi` user (SMB only)
3. Select the new location → **OK**. Name the source "Movies".
4. **Set content:** *This directory contains* → **Movies**.
   - Information provider: **The Movie Database Python** (default).
   - Enable *Movies are in separate folders that match the movie title* if you
     used the layout above.
5. Let it scan. Repeat for **TV Shows** (content: *TV shows*) and **Music**.

### A4. Keep the library fresh

*Settings → Media → Library*

- **Update library on startup:** on.
- Or install the **Library Auto Update** add-on (official Kodi repository →
  *Program add-ons*) to scan on a schedule.

---

## B. Self-hosted media server

This guide runs **Jellyfin** in Docker on the Pi 5 itself. Server setup is in
[05 · Docker](05-docker-pi5.md#step-4-run-jellyfin). Here we set up the
Kodi **client** side. If you already run Jellyfin, Emby or Plex on another
machine, these steps are the same. Point them at that machine instead.

### B1. Jellyfin for Kodi

There are two ways to use it:

| Add-on | What it does | Choose it if |
|---|---|---|
| **Jellyfin for Kodi** | Syncs the Jellyfin library into Kodi's own library and skins | You want Kodi's normal menus, with watched status synced to Jellyfin |
| **JellyCon** | Browses Jellyfin live from an add-on menu, no sync | You want lighter setup and several users/profiles |

Install (example: Jellyfin for Kodi):

1. *Settings → System → Add-ons* → enable **Unknown sources** (accept the warning).
   Only install from repositories you trust.
2. *Settings → File manager → Add source* → enter the URL
   `https://kodi.jellyfin.org` → name it `jellyfin`.
3. *Settings → Add-ons → Install from zip file* → `jellyfin` →
   `repository.jellyfin.kodi.zip`.
4. *Install from repository* → **Kodi Jellyfin Add-ons** → *Video add-ons* →
   **Jellyfin** (or **JellyCon**).
5. Launch it. It discovers the server on the LAN, or you can enter
   `http://192.168.1.3:8096`. Sign in, then choose which libraries to sync.

Check the current repository URL on the
[Jellyfin Kodi client docs](https://jellyfin.org/docs/general/clients/kodi/)
if the one above doesn't load.

### B2. If you use Plex or Emby instead

- **Emby:** the *Emby for Kodi* add-on from Emby's own repository.
- **Plex:** Plex discontinued its official Kodi add-on. The community
  **PM4K (Plex for Kodi)** add-on is the usual replacement.

---

## C. Streaming add-ons

Stick to the **official Kodi repository** and well-known legitimate add-ons.
"Builds" and add-ons that offer free premium content are usually piracy,
often carry malware, and break all the time.

### C1. Inputstream components (needed for most streaming)

*Settings → Add-ons → My add-ons → VideoPlayer InputStream*

- Enable **InputStream Adaptive**.
- Enable **InputStream Helper** (from *Install from repository → Kodi Add-on
  repository → Program add-ons*, if not already present). It downloads the
  Widevine DRM component on first use for services that need it.

### C2. Examples of legitimate add-ons

| Add-on | Where | Notes |
|---|---|---|
| **YouTube** | Official repo → Video add-ons | Needs your own free Google API keys. The add-on's settings link a how-to. |
| **Twitch**, **Vimeo**, **SoundCloud**, podcasts | Official repo | No DRM, work out of the box |
| Your country's public broadcaster(s) | Official repo / broadcaster's own repo | Many are free and legal |
| **Netflix**, **Amazon VOD** | Third-party repos (e.g. CastagnaIT for Netflix) | Unofficial but use your own paid account. DRM limits resolution on Linux. Expect occasional breakage. |

Install: *Settings → Add-ons → Install from repository → Kodi Add-on
repository → Video add-ons*.

### C3. Pi-hole interactions

If a streaming add-on fails with network errors, open the Pi-hole Query Log,
filter by `192.168.1.3`, and look for blocked domains at that moment. Allow the
ones the service needs. See [01 · Step 7](01-pi-zero-pihole.md#step-7-tune-it-for-a-media-household).

---

← [02 · LibreELEC](02-pi5-libreelec.md) · Next: [04 · RetroPlayer and controllers →](04-retroplay-controllers.md)
