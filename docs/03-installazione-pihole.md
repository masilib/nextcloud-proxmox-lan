# Installazione Pi-hole su Docker per gestione DNS locale

Questa guida mostra come installare Pi-hole in un container Docker per gestire i domini locali della LAN e filtrare pubblicità/ads.

---

## 1️⃣ Prerequisiti

- Container Alpine Linux con Docker e Docker Compose già installati
- Rete locale configurata (LAN)
- IP statico riservato per Pi-hole (es. `192.168.1.80`)
- Docker Compose versione ≥ 1.27

---

## 2️⃣ Creare cartella di lavoro

```bash
mkdir -p pihole
cd pihole
```

- La cartella conterrà i dati persistenti (`/etc/pihole` e `/etc/dnsmasq.d`)  

---

## 3️⃣ Creare il file `docker-compose.yml`

```bash
nano docker-compose.yml
```

Incolla dentro:

```yaml
version: '3.8'

services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    hostname: pihole
    networks:
      lanufficio:
        ipv4_address: 172.23.0.100
    ports:
      - "192.168.1.80:53:53/tcp"
      - "192.168.1.80:53:53/udp"
      - "192.168.1.80:8089:80/tcp"
      #- "192.168.1.80:443:443/tcp"
    environment:
      TZ: 'Europe/Rome'
      WEBPASSWORD: '123456'  # Cambia questa password!
      FTLCONF_LOCAL_IPV4: '192.168.1.80'
      PIHOLE_DNS_: '8.8.8.8;1.1.1.1'
      DNSSEC: 'false'
      PIHOLE_INTERFACE: 'eth0'
      DNSMASQ_LISTENING: 'all'
    volumes:
      - './pihole/etc-pihole:/etc/pihole'
      - './pihole/etc-dnsmasq.d:/etc/dnsmasq.d'
    cap_add:
      - NET_ADMIN
    restart: unless-stopped
    dns:
      - 127.0.0.1
      - 8.8.8.8

networks:
  lanufficio:
    external: true
```

> ⚠️ Assicurati che la rete `lanufficio` esista già su Docker (`docker network ls`)  
> e corrisponda alla LAN locale del container.
> Se la rete non è stata ancora creata:
> Per una rete bridge con subnet specifica:
```bash
docker network create \
  --driver=bridge \
  --subnet=172.23.0.0/24 \
  lanufficio
```

---

## 4️⃣ Avviare Pi-hole

```bash
docker-compose up -d
```

- Il container verrà creato e avviato in background  
- I dati saranno persistenti nella cartella `./pihole`

---

## 5️⃣ Accesso all'interfaccia web

Apri il browser su:

```
http://192.168.1.80:8089
```

- Login: password impostata in `WEBPASSWORD`  
- Puoi configurare domini locali, blacklist, whitelist e statistiche DNS

---

## 6️⃣ Note importanti

- Per utilizzare Pi-hole come DNS della rete, configura il router o i client LAN con IP del container `192.168.1.80`  
- Cambia sempre la password predefinita (`WEBPASSWORD`)  
- Puoi aggiornare Pi-hole e il container con:

```bash
docker-compose pull
docker-compose up -d
```

- Assicurati di avere Docker Compose aggiornato e la rete esterna `lanufficio` correttamente configurata.
