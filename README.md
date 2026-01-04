# System Setup

## Install Debian and XFCE4
```shell
sudo apt install xfce4 xfce4-goodies
```

Add user to sudoers
```shell
su root
```

Modify `/etc/sudoers`
```shell
# Append
ron ALL=(ALL) ALL
```


## Prepare the external disk
```shell
# List available disks using:
lsblk

# Suppose your storage drive is /dev/sda, you need to partition and format it:
sudo parted /dev/sda -- mklabel gpt
sudo parted /dev/sda -- mkpart primary ext4 1MiB 100%

# Format the partition:
sudo mkfs.ext4 /dev/sda1

# Mount the partition:
sudo mkdir -p /mnt/nas
sudo mount /dev/sda1 /mnt/nas

# To make this mount permanent, edit /etc/fstab:
echo '/dev/sda1 /mnt/nas ext4 defaults 0 2' | sudo tee -a /etc/fstab

# Edit permissions
sudo chown -R $USER:$USER /mnt/nas
```

[Source](https://www.siberoloji.com/how-to-set-up-network-attached-storage-nas-in-debian-12-bookworm-system/)

### Set the spindowntime for the disk
```shell
sudo hdparm -S 60 /dev/sda1
```

Make it permanent modifying `/etc/hdparm.conf`
```shell
# Append
/dev/sda1 {
    spindown_time = 60
}
```
[Source](https://guide.debianizzati.org/index.php/Hdparm)

## Install XRDP
```shell
sudo apt install xrdp
sudo systemctl enable xrdp
```

Modify `/etc/xrdp/startwm.sh`
```shell
# Comment
test -x /etc/X11/Xsession && exec /etc/X11/Xsession
exec /bin/sh /etc/X11/Xsession

# Append
startxfce4
```

Restart the xrdp service:
```shell
sudo systemctl restart xrdp
```

Open firewall ports 3389/tcp


[Source](https://phoenixnap.com/kb/debian-remote-desktop)

# Firewall
```shell
sudo apt install firewalld
sudo systemctl enable firewalld
```

Commands
```shell
# Open a port
sudo firewall-cmd --permanent --add-port=7878/tcp

# Reload the firewall
sudo firewall-cmd --reload

# Check if ports are open
sudo firewall-cmd --list-ports
```

[Source](https://phoenixnap.com/kb/debian-remote-desktop)

## Install Samba
```shell
sudo apt install samba
```

Modify `/etc/samba/smb.conf`
```shell
# Append

[Shared]
   path = /mnt/nas
   browseable = yes
   writable = yes
   guest ok = no
   valid users = @smbusers
```

Add the user to @smbusers
```shell
sudo groupadd smbusers
sudo usermod -aG smbusers $USER
sudo smbpasswd -a $USER
```

Restart samba
```shell
sudo systemctl restart smbd
```

Open firewall ports 445/tcp and 139/tcp

[Source](https://www.siberoloji.com/how-to-set-up-network-attached-storage-nas-in-debian-12-bookworm-system/)


## Docker Compose

### Installation
Set up Docker's apt repository
```shell
# Add Docker's official GPG key:
sudo apt install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
sudo tee /etc/apt/sources.list.d/docker.sources <<EOF
Types: deb
URIs: https://download.docker.com/linux/debian
Suites: $(. /etc/os-release && echo "$VERSION_CODENAME")
Components: stable
Signed-By: /etc/apt/keyrings/docker.asc
EOF

sudo apt update
```

Install the latest version
```shell
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

[Source](https://docs.docker.com/engine/install/debian/)

### Post installation
Create the docker group
```shell
sudo groupadd docker
```
Add your user to the docker group
```shell
sudo usermod -aG docker $USER
```
Activate the changes to groups
```shell
newgrp docker
```

[Source](https://docs.docker.com/engine/install/linux-postinstall)

---

# Applications
[General Source](https://trash-guide.info/)

### Pi-Hole
Setup local DNS

#### Router changes
- Internet DNS
- Local Network DNS

### Proxy


### Prowlarr
- Set login information
- Add Indexer
- Add Sonarr and Radarr (Settings → Apps → Applications)

### Sonarr
Configuration:
- Set login information
- Set language to ITA
- Add Download Clients to Tranmission (delete category)
- Set Sonar API Key (Settings → General → Security → API Key) in Prowlarr
- Set Settings → Media Management → Episode Naming → Rename Episodes to true
- Add new Settings → Custom Format with Conditions Language Italian

Naming convention
- Series Folder Format: `{Series TitleYear}`
- Season Folder Format: `Season {season:00}`
- Episode Format
    - Standard: `{Series TitleYear} - S{season:00}E{episode:00} - {Episode CleanTitle:90} {[Custom Formats]}{[Quality Full]}{[Mediainfo AudioCodec}{ Mediainfo AudioChannels]}{[MediaInfo VideoDynamicRangeType]}{[Mediainfo VideoCodec]}{-Release Group}`
    - Daily: `{Series TitleYear} - {Air-Date} - {Episode CleanTitle:90} {[Custom Formats]}{[Quality Full]}{[Mediainfo AudioCodec}{ Mediainfo AudioChannels]}{[MediaInfo VideoDynamicRangeType]}{[Mediainfo VideoCodec]}{-Release Group}`
    - Anime: `{Series TitleYear} - S{season:00}E{episode:00} - {absolute:000} - {Episode CleanTitle:90} {[Custom Formats]}{[Quality Full]}{[Mediainfo AudioCodec}{ Mediainfo AudioChannels]}{MediaInfo AudioLanguages}{[MediaInfo VideoDynamicRangeType]}[{Mediainfo VideoCodec }{MediaInfo VideoBitDepth}bit]{-Release Group}`

### Sonarr Anime Plugin
- Set Sonarr API Key in the .env file

### Radarr
Configuration:
- Set login information
- Add qBittorrent as Download Client (delete category)
- Set Radarr API Key (Settings → General → Security → API Key) in Prowlarr
- Set Settings → UI → Language → Movie Info Language to Italian
- Set Settings → Metadata → Certification Country to Italy
- Set Settings → Media Management → Movie Naming → Rename Movies to true

Naming convention
- Standard Movie Format: `{Movie CleanTitle} {(Release Year)} - {{Edition Tags}} {[MediaInfo 3D]}{[Custom Formats]}{[Quality Full]}{[Mediainfo AudioCodec}{ Mediainfo AudioChannels]}{[MediaInfo VideoDynamicRangeType]}{[Mediainfo VideoCodec]}{-Release Group}`

### qBittorrent
Follow settings in: 