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

## 3️⃣ File `docker-compose.yml` Nextcloud

➡️ Esempio file docker-compose.yml pronto da usare:
[compose/nextcloud-docker-compose](../compose/nextcloud-docker-compose.yml)


```yaml
version: "3.9"

services:
  nextcloud:
    image: nextcloud:latest
    container_name: nextcloud
    restart: unless-stopped

    dns:
      - 172.23.0.100
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

    # se si vuole impostare un dominio 
      NEXTCLOUD_TRUSTED_DOMAINS: ufficio.miodominio.it
      OVERWRITEHOST: ufficio.miodominio.it.it
      OVERWRITEPROTOCOL: https

    volumes:
      # se si vuole usare una cartella dedicata per i dati (consigliato)
      # - /mnt/nextcloud-data:/var/www/html
      - ./nextcloud/html:/var/www/html
      - ./docker-entrypoint-hooks.d:/docker-entrypoint-hooks.d


    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.nextcloud.rule=Host(`ufficio.local`)"
      # - "traefik.http.routers.nextcloud.rule=Host(`ufficio.miodominio.it`)"
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

## 5. Avviare Nextcloud

➡️ Esempio file .env pronto da usare:
[compose/.env](../compose/.env)


- Ricordarsi di creare il file .env con le password.
- Avviare il docker

```bash
cd nextcloud
docker-compose up -d
```

- Dashboard visibile su `https://ufficio.local`  
- Usa le credenziali impostate nelle labels per autenticarti

---

## 6. Verifiche

Nextcloud sarà disponibile su HTTPS all’indirizzo:

```bash
https://office.local
```
- **MariaDB**:  cambia le password di default (supersecret e nextsecret) per sicurezza.
- **Redis**:  non richiede password interna, ma può essere configurata se necessario.
- **Volumes**:  db_data, redis_data e nextcloud_data garantiscono persistenza dei dati.
- **Accesso alla dashboard di Nextcloud**:  la prima volta seguirai il wizard di setup per creare l’admin


Per l'installazione di nextcloud, assicurarsi che il servizio mariadb sia nella stessa rete di nextcloud

```bash
docker network inspect lanufficio   # deve esistere
docker network disconnect lanufficio mariadb # per scollegare mariadb dalla rete
docker network connect --ip 172.23.0.103 lanufficio mariadb #per collegarlo manualmente 
```

Se si vogliono installare gli strumenti di diagnosi della rete:
```bash
apt update && apt install -y dnsutils 
nslookup collabora.local
```
Nota: i pacchetti installati spariscono al prossimo docker-compose up --force-recreate, perché non sono persistenti.

## 6. Ottimizzazioni

Per forzare nextcloud ad aprire solo pagine con il prefisso https aggiornare/aggiungere nel file config.php di nextcloud:
```php
'overwrite.cli.url' => 'https://ufficio.local',
'overwriteprotocol' => 'https',
'trusted_proxies' => ['172.23.0.3'],
```

Se necessario verificare che docker-compose di nextcloud le label di traefik siano le seguenti:
```yaml
labels:
  - "traefik.http.routers.nextcloud.rule=Host(`ufficio.local`)"
  - "traefik.http.routers.nextcloud.entrypoints=websecure"
  - "traefik.http.routers.nextcloud.tls=true"
  - "traefik.http.services.nextcloud.loadbalancer.server.port=80"
  ```