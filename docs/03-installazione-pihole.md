# Installazione Pi-hole su Docker per gestione DNS locale

Questa guida mostra come installare Pi-hole in un container Docker per gestire i domini locali della LAN e filtrare pubblicità/ads.

---

## Prerequisiti

- Container Alpine Linux con Docker e Docker Compose già installati
- Rete locale configurata (LAN)
- IP statico riservato per Pi-hole (es. `192.168.1.80`)
- Docker Compose versione ≥ 1.27

---

## Creare cartella di lavoro

```bash
mkdir -p pihole
cd pihole
```

- La cartella conterrà i dati persistenti (`/etc/pihole` e `/etc/dnsmasq.d`)  

---

## Creare il file `docker-compose.yml`
➡️ Esempio file docker-compose.yml pronto da usare:
[compose/pihole-docker-compose](../compose/pihole-docker-compose.yml)

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

**Spiegazione dei parametri:**

| Opzione | Significato |
|---------|-------------|
| `--driver=bridge` | Tipo di rete interna Docker (bridge) |
| `--subnet=172.23.0.0/24` | Subnet privata per i container |
| `lanufficio` | Nome della rete, da usare nel docker-compose |

> L’IP statico del container Pi-hole (`172.23.0.100`) deve stare all’interno di questa subnet.

---

### Controllare la rete creata

```bash
docker network inspect lanufficio
```

- Controlla che la subnet sia corretta e pronta all’uso

---

## Avviare Pi-hole

```bash
docker-compose up -d
```

- Il container verrà creato e avviato in background  
- I dati saranno persistenti nella cartella `./pihole`

---

## Accesso all'interfaccia web

Apri il browser su:

```
http://192.168.1.80:8089
```

- Login: password impostata in `WEBPASSWORD`  
- Puoi configurare domini locali, blacklist, whitelist e statistiche DNS

### Nuova Password

-Se si vuole impostare una nuova password, si puà lanciare il seguente comando direttamente dal container Docker

```bash
docker exec -it pihole bash
pihole setpassword
exit
```
- Questo ti apre una shell dentro il container pihole
- Lanciare il comando per impostare una nuova password scelta dall’utente

---

## Note importanti

- Per utilizzare Pi-hole come DNS della rete, configura il router o i client LAN con IP del container `192.168.1.80`  
- Cambia sempre la password predefinita (`WEBPASSWORD`)
- In Pi-Hole -> Settings -> DNS -> Expert impostare 'Permit all origins'  
- Puoi aggiornare Pi-hole e il container con:

```bash
docker-compose pull
docker-compose up -d
```

- Assicurati di avere Docker Compose aggiornato e la rete esterna `lanufficio` correttamente configurata.

## Perché gestire la rete con Pi-hole?

- **DNS locale centralizzato**: tutti i dispositivi della LAN possono usare Pi-hole come DNS principale  
- **Filtraggio pubblicità e tracker**: blocca automaticamente ads e tracker su tutta la rete  
- **Gestione domini locali**: puoi creare record DNS locali per container o server interni (es. `nextcloud.lan`)  
- **Statistiche e monitoraggio**: Pi-hole fornisce report su richieste DNS, dispositivi e domini visitati  
- **Facilità di configurazione**: cambiando il DNS del router o dei client, tutti beneficiano del filtraggio e della risoluzione locale senza modifiche individuali

> In pratica, Pi-hole diventa il **gestore centrale della rete**, rendendo più semplice configurare e accedere ai servizi locali come Nextcloud, Portainer o altri container.
