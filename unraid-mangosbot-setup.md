# Setting Up MangosBot Stack in Unraid

## Prerequisites
- Unraid server with Docker enabled
- Access to Unraid web interface
- Network share for persistent storage
- Docker Hub account for the prebuilt images

## Storage Setup

First, create the following directory structure on your Unraid share (e.g., under `/mnt/user/appdata/mangosbot/`):

```
mangosbot/
├── backup/
├── bin/
├── config/
├── data/
├── db/
├── logs/
├── scripts/
├── sql/
```

## Container 1: MariaDB (Database)

1. In Unraid, click "Add Container"
2. Fill in the following details:

```
Name: mangosbot-db
Repository: linuxserver/mariadb
Network Type: Custom: mangosbot-network (bridge)

Port Mappings:
- Host Port: 33307
- Container Port: 3306

Environment Variables:
- PUID=1028
- PGID=65537
- TZ=Europe/London
- MYSQL_ROOT_PASSWORD=mangos

Path Mappings:
- Host: /mnt/user/appdata/mangosbot/db
- Container: /config
```

Extra Parameters: 
```
--restart=unless-stopped
--health-cmd="mysqladmin ping -h 127.0.0.1 -u root -p$$MYSQL_ROOT_PASSWORD || exit 1"
--health-interval=10s
--health-timeout=10s
--health-retries=5
```

## Container 2: MangosD Server

1. Click "Add Container"
2. Fill in the following details:

```
Name: mangosbot-mangosd
Repository: [YOUR-DOCKERHUB]/mangosbot-mangosd
Network Type: Custom: mangosbot-network (bridge)

Port Mappings:
- Host Port: 8085
- Container Port: 8085
- Host Port: 3443
- Container Port: 3443
# Optional SOAP port:
# - Host Port: 7878
# - Container Port: 7878

Environment Variables:
- PUID=1028
- PGID=65537
- TZ=Europe/London
- CHARACTERS_DB=classiccharacters
- MANGOSD_DB=classicmangos
- REALMD_DB=classicrealmd
- PLAYERBOTS_DB=classicplayerbots
- DB_SERVER=mangosbot-db
- DB_ROOT_USER=root
- DB_ROOT_PASS=mangos
- DB_PORT=3306
- DB_USER=mangos
- DB_PASS=mangos

Path Mappings:
- Host: /mnt/user/appdata/mangosbot/config
- Container: /opt/cmangos/etc
- Host: /mnt/user/appdata/mangosbot/logs
- Container: /opt/cmangos/logs
- Host: /mnt/user/appdata/mangosbot/data
- Container: /opt/cmangos/data
- Host: /mnt/user/appdata/mangosbot/scripts
- Container: /opt/cmangos/scripts
- Host: /mnt/user/appdata/mangosbot/backup
- Container: /opt/cmangos/backup
- Host: /mnt/user/appdata/mangosbot/sql
- Container: /opt/cmangos/sql
- Host: /mnt/user/appdata/mangosbot/bin
- Container: /opt/cmangos/bin
```

Extra Parameters:
```
--restart=unless-stopped
--stop-timeout 1800
--security-opt=seccomp=unconfined
--cap-add=SYS_PTRACE
```

## Container 3: RealmD Server

1. Click "Add Container"
2. Fill in the following details:

```
Name: mangosbot-realmd
Repository: [YOUR-DOCKERHUB]/mangosbot-realmd
Network Type: Custom: mangosbot-network (bridge)

Port Mappings:
- Host Port: 3724
- Container Port: 3724

Environment Variables:
- PUID=1028
- PGID=65537
- TZ=Europe/London
- REALMD_DB=classicrealmd
- DB_USER=mangos
- DB_PASS=mangos
- DB_SERVER=mangosbot-db
- DB_PORT=3306

Path Mappings:
- Host: /mnt/user/appdata/mangosbot/config
- Container: /opt/cmangos/etc
- Host: /mnt/user/appdata/mangosbot/logs
- Container: /opt/cmangos/logs
- Host: /mnt/user/appdata/mangosbot/bin
- Container: /opt/cmangos/bin
- Host: /mnt/user/appdata/mangosbot/scripts
- Container: /opt/cmangos/scripts
```

Extra Parameters:
```
--restart=unless-stopped
```

## Container Start Order

1. Start the mangosbot-db container first and wait for it to be healthy
2. Start the mangosbot-mangosd container
3. Start the mangosbot-realmd container

## Network Setup Notes

- All containers will automatically join the custom bridge network 'mangosbot-network'
- The containers will be able to communicate with each other using their container names as hostnames
- The database will be accessible from the host system on port 33307

## Important Notes

1. Replace `[YOUR-DOCKERHUB]` with your Docker Hub username where you've pushed the prebuilt images
2. The PUID and PGID values (1028 and 65537) should match your Unraid user permissions
3. All containers use the same timezone (Europe/London) - adjust as needed
4. Adjust memory limits and other resources in Unraid's advanced container settings if needed
5. Make sure all required directories exist before starting the containers

Would you like me to clarify any part of this setup guide?
