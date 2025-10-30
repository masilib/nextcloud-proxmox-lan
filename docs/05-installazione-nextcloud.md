# Nextcloud + MariaDB + Redis

In questo capitolo configuriamo **Nextcloud** con **MariaDB** come database e **Redis** per caching e gestione delle sessioni, il tutto in **Docker** su rete locale `lanufficio` con Traefik che gestisce il TLS.

---

## 1. Motivazioni

- **Nextcloud**: soluzione cloud self-hosted per file, calendario, contatti e collaborazioni.  
- **MariaDB**: database relazionale per Nextcloud.  
- **Redis**: migliora le prestazioni grazie al caching e alla gestione delle sessioni.  

Docker ci permette di avere isolamento, aggiornamenti semplici e gestione centralizzata dei servizi.

---

## 2️⃣ Cartella di lavoro

```bash
mkdir -p nextcloud
cd nextcloud
```

Salva qui il `docker-compose.yml`, `.env`

---

## 3️⃣ File `docker-compose.yml` Traefik
```yaml
version: "3.9"

services:
  nextcloud:
    image: nextcloud:latest
    container_name: nextcloud
    restart: unless-stopped

    #extra_hosts:
    #  - "collabora.local:172.23.0.3"  # IP del container Collabora

    dns:
      - 192.168.1.80
    networks:
      lanufficio:
        ipv4_address: 172.23.0.102

    depends_on:
      - db
      - redis

    environment:
      MYSQL_HOST: db
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
      MYSQL_DATABASE: nextcloud
      MYSQL_USER: nextcloud
      NEXTCLOUD_ADMIN_USER: ${NC_ADMIN}
      NEXTCLOUD_ADMIN_PASSWORD: ${NC_PASSWORD}
      REDIS_HOST: redis

    volumes:
      - ./nextcloud/html:/var/www/html
      - ./docker-entrypoint-hooks.d:/docker-entrypoint-hooks.d


    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.nextcloud.rule=Host(`ufficio.local`)"
      - "traefik.http.routers.nextcloud.entrypoints=websecure"
      - "traefik.http.routers.nextcloud.tls=true"
      - "traefik.docker.network=lanufficio"
      - "traefik.http.routers.nextcloud.tls.certresolver=myresolver"
      - "traefik.http.services.nextcloud.loadbalancer.server.port=80"
  
  db:
    image: mariadb:11
    container_name: mariadb
    restart: unless-stopped

    networks:
      lanufficio:
        ipv4_address: 172.23.0.103

    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT}
      MYSQL_PASSWORD: ${MYSQL_PASSWORD}
      MYSQL_DATABASE: nextcloud
      MYSQL_USER: nextcloud
      TZ: Europe/Rome

    command: --transaction-isolation=READ-COMMITTED --innodb_read_only_compressed=OFF

    volumes:
      - ./database:/var/lib/mysql

  redis:
    image: redis:alpine
    container_name: redis
    restart: unless-stopped
    networks:
      lanufficio:
        ipv4_address: 172.23.0.104

    environment:
      TZ: Europe/Rome

networks:
  lanufficio:
    external: true
```

## 5. Collegamento con Traefik

Nextcloud sarà disponibile su HTTPS all’indirizzo:

```bash
https://office.local
```
- **MariaDB**:  cambia le password di default (supersecret e nextsecret) per sicurezza.
- **Redis**:  non richiede password interna, ma può essere configurata se necessario.
- **Volumes**:  db_data, redis_data e nextcloud_data garantiscono persistenza dei dati.
- **Accesso alla dashboard di Nextcloud**:  la prima volta seguirai il wizard di setup per creare l’admin

## 6. Verifiche

Per l'installazione di nextcloud, assicurarsi che il servizio mariadb sia nella stessa rete di nextcloud

```bash
docker network inspect lanufficio   # deve esistere
docker network disconnect lanufficio mariadb # per scollegare mariadb dalla rete
docker network connect --ip 172.23.0.103 lanufficio mariadb #per collegarlo manualmente 
```