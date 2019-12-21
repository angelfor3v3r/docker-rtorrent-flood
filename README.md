# docker-rtorrent-flood
[rTorrent](https://github.com/rakshasa/rtorrent "rTorrent's Github") & [Flood](https://github.com/Flood-UI/flood "Flood's Github") for Docker.

## Usage
1. Edit `docker-compose.yml` and set up your volumes for rTorrent:
```
# Your session, files and watch directories should be relative to the "/rtorrent" directrory. Heres an example:
# rtorrent.rc must be in the "/rtorrent" directory.
services:
    # rTorrent service
    rtorrent:
        ...
        
        # replace "session_path", "files_path" and "watch_path" with your own directories.
        volumes:
            - "/session_path:/rtorrent/.session" # rtorrent session
            - "/files_path:/rtorrent/files"      # rtorrent files
            - "/watch_path:/rtorrent/watch"      # rtorrent watch
```

2. Edit `docker-compose.yml` and set up your volumes and environment for Flood:
```
# flood service (a web UI for rTorrent)
    flood:
        ...
        
        # replace "flood_data_path", "files_path" and "watch_path" with your own directories.
        # note that "scgi.sock" must come from your "rtorrent session" path in step #1.
        # ... and "files_path" must be the same as step #1.
        volumes:
            - "/flood_data_path:/data"             # flood data
            - "/files_path:/rtorrent/files"        # rtorrent files (local path should be the same as rTorrent)
            - "/session_path/scgi.sock:/scgi.sock" # rtorrent SCGI socket file

        environment: # https://github.com/jfurrow/flood/blob/master/config.docker.js
            RTORRENT_SOCK_PATH: "/scgi.sock" # from the volume above
            FLOOD_SECRET: "my_cool_secret"
```

3. Run `docker-compose up -d`.

4. Access `server ip(localhost):3000` and create an account for Flood with "rTorrent Connection Type" set to "Unix Socket" and enter the value for your "RTORRENT_SOCK_PATH".

## Contributing
Pull requests and suggestions are welcome.

## TODO
...

## Other notes
hmmm...,,
