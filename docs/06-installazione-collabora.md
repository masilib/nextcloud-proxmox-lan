# Collabora per Nextcloud

In questo capitolo configuriamo **Collabora**. 

Il modulo Collabora per Nextcloud (chiamato anche Collabora Online o Nextcloud Office) è l’integrazione tra Nextcloud e Collabora Online, una suite di office collaborativo basata su LibreOffice.
Serve per creare, modificare e collaborare in tempo reale su documenti di testo, fogli di calcolo e presentazioni direttamente all’interno dell’interfaccia web di Nextcloud, senza bisogno di scaricare i file.
https://collabora.local

Assicurati che Traefik abbia o possa generare il certificato per collabora.local. Se usi un'autorità certificativa locale (come mkcert), genera anche il certificato per Collabora.

---

## 1. Come funziona

- **Collabora Online**: è un’applicazione server (derivata da LibreOffice Online) che elabora i documenti.
- **Collabora Online Connector**: in Nextcloud fa da ponte tra i due, consentendo l’editing in browser. 


---

## 2. Funzionalità principali

- **Editing collaborativo in tempo reale**: più utenti possono scrivere contemporaneamente e vedere le modifiche live.
- **Compatibilità con i formati Microsoft Office**: (.docx, .xlsx, .pptx) e LibreOffice/OpenDocument (.odt, .ods, .odp). 
- **Commenti e suggerimenti**: (solo visualizzazione, modifica, commento, ecc.). 
- **Anteprima e visualizzazione diretta**: dei documenti in Nextcloud Files. 
- **Integrazione con Nextcloud Talk**: collaborazione e chat mentre si lavora su un documento. 
- **Supporto per modelli**: (template) personalizzati. 

## 2️⃣ Cartella di lavoro

```bash
mkdir -p collabora
cd collabora
```

➡️ Esempio file docker-compose.yml pronto da usare:
[compose/collabora-docker-compose](../compose/collabora-docker-compose.yml)

---

## 3️⃣ File `docker-compose.yml` Collabora
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

## 5. Avviare Nextcloud

- Ricordarsi di creare il file .env con le password.
- Avviare il docker

```bash
cd nextcloud
docker-compose up -d
```

- Dashboard visibile su `https://ufficio.local:8080`  
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