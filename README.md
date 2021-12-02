# automation
Home theater automation scripts for downloading and processing.

# Setup
This repository assumes you have a baseline understanding of how Docker works, what a docker-compose file is, and how to configure one.

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
│   └── config/ (only contains rclone mount configs)
│       └── rclone/
│           ├── mount_one_config/
│           └── mount_two_config/
│   └── docker-compose.yml
├── prowlarr
│   └── config/
│   └── docker-compose.yml
├── radarr
│   └── config/
│   └── docker-compose.yml
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

[Sonarr](https://sonarr.tv/) is used for interacting with TV show torrent trackers and Usenet indexers. This service automatically checks for and downloads the .torrent or .nzb for the latest episodes of a TV show and sends them to your torrent client for downloading. In my case, I also have connections established under settings -> Connect to connect my Sonarr instance with my Discord server and my Plex Media Server.

[Radarr](https://radarr.video/) is used for interacting with movies torrent trackers and Usenet indexers. This service automatically checks for and downloads the .torrent or .nzb for movies and sends them to your torrent client for downloading. In my case, I also have connections established under settings -> Connect to connect my Sonarr instance with my Discord server and my Plex Media Server.

[Transmission](https://transmissionbt.com/) is an extremely stable BitTorrent client used for downloading .torrent files. In my case, I am using a 3rd party web UI for Transmission called Flood which hooks into it via the Transmission API and provides a better UI/UX experience than stock Transmission. I also have a WireGuard VPN container in front of my Transmission container, routing all of my Transmission traffic through the VPN.

[Flood](https://flood.js.org/) is a front-end web UI for various torrent clients. It offers mobile responsiveness and a better more modern experience than the stock Transmission web UI.

[Prowlarr](https://github.com/Prowlarr/Prowlarr) is used for connecting to torrent trackers that do not natively support Sonarr or Radarr via a Torznab proxy. It will automatically link in with your Sonarr and Radarr once setup properly and doesn't require manual setup for both *arr services unlike Jackett.

[Requestrr](https://github.com/darkalfx/requestrr) is a Discord bot that can be added to a server and used for easily processing requests without having to reveal your Sonarr or Radarr credentials to 3rd parties. It's extremely useful if you're offering a Plex share for others and it also allows you to split TV and anime by category for individual *arr profiles, language, and save pathing unike Ombi.

[SABnzbd](https://sabnzbd.org/) is a Usenet client for downloading .nzb files. You can think of it like Transmission but for Usenet.

[Plex](https://www.plex.tv/) is home theater media server but you probably already know what Plex is if you're here in the first place.

[rclone](https://rclone.org/) is a command line based program used for interacting with files on cloud storage. It's most prominently used feature is the ability to mount a cloud storage drive to your local file system. In my case, this is used inconjunction with Google Drive to play files from my remote cloud storage accounts on Plex.

[WireGuard](https://www.wireguard.com/) is a fast, more modern, and secure alternative to OpenVPN. In my case, it's used for routing traffic from my Transmission container securely to my VPN provider.
