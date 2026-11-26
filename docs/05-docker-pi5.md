# 05 · Docker on the Pi 5 (LibreELEC add-on)

Goal: run extra services in Docker containers on the Pi 5, next to Kodi. The
main example is **Jellyfin**, the self-hosted media server from
[03 · B](03-media-sources.md#b-self-hosted-media-server).

> **Pi-hole stays on the Pi Zero.** Don't also run it here, because two DNS
> servers fighting over port 53 on the network is a headache.

## Before you start: limits to know

- **Kodi comes first.** Containers share the CPU and RAM with playback and
  emulation. An 8 GB Pi 5 is recommended for this part.
- **No hardware transcoding for Jellyfin.** The Pi 5 has no hardware video
  encoder, so Jellyfin can't transcode efficiently. Make sure your client apps
  **direct play** (most modern phones/TVs handle H.264/HEVC natively). Turn off
  transcoding or cap it for remote streaming.
- **The root filesystem is read-only.** All container data must live under
  `/storage` (e.g. `/storage/docker/<app>`). Anything elsewhere is lost on reboot.

## Step 1: Install the Docker add-on

1. *Settings → Add-ons → Install from repository → **LibreELEC Add-ons** →
   **Services** → **Docker***.
2. Install and wait for the notification.
3. Check it from SSH:

   ```bash
   ssh root@192.168.1.3
   docker version
   docker run --rm hello-world
   ```

## Step 2: (Optional) Docker Compose

Compose makes containers repeatable with one YAML file per service. Check
whether your Docker add-on already includes it:

```bash
docker compose version
```

If not, install the standalone plugin into your home directory (`/storage` is
`$HOME` on LibreELEC):

```bash
mkdir -p ~/.docker/cli-plugins
curl -fsSL \
  https://github.com/docker/compose/releases/latest/download/docker-compose-linux-aarch64 \
  -o ~/.docker/cli-plugins/docker-compose
chmod +x ~/.docker/cli-plugins/docker-compose
docker compose version
```

The examples below show both `docker run` and Compose versions. Use whichever
you prefer.

## Step 3: Make the NAS media visible to containers

Containers can only see folders on the Pi 5's filesystem, so mount the NAS share
on the host first. LibreELEC uses **systemd mount units** in
`/storage/.config/system.d/`.

The **file name must match the mount path** (`/` becomes `-`). For a mount at
`/storage/nas/media`, the file is `storage-nas-media.mount`:

```bash
mkdir -p /storage/nas/media
nano /storage/.config/system.d/storage-nas-media.mount
```

**NFS version:**

```ini
[Unit]
Description=NAS media (NFS)
Requires=network-online.service
After=network-online.service
Before=kodi.service

[Mount]
What=192.168.1.4:/volume1/media
Where=/storage/nas/media
Type=nfs
Options=ro,nfsvers=4

[Install]
WantedBy=multi-user.target
```

**SMB version:** replace the `[Mount]` section with:

```ini
[Mount]
What=//192.168.1.4/media
Where=/storage/nas/media
Type=cifs
Options=ro,username=kodi,password=CHANGEME,vers=3.0,iocharset=utf8
```

Enable and check it:

```bash
systemctl daemon-reload
systemctl enable --now storage-nas-media.mount
ls /storage/nas/media
```

> USB drives plugged into the Pi 5 are mounted automatically under
> `/var/media/<drive-label>`. You can bind those into containers directly.

## Step 4: Run Jellyfin

Create folders for its data:

```bash
mkdir -p /storage/docker/jellyfin/{config,cache}
```

**Option A: `docker run`**

```bash
docker run -d \
  --name jellyfin \
  --restart unless-stopped \
  --network host \
  -v /storage/docker/jellyfin/config:/config \
  -v /storage/docker/jellyfin/cache:/cache \
  -v /storage/nas/media:/media:ro \
  jellyfin/jellyfin:latest
```

**Option B: Compose.** Save as `/storage/docker/jellyfin/compose.yaml`:

```yaml
services:
  jellyfin:
    image: jellyfin/jellyfin:latest
    container_name: jellyfin
    restart: unless-stopped
    network_mode: host      # simplest for LAN discovery (port 8096)
    volumes:
      - /storage/docker/jellyfin/config:/config
      - /storage/docker/jellyfin/cache:/cache
      - /storage/nas/media:/media:ro
```

```bash
cd /storage/docker/jellyfin
docker compose up -d
```

Then:

1. Browse to **http://192.168.1.3:8096** (or `http://kodi:8096` if you added the
   Pi-hole local DNS record) and finish the setup wizard.
2. Add libraries pointing at `/media/Movies`, `/media/TV Shows`, `/media/Music`.
3. *Dashboard → Playback → Transcoding:* leave hardware acceleration **off**
   (nothing to accelerate on the Pi 5). Lower the max streaming bitrate for
   clients outside the house so they don't force CPU transcodes.
4. Connect Kodi to it as described in [03 · B1](03-media-sources.md#b1-jellyfin-for-kodi).

## Step 5: Other containers worth considering

Keep the list short. Every container takes resources from Kodi.

| Container | Why | Image |
|---|---|---|
| **Portainer CE** | Web UI to manage containers | `portainer/portainer-ce` |
| **Watchtower** | Auto-updates container images on a schedule | `containrrr/watchtower` |
| **Syncthing** | Sync game saves / ROMs / photos with other devices | `syncthing/syncthing` |
| **Uptime Kuma** | Alerts if the Pi-hole or NAS goes down | `louislam/uptime-kuma` |

Use the same pattern each time: data under `/storage/docker/<name>`,
`--restart unless-stopped`, and a fixed port that doesn't clash with Kodi
(`8080`) or Jellyfin (`8096`).

## Step 6: Day-to-day commands

```bash
docker ps                          # what's running
docker logs -f jellyfin            # follow logs
docker restart jellyfin
docker stats                       # live CPU/RAM per container

# Update an image (docker run version)
docker pull jellyfin/jellyfin:latest
docker stop jellyfin && docker rm jellyfin
# …then run the same docker run command again (data is safe in /storage)

# Update an image (Compose version)
cd /storage/docker/jellyfin && docker compose pull && docker compose up -d

docker image prune -f              # free space from old images
```

---

← [04 · RetroPlayer](04-retroplay-controllers.md) · Next: [06 · Maintenance and troubleshooting →](06-maintenance-troubleshooting.md)
