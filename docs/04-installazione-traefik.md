# Configurazione Traefik come Reverse Proxy in LAN

Questa guida mostra come installare Traefik come reverse proxy in Docker per gestire Nextcloud e altri servizi locali con HTTPS sul dominio `ufficio.local`.

---

## 1️⃣ Motivazioni per usare Traefik

- **Reverse Proxy centralizzato**: un unico punto di ingresso per tutti i container
- **HTTPS centralizzato**: gestione certificati SSL (ACME / self-signed)
- **Routing dinamico**: rileva automaticamente i container con le label Docker
- **Dashboard di monitoraggio**: permette di vedere lo stato dei router e dei container
- **Gestione domini locali**: puoi usare nomi come `nextcloud.ufficio.local` senza modificare ogni container

---

## 2️⃣ Cartella di lavoro

```bash
mkdir -p traefik
cd traefik
```

- Salva qui il `docker-compose.yml`, `traefik.yml`, certificati e file dinamici

---

## 3️⃣ File `docker-compose.yml` Traefik

```yaml
version: '3.8'

services:
  traefik:
    image: traefik:v2.10
    container_name: traefik
    restart: unless-stopped

    networks:
      - lanufficio
    dns:
      - 172.23.0.100

    ports:
      - "80:80"
      - "443:443"
      - "8080:8080"  # Dashboard Traefik (opzionale)

    volumes:
      # Socket Docker per auto-discovery
      - /var/run/docker.sock:/var/run/docker.sock:ro

      # Configurazione Traefik
      - ./traefik.yml:/traefik.yml:ro

      # Certificati SSL
      - ./certs:/certs:ro

      # File di configurazione dinamica
      - ./dynamic:/dynamic:ro

      # Log e dati persistenti
      - ./acme.json:/acme.json

    environment:
      - TZ=Europe/Rome

    labels:
      # Abilita Traefik per se stesso
      - "traefik.enable=true"

      # Dashboard (opzionale)
      - "traefik.http.routers.dashboard.rule=Host(`traefik.local`)"
      - "traefik.http.routers.dashboard.entrypoints=websecure"
      - "traefik.http.routers.dashboard.tls=true"
      - "traefik.http.routers.dashboard.service=api@internal"

      # Autenticazione dashboard (cambia user e password!)
      # Genera con: htpasswd -nb admin password
      - "traefik.http.middlewares.auth.basicauth.users=admin:$$apr1$$eqbGg5RA$$RMoGgmMqOgAxuBkYfgm820"
      - "traefik.http.routers.dashboard.middlewares=auth"

networks:
  lanufficio:
    external: true
```

---

## 4️⃣ Spiegazione delle sezioni principali

| Sezione | Funzione |
|---------|----------|
| `networks: lanufficio` | Collega Traefik alla rete Docker dedicata alla LAN |
| `dns: 172.23.0.100` | Usa Pi-hole come DNS della rete locale |
| `ports: 80/443/8080` | HTTP, HTTPS e dashboard Traefik |
| `volumes` | Persistenza configurazioni, certificati e logs |
| `labels` | Abilita dashboard, routing e autenticazione con base auth |

## 5️⃣ Creare la rete Docker `lanufficio` (se non esiste)

```bash
docker network create \
  --driver=bridge \
  --subnet=172.23.0.0/24 \
  lanufficio
```

- L’IP statico di Pi-hole (`172.23.0.100`) deve stare nella stessa subnet

---

## 6️⃣ Avviare Traefik

```bash
cd traefik
docker-compose up -d
```

- Dashboard visibile su `https://traefik.local:8080`  
- Usa le credenziali impostate nelle labels per autenticarti

---

## 7️⃣ Collegare i container dei servizi a Traefik

- Per ogni container, aggiungi **labels Traefik** nel `docker-compose.yml`:

```yaml
labels:
  - "traefik.enable=true"
  - "traefik.http.routers.nextcloud.rule=Host(`nextcloud.ufficio.local`)"
  - "traefik.http.routers.nextcloud.entrypoints=websecure"
  - "traefik.http.routers.nextcloud.tls=true"
```

- Così Traefik instraderà automaticamente il traffico HTTPS al container corretto

---

## 8️⃣ Aggiornare file hosts dei client LAN

```text
192.168.1.80  nextcloud.ufficio.local
192.168.1.80  traefik.local
```

> Sostituisci `192.168.1.80` con l’IP del container host Traefik  
> In LAN, questo permette di risolvere i domini locali senza un DNS esterno

---

### ✅ Vantaggi dell’uso di Traefik in LAN

- HTTPS centralizzato per tutti i servizi locali  
- Routing automatico dei container senza modifiche manuali ai container  
- Dashboard per monitorare traffico e servizi  
- Facile aggiungere nuovi servizi con poche labels Docker  
- Integrazione con Pi-hole per DNS locale
