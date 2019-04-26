---
layout: archive
title: "My personal server"
permalink: /server/
author_profile: true
---

Ho recentemente assemblato un mio piccolo server personale, bassato su [ArchLinux](www.archlinux.org), la mia distribuzione Linux preferita. Il serverino fornisce servizi di cloud storage e streaming per tutta la mia famiglia. Utilizza una scheda madre mini-ITX con una CPU a basso consumo Intel J3455 della famiglia Apollo Lake. Può ospitare 4 hard disk da 3.5": al momento utilizzo due dischi da 3TB per i dati e uno da 1TB per backup del sistema operativo e come cache di sistema. L'OS è installato su un SSD da 500GB, che funziona anche come cache dati di primo livello.

Software di inteersse:

- [Netdata](https://my-netdata.io/) per il monitoraggio del sistema;
- [QBittorrent](https://www.qbittorrent.org/) per torrenting via interfaccia remota;
- [NextCloud](https://nextcloud.com) per condivisione dati nel cloud;
- [Plex](https://www.plex.tv) per streaming video e musica;
- [NFS](https://en.wikipedia.org/wiki/Network_File_System_(protocol)) e [SFTP](https://www.ssh.com/ssh/sftp/) per accesso remoto ai dati;
- [BTRFS](https://en.wikipedia.org/wiki/Btrfs) per il mirroring  dei dati e protezione dal [bitrot](https://en.wikipedia.org/wiki/Data_degradation);
- [rsync](https://en.wikipedia.org/wiki/Rsync) per backup periodici off-site;
- [systemd](https://en.wikipedia.org/wiki/Systemd) per calendarizzare servizi di manutenzione e script di pulizia;
- A breve implementerò anche una cache [Steam](https://store.steampowered.com).

<img src="/img/server/serverino.JPG" style="width:50%;" title="Il mio piccolo server autocostruito">

Il serverino risiede nella nostra camera da letto, quindi la sfida stava nel renderlo più silenzioso possibile. Ciò è stato ottenuto costruendo un piccolo case in legno, perfetto per assorbire i rumori elettronici, di circa 22x22x25cm. Questo primo prototipo è risultato perfetto per l'uso che ne facciamo, ma sto già progettando una seconda versione.

Una singola ventola Noctua da 20cm si occupa del raffreddamento in caso di carichi di lavoro elevati, estraendo aria dalla base e dal retro, dove sono presenti delle prese d'aria filtrate. La ventola è spenta e il sistema completamente silenzioso per la maggior parte del tempo, sfruttando per il raffreddamento il piccolo flusso d'aria generato dalla convezione naturale, grazie all'ampia apertura nella parte alta del case.

Il server utilizza circa 12W in idle e 50W sotto stress.
