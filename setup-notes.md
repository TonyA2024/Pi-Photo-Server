# Setup Notes

Command-by-command reference for how this server was built.

## 1. Storage: Partition, Format, and Mount the SSD

```bash
lsblk                                  # identify the drive (e.g. /dev/sda1)
sudo umount /dev/sda1                  # if already mounted
sudo mkfs.ext4 /dev/sda1               # format to ext4
sudo mkdir /mnt/photos
sudo mount /dev/sda1 /mnt/photos
```

Make the mount persistent across reboots:

```bash
sudo blkid /dev/sda1                   # copy the UUID
sudo nano /etc/fstab
```

Add to the bottom of `/etc/fstab`:

```
UUID=<drive-uuid> /mnt/photos ext4 defaults,nofail 0 2
```

`nofail` ensures the Pi still boots normally even if the drive isn't detected for some reason.

Verify without rebooting, then confirm with a real reboot:

```bash
sudo systemctl daemon-reload
sudo mount -a
sudo reboot
# after reboot:
lsblk   # confirm /mnt/photos is mounted
```

## 2. Docker + Immich

```bash
curl -sSL https://get.docker.com | sh
sudo usermod -aG docker $USER
# log out/in or reboot for group change to apply

mkdir ~/immich-app && cd ~/immich-app
wget -O docker-compose.yml https://github.com/immich-app/immich/releases/latest/download/docker-compose.yml
wget -O .env https://github.com/immich-app/immich/releases/latest/download/example.env
wget -O hwaccel.transcoding.yml https://github.com/immich-app/immich/releases/latest/download/hwaccel.transcoding.yml
```

Edit `.env`:
- `UPLOAD_LOCATION=/mnt/photos`
- `DB_PASSWORD=<strong-password>`

Launch:

```bash
docker compose up -d
docker compose ps        # confirm all containers are Up
```

Access at `http://<pi-ip>:2283` to create the admin account.

## 3. Photo Library Import

Existing ~80GB library (organized as `Year/Month/photo.jpg`) was imported via the Immich web client's upload feature. Immich indexes by EXIF capture date rather than source folder structure, so the nested Year/Month organization required no restructuring beforehand.

## 4. SSH Hardening (Key-Based Auth)

On each client device:

```bash
ssh-keygen -t ed25519
```

Copy the public key to the Pi (Linux/Mac):

```bash
ssh-copy-id <user>@<pi-ip>
```

Windows (no `ssh-copy-id` by default):

```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh <user>@<pi-ip> "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys"
```

After confirming key-based login works from a fresh terminal session:

```bash
sudo nano /etc/ssh/sshd_config
# set: PasswordAuthentication no
sudo systemctl restart ssh
```

## 5. Remote Access via Tailscale

```bash
curl -fsSL https://tailscale.com/install.sh | sh
sudo tailscale up
```

Authorize via the printed URL, then install Tailscale on all client devices (same account). Once connected, both SSH and the Immich web app (`http://<pi-tailscale-address>:2283`) are reachable from any network, with no router port forwarding required.

## 6. Safe Shutdown (for physically relocating the Pi)

```bash
sudo shutdown -h now
```

Wait for the activity LED to stop blinking before disconnecting power. On restart, the SSD auto-mounts (fstab), Docker containers auto-restart, and Tailscale reconnects — no manual steps required.
