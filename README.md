# automation
Home theater automation scripts for downloading and processing.

# Setup
1. Create a network for your containers to talk to each other on (https://docs.docker.com/engine/reference/commandline/network_create/)
2. `git clone https://https://github.com/Snaacky/automation`
3. Edit the relevant config files for your personal situation.
4. Optional: Generate a `.rclone.conf` (or multiple) if you plan on using rclone remotes.
5. Optional: Generate a `wg0.conf` WireGuard VPN config if you plan on putting Transmission behind a VPN.
6. `docker-compose up -d` each of the `docker-compose.yml` files.

# Folder layout
Most of my containers are hosted in `/srv/containers` and the layout looks something like this:
```
~/containers$ tree -d
├── flood
│   └── config/
│   └── docker-compose.yml
├── plex
│   └── config/
│       └── rclone/
│           ├── drive_1_config/
│           └── drive_2_config/
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
    │   └── wg0.conf
    └── watch/
    └── docker-compose.yml
```

Every Docker container has its own parent folder named after the service for storing the relevant data. Inside the parent folder is the `docker-compose.yml` file and the `/config` folder that contains any sort of configuration files or any files that need to be persistently stored on the host OS.

For containers that use larger amounts of data or for storing downloads, I use a mergefs mount to bind 3x8TB drives together at `/mnt/pool` and store data there rather than in the `/srv/containers` folder on my storage limited host OS SSD. This is where I store my downloads and my Plex library.

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
