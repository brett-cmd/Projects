#Docker_LibreNMS 
# Docker

An official LibreNMS docker image based on Alpine Linux and Nginx is available on [DockerHub](https://hub.docker.com/r/librenms/librenms/).

# Documentation

Full install and configuration documentation can be found on the [GitHub repository](https://github.com/librenms/docker).

# Quick install

1. Install docker: https://docs.docker.com/engine/install/
2. Download and unzip composer files:
    
    `mkdir librenms 
    `cd librenms 
    `wget https://github.com/librenms/docker/archive/refs/heads/master.zip 
    `unzip master.zip 
    `cd docker-master/examples/compose`
    
3. Set a new mysql password in .env and inspect compose.yml. 
	.env:
	```yml TZ=Europe/Paris
PUID=1000
PGID=1000

MYSQL_DATABASE=librenms
MYSQL_USER=librenms
MYSQL_PASSWORD=librenms #change password for securirty

``` 

4. Many of the defaults here can be left the same. You made to add some additional configuration in order to get syslog to work, however this will be enough to get you up and running. 

```yml
name: librenms

services:
  db:
    image: mariadb:10.5
    container_name: librenms_db
    command:
      - "mysqld"
      - "--innodb-file-per-table=1"
      - "--lower-case-table-names=0"
      - "--character-set-server=utf8mb4"
      - "--collation-server=utf8mb4_unicode_ci"
    volumes:
      - "./db:/var/lib/mysql"
    environment:
      - "TZ=${TZ}"
      - "MYSQL_ALLOW_EMPTY_PASSWORD=yes"
      - "MYSQL_DATABASE=${MYSQL_DATABASE}"
      - "MYSQL_USER=${MYSQL_USER}"
      - "MYSQL_PASSWORD=${MYSQL_PASSWORD}"
    restart: always

  redis:
    image: redis:5.0-alpine
    container_name: librenms_redis
    environment:
      - "TZ=${TZ}"
    restart: always

  msmtpd:
    image: crazymax/msmtpd:latest
    container_name: librenms_msmtpd
    env_file:
      - "./msmtpd.env"
    restart: always

  librenms:
    image: librenms/librenms:latest
    container_name: librenms
    hostname: librenms
    cap_add:
      - NET_ADMIN
      - NET_RAW
    ports:
      - target: 8000
        published: 8000
        protocol: tcp
    depends_on:
      - db
      - redis
      - msmtpd
    volumes:
      - "./librenms:/data"
    env_file:
      - "./librenms.env"
    environment:
      - "TZ=${TZ}"
      - "PUID=${PUID}"
      - "PGID=${PGID}"
      - "DB_HOST=db"
      - "DB_NAME=${MYSQL_DATABASE}"
      - "DB_USER=${MYSQL_USER}"
      - "DB_PASSWORD=${MYSQL_PASSWORD}"
      - "DB_TIMEOUT=60"
    restart: always

  dispatcher:
    image: librenms/librenms:latest
    container_name: librenms_dispatcher
    hostname: librenms-dispatcher
    cap_add:
      - NET_ADMIN
      - NET_RAW
    depends_on:
      - librenms
      - redis
    volumes:
      - "./librenms:/data"
    env_file:
      - "./librenms.env"
    environment:
      - "TZ=${TZ}"
      - "PUID=${PUID}"
      - "PGID=${PGID}"
      - "DB_HOST=db"
      - "DB_NAME=${MYSQL_DATABASE}"
      - "DB_USER=${MYSQL_USER}"
      - "DB_PASSWORD=${MYSQL_PASSWORD}"
      - "DB_TIMEOUT=60"
      - "DISPATCHER_NODE_ID=dispatcher1"
      - "SIDECAR_DISPATCHER=1"
    restart: always

  syslogng:
    image: librenms/librenms:latest
    container_name: librenms_syslogng
    hostname: librenms-syslogng
    cap_add:
      - NET_ADMIN
      - NET_RAW
    depends_on:
      - librenms
      - redis
    ports:
      - target: 514
        published: 514
        protocol: tcp
      - target: 514
        published: 514
        protocol: udp
    volumes:
      - "./librenms:/data"
    env_file:
      - "./librenms.env"
    environment:
      - "TZ=${TZ}"
      - "PUID=${PUID}"
      - "PGID=${PGID}"
      - "DB_HOST=db"
      - "DB_NAME=${MYSQL_DATABASE}"
      - "DB_USER=${MYSQL_USER}"
      - "DB_PASSWORD=${MYSQL_PASSWORD}"
      - "DB_TIMEOUT=60"
      - "SIDECAR_SYSLOGNG=1"
    restart: always

  snmptrapd:
    image: librenms/librenms:latest
    container_name: librenms_snmptrapd
    hostname: librenms-snmptrapd
    cap_add:
      - NET_ADMIN
      - NET_RAW
    depends_on:
      - librenms
      - redis
    ports:
      - target: 162
        published: 162
        protocol: tcp
      - target: 162
        published: 162
        protocol: udp
    volumes:
      - "./librenms:/data"
    env_file:
      - "./librenms.env"
    environment:
      - "TZ=${TZ}"
      - "PUID=${PUID}"
      - "PGID=${PGID}"
      - "DB_HOST=db"
      - "DB_NAME=${MYSQL_DATABASE}"
      - "DB_USER=${MYSQL_USER}"
      - "DB_PASSWORD=${MYSQL_PASSWORD}"
      - "DB_TIMEOUT=60"
      - "SIDECAR_SNMPTRAPD=1"
    restart: always
```


5. Update librenms.env and modify the IP address from /32 to whatever you're currently using (0.0.0.0/24) 

         ![[Pasted image 20240925153746.png]]

7. Bring up the docker containers
    
    `sudo docker compose -f compose.yml up -d`
    
5. Open webui to finish configuration. `http://localhost:8000` (use the correct ip or name instead of localhost)


6. Once you log in for the first time, you should be prompted to set and administrator user password.




[[[LibreNMS]]]

#snmp #monitor 

