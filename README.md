# automation
Home theater automation scripts for downloading and processing.

# Introduction
This repository assumes you have a baseline understanding of how Docker works, what a docker-compose file is, and how to configure one. 

Most of the services/containers included in this setup operate independently and thus can be added or removed as you feel applicable to your own personal setup. Any services that are inherently bound to another container (i.e Transmission and WireGuard or Plex and rclone) are put together in the same `docker-compose.yml` file and can still be detangled from one another but will require a tiny bit more hands-on config editing. Nonetheless, instructions are included in the comments of those files for how to detangle them.

# Setup:
1. [Create a network for your containers to talk to each other on](https://docs.docker.com/engine/reference/commandline/network_create/)
2. `git clone https://https://github.com/Snaacky/automation`
3. Configure the `docker-compose.yml` files for your personal setup.
4. *Optional: Generate a `.rclone.conf` (or multiple) if you plan on using rclone remotes.*
5. *Optional: Generate a `wg0.conf` WireGuard VPN config if you plan on putting Transmission behind a VPN.*
6. `docker-compose up -d` each of the `docker-compose.yml` files.

# Folder layout
Most of my containers are hosted in `/srv/containers` and the layout looks something like this:
```
~/containers$ tree -d
├── flood
│   └── config/
│   └── docker-compose.yml
├── plex
│   └── docker-compose.yml
├── prowlarr
│   └── config/
│   └── docker-compose.yml
├── radarr
│   └── config/
│   └── docker-compose.yml
├── rclone
│   └── config/
│       └── mount_one_config/
│           └── .rclone.conf
│       └── mount_two_config/
│           └── .rclone.conf
├── requestrr
│   └── config/
│   └── docker-compose.yml
├── sabnzbd
│   └── config/
│   └── docker-compose.yml
├── sonarr
│   └── config/
│   └── docker-compose.yml
└── transmission
    ├── config/
    │   └── wg0.conf (only if using WireGuard VPN container)
    └── watch/
    └── docker-compose.yml
```
Every Docker container has its own parent folder named after the service for storing the relevant data. Inside the parent folder is the `docker-compose.yml` file and the `/config` folder that contains any sort of configuration files or any files that need to be persistently stored on the host OS.

For containers that use larger amounts of data (such as Plex's metadata) or for storing downloads, I use a mergefs mount to bind 3x8TB drives together at `/mnt/pool` and store data there rather than in the `/srv/containers` folder on my storage limited host OS SSD. This is where I store my downloads and my Plex library.

```
/mnt/pool$ tree -d
├── containers
│   └── plex
│       └── config
├── downloads
│   └── nzbs/
│   └── torrents/
└── media
    ├── anime/
    ├── cartoons/
    ├── movies/
    └── rclone/
    │   └── cache/
    │   └── drive_1/
    │   └── drive_2/
    └── tv/
   ```
   
# Service breakdown

[Sonarr](https://sonarr.tv/) is used for interacting with TV show torrent trackers and Usenet indexers. This service automatically checks for and downloads the `.torrent` or `.nzb` for the latest episodes of a TV show and sends them to your BitTorrent or Usenet client for downloading. In my case, I also have connections established under `Settings -> Connect` to connect my Sonarr instance with my Discord server and my Plex Media Server. 

[Radarr](https://radarr.video/) is used for interacting with movies torrent trackers and Usenet indexers. This service automatically checks for and downloads the `.torrent` or `.nzb` for movies and sends them to your BitTorrent or Usenet for downloading. In my case, I also have connections established under `Settings -> Connect` to connect my Radarr instance with my Discord server and my Plex Media Server.

[Transmission](https://transmissionbt.com/) is an extremely stable BitTorrent client used for downloading `.torrent` files. In my case, I am using a 3rd party web UI for Transmission called Flood which hooks into it via the Transmission API and provides a better UI/UX experience than stock Transmission. I also have a WireGuard VPN container in front of my Transmission container, routing all of my Transmission traffic through the VPN.

[Prowlarr](https://github.com/Prowlarr/Prowlarr) is used for connecting to torrent trackers that do not natively support Sonarr or Radarr via a Torznab proxy. It will automatically link in with your Sonarr and Radarr once setup properly and doesn't require manual setup for both *arr services unlike Jackett. I primarily use torrent trackers and Usenet indexers that natively link in with *arr services but I use Prowlarr for the ones that do not.

[Requestrr](https://github.com/darkalfx/requestrr) is a Discord bot that can be added to a server and used for easily processing requests without having to reveal your Sonarr or Radarr credentials to 3rd parties. It's extremely useful if you're offering a Plex share for others and it also allows you to split TV and anime by category for individual *arr profiles, language, and save pathing unlike Ombi.

[SABnzbd](https://sabnzbd.org/) is a Usenet client for downloading `.nzb` files. You can think of it like Transmission but for Usenet.

[Plex](https://www.plex.tv/) is home theater media server but you probably already know what Plex is if you're here in the first place.

[rclone](https://rclone.org/) is a command line based program used for interacting with files on cloud storage. It is most prominently known for the ability to mount a cloud storage drive to your local file system. I use rclone to mount files from my Google Drive to my local file system and play those files via Plex.

[WireGuard](https://www.wireguard.com/) is a fast, more modern, and secure alternative to OpenVPN. I use WireGuard for routing traffic from my Transmission container securely to my VPN provider.

# Security warning

I highly recommend that you use the WireGuard VPN container to proxy Transmission traffic if you use an ISP or live in a country that cares about BitTorrent traffic. 

Ensure that all of your containers are properly protected with a secure and unique username and password for accessing their respective web UIs. Only Transmission requires a username and password out of the box and all the others are unsecured by default. 

Furthermore, you should never port forward an unsecured container through your firewall as you will be exposing all of your BitTorrent and Usenet information onto the Internet for anyone to steal.
