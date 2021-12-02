# automation
Home theater automation scripts for downloading and processing.

# Setup
1. Create a network for your containers to talk to each other on (https://docs.docker.com/engine/reference/commandline/network_create/)
2. `git clone https://https://github.com/Snaacky/automation`
3. Edit the relevant config files for your personal situation.
4. Optional: Generate a `.rclone.conf` file for your Plex `docker-compose.yml` if you plan on using rclone remotes.
5. Optional: Generate a `wg0.conf` WireGuard VPN config file for your Transmission instance if you plan on putting your Tranmission container behind a VPN.
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

Every Docker container has its own folder for the relevant services data. This usually means the parent directory, the docker-compose.yml within the folder, and a config folder for storing any sort of persistent data on the host.

For containers that use larger amounts of data or for storing downloads, I use a mergefs mount to bind 3x8TB drives together at `/mnt/pool` and store data there rather than on my limited host OS SSD. This is where I store my downloads and my Plex library.

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
