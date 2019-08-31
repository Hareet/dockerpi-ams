# Docker Pi - AMS
Automated Media Server via Docker on Raspberry Pi

### Scenario

With Raspberry Pi releasing a version of their hardware with 4GB of RAM, it is possible to setup a lightweight fully automated media server running a set of docker containers. 

You're in a conversation with a friend, and he mentions something new to watch. You can pull out your cellphone, browse to a personal website, search the name and queue it to download on your dockerpi-ams. Once the download is finished, you will be able to playback most formats to 1 client, and potentially transcode the format for 1 stream.

### Hardware

- [RaspberryPi 4 Model B 4GB](https://chicagodist.com/products/raspberry-pi-4-model-b-4gb?src=raspberrypi)

- [Case with Fan + Heatsinks](https://www.amazon.com/Miuzei-Raspberry-Cooling-Heat-Sinks-Included/dp/B07TTN1M7G/ref=sr_1_4?keywords=raspberry+pi+4+case&qid=1567271850&s=gateway&sr=8-4)

- microSD card 

- [External TB Storage](https://www.amazon.com/LaCie-Professional-USB-C-External-STHA4000800/dp/B07G8JT7XN/ref=sr_1_3?keywords=lacie+d2&qid=1567272355&s=gateway&sr=8-3)

### Tools

- [HypriotOS](https://blog.hypriot.com/)
    - Fastest way to get Docker up and running on any Raspberry Pi.
- Private Torrent Tracker (PTP, etc)
- [Radarr](https://github.com/Radarr/Radarr)
- [Jackett](https://github.com/Jackett/Jackett)
- [Transmission](https://github.com/linuxserver/docker-transmission)
- [Custom Filebot for RPI](Dockerfile link)
- [Plex](https://github.com/plexinc/pms-docker)
- [LetsEncrypt](https://github.com/linuxserver/docker-letsencrypt)

## Method

### Setup MicroSD and External Storage

- Install Hypriot Flash tool
    - [flash](https://github.com/hypriot/flash)
- Change the contents of the cloud_config.yaml found in this directory with the following attributes:
    - users.name, users.plain_text_password
    - write_files.content.ssid, write_files.content.psk (password)
    - *Note* This will enable your RPI to automatically connect to your WiFi upon boot. You will be able to ssh from there.
- Open a terminal
    - Discover the mount point of your SD card. In OSX, run: `diskutil -l`
    - *Note* Replace the value after the -d parameter below with your own. There is a potential to flash one of your hard disks if you are not correct.
    - `flash -d /dev/disk3 --userdata cloud_config.yaml https://github.com/hypriot/image-builder-rpi/releases/download/v1.11.1/hypriotos-rpi-v1.11.1.img.zip`
- Insert SD card into Pi and boot up.
- Check ssh:
    `ssh ams@dockerpi.local`

- Mount External Storage:
    - `lsblk` to list where all devices are
    - `sudo mount -t hfsplus -o force,rw /dev/sda3 /srv` 
- *Optional* : Setup External Storage to AutoMount on boot
    - Once you have SSH into your dockerpi box, run: `blkid`
    - *Note* Copy the UUID of your External Storage
    - `sudo vi /etc/fstab`
    - Inside the editor, add a new line entry:
    `UUID=<your_value> /srv hfsplus force,rw,nofail,x-systemd.device-timeout=1,noatime 0 0`

- All setup of containers (below) happens from SSH to your dockerpi-ams.

### Setup Containers

#### Transmission Setup
- Create directory structure and change to it: 
    -`/srv/plexMediaServer/config/combustion`
- Download Combustion: 
    - `rm -f release.zip && wget https://github.com/Secretmapper/combustion/archive/release.zip && unzip release.zip;`
- *Note* Export the above directory where combustion was installed as a variable to the transmission container:

```
$ docker create \
    --name=transmission \
    --network=rpi \
    -e PUID=1000 \
    -e PGID=1000 \
    -e TZ=Denver/America \
    -e TRANSMISSION_WEB_HOME=/config/combustion/combustion-release \
    -p 9091:9091 \
    -p 51413:51413 \
    -p 51413:51413/udp
    -v /srv/plexMediaServer/config:/config \
    -v /srv/plexMediaServer/Downloads:/downloads \
    -v /srv/plexMediaServer/config/torrents:/watch \
    --restart unless-stopped \
    linuxserver/transmission
```

- Start the container:
    - `docker start transmission`
- Edit transmission settings to add login and filebot ping script
    - `sudo vi /srv/plexMediaServer/config/settings.json`
    - Change the following values and uncomment:
        - rpc-username: <username>
        - rpc-password: <plaintext_entry>
        - script-call-after: "/config/transmission/transmission-postprocess.sh"
        - rpc-host-whitelist:"t.dnsname.com"

### Jackett Setup
```
$ docker create \
    --name=jackett \
    --network=rpi \
    -e PUID=1000 \
    -e PGID=1000 \
    -e TZ=Denver/America \
    -p 9117:9117 \
    -v /srv/plexMediaServer/config:/config \
    -v /srv/plexMediaServer/Downloads:/downloads \
    -- restart unless-stopped \
    linuxserver/jackett
```

### Radarr Setup
```
$ docker create \
    --name=radarr \
```







