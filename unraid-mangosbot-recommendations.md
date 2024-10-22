# Additional Recommendations for MangosBot on Unraid

## Backup Strategy

1. **Database Backups**
```bash
# Add to your Unraid user scripts
#!/bin/bash
timestamp=$(date +%Y%m%d_%H%M%S)
backup_dir="/mnt/user/appdata/mangosbot/db_backups"
mkdir -p "$backup_dir"

# Dump all relevant databases
docker exec mangosbot-db mysqldump -u root -pmangos classiccharacters > "$backup_dir/classiccharacters_$timestamp.sql"
docker exec mangosbot-db mysqldump -u root -pmangos classicmangos > "$backup_dir/classicmangos_$timestamp.sql"
docker exec mangosbot-db mysqldump -u root -pmangos classicrealmd > "$backup_dir/classicrealmd_$timestamp.sql"
docker exec mangosbot-db mysqldump -u root -pmangos classicplayerbots > "$backup_dir/classicplayerbots_$timestamp.sql"

# Keep only last 7 days of backups
find "$backup_dir" -type f -mtime +7 -delete
```

## Performance Optimization

1. **Cache Drive Usage**
   - Consider moving the database files to the cache drive:
   ```
   /mnt/cache/appdata/mangosbot/db
   ```
   - Add the following to your share settings:
   ```
   Use Cache (yes)
   ```

2. **Memory Settings**
   - For MariaDB container:
   ```
   Add these environment variables:
   MYSQL_INNODB_BUFFER_POOL_SIZE=1G
   MYSQL_MAX_CONNECTIONS=200
   ```

3. **Network Optimization**
   ```
   Add to Extra Parameters for all containers:
   --network-timeout=60
   ```

## Monitoring Setup

1. **Resource Monitoring**
   - Install Netdata or Grafana docker containers
   - Add these labels to your containers:
   ```
   org.label-schema.group=mangosbot
   ```

2. **Log Management**
   ```
   Add to Extra Parameters:
   --log-opt max-size=10m
   --log-opt max-file=3
   ```

## Security Enhancements

1. **Network Isolation**
   ```
   # Create a more isolated network
   docker network create \
     --driver=bridge \
     --subnet=172.20.0.0/16 \
     --ip-range=172.20.10.0/24 \
     mangosbot-network
   ```

2. **Database Security**
   - Use Docker secrets instead of environment variables for passwords
   - Add to MariaDB:
   ```
   Environment Variables:
   MYSQL_ROOT_PASSWORD_FILE=/run/secrets/db_root_password
   ```

3. **Container Hardening**
   ```
   Add to Extra Parameters:
   --read-only
   --tmpfs /tmp:rw,noexec,nosuid,size=1g
   ```

## Quality of Life Improvements

1. **Container Labels**
   ```
   Add to each container:
   Labels:
   - MAINTAINER=YourName
   - SERVICE_NAME=mangosbot
   - BACKUP=yes
   ```

2. **Health Check Web UI**
   Create a simple status page container:
```
Name: mangosbot-status
Repository: healthchecks/healthchecks
Ports: 8000:8000
Environment Variables:
ALLOWED_HOSTS=*
SITE_ROOT=http://your-unraid-ip:8000
```

3. **Automatic Updates**
   Install Watchtower container for automated updates:
```
Name: watchtower
Repository: containrrr/watchtower
Volumes: /var/run/docker.sock:/var/run/docker.sock
Environment Variables:
WATCHTOWER_SCHEDULE=0 0 4 * * *
WATCHTOWER_CLEANUP=true
WATCHTOWER_LABEL_ENABLE=true
```

## Maintenance Scripts

1. **Server Maintenance Script**
```bash
#!/bin/bash
# Add to Unraid user scripts

# Stop all containers gracefully
docker stop mangosbot-realmd
docker stop mangosbot-mangosd
docker stop mangosbot-db

# Backup databases
# ... (backup code from above)

# Check and repair database tables
docker start mangosbot-db
sleep 30
docker exec mangosbot-db mysqlcheck -u root -pmangos --auto-repair --optimize --all-databases

# Restart services
docker start mangosbot-db
sleep 30
docker start mangosbot-mangosd
sleep 10
docker start mangosbot-realmd
```

2. **Log Rotation**
```bash
#!/bin/bash
# Add to Unraid user scripts

log_dir="/mnt/user/appdata/mangosbot/logs"
find "$log_dir" -type f -name "*.log" -mtime +30 -delete
```

## Development Environment

1. **Local Development Setup**
   Create a development container:
```
Name: mangosbot-dev
Repository: ubuntu:latest
Ports: 22:22
Volumes: /mnt/user/appdata/mangosbot:/mangosbot
Environment Variables:
PUID=1028
PGID=65537
```

## Documentation

1. Create a documentation folder:
```
/mnt/user/appdata/mangosbot/docs/
```

2. Add these essential documents:
   - Setup instructions
   - Network diagram
   - Backup/restore procedures
   - Common troubleshooting steps
   - Configuration reference

## Recovery Procedures

1. **Quick Recovery Script**
```bash
#!/bin/bash
# Add to Unraid user scripts

# Variables
BACKUP_DIR="/mnt/user/appdata/mangosbot/db_backups"
LATEST_BACKUP=$(ls -t "$BACKUP_DIR"/*.sql | head -n 1)

# Stop services
docker stop mangosbot-realmd
docker stop mangosbot-mangosd
docker stop mangosbot-db

# Restore latest backup
docker start mangosbot-db
sleep 30
cat "$LATEST_BACKUP" | docker exec -i mangosbot-db mysql -u root -pmangos

# Restart services
docker start mangosbot-mangosd
sleep 10
docker start mangosbot-realmd
```

Would you like me to expand on any of these recommendations or provide additional details for specific areas?