---
layout: archive
title: "My personal server"
permalink: /server/
author_profile: true
---



I recently built my own low-power home server, which runs on [ArchLinux](www.archlinux.org) (my favourite distribution) and provides cloud storage and streaming services for my family.
The server houses a low power mini-ITX motherboard with a soldered Intel Apollo Lake J3455 4-core CPU.
It has room for 4 standard 3.5" hard drives; currently it runs two 3TB data disks and one 1TB drive for OS backups and caches.
The OS is hosted on a 500GB M.2 SSD.

Notable software:

- [Netdata](https://my-netdata.io/) for system monitoring;
- [QBittorrent](https://www.qbittorrent.org/) for torrenting with a remote interface;
- [NextCloud](https://nextcloud.com) for data sharing in the cloud;
- [Plex](https://www.plex.tv) for video and music streaming;
- [NFS](https://en.wikipedia.org/wiki/Network_File_System_(protocol)) and [SFTP](https://www.ssh.com/ssh/sftp/) for remote access to data;
- [BTRFS](https://en.wikipedia.org/wiki/Btrfs) for data mirroring and protection;
- A [Steam](https://store.steampowered.com) cache is currently my next upcoming improvement.

<img src="/img/server/serverino.JPG" style="width:50%;" title="My small home-built server">

The server sits in our bedroom, so the challenge was to make it almost completely silent: this was accomplished by building a small wooden, noise-absorbing case, around 22x22x25cm in size. This is a prototype which I plan to improve on in the next iteration.

One large 20cm Noctua fan takes care of the ventilation under high loads, drawing fresh air from filtered openings on the bottom and back. Most of the time the fan is off and the system only uses natural convection for cooling, thanks to the large slot opening in the top and free-flowing design.

Power usage is 12W at idle and around 50W at load.
