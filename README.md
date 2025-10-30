# Nextcloud su Proxmox (LAN)

Questa repository contiene la documentazione e i file di esempio per configurare un server Nextcloud in una rete locale (LAN) utilizzando:

- Proxmox LXC non privilegiato
- Alpine Linux come OS del container
- Docker + Docker Compose
- Portainer per gestione dei container

---

## Contenuti della repository

- `docs/` → Documentazione passo passo in Markdown (compatibile con MkDocs)  
- `lxc/` → File di configurazione LXC di esempio (`<ID>.conf`)  
- `compose/` → File `docker-compose.yml` pronti all’uso  
- `scripts/` → Script utili (backup, restore, mount storage)  

---

## Documentazione principale

- [Creazione Container LXC](docs/01-creazione-container.md) – Configurazione container su Proxmox con Alpine Linux  
- [Installazione Docker + Portainer](docs/02-installazione-docker.md) – Setup ambiente Docker e Portainer  
- [PiHole](docs/03-installazione-pihole.md) – Configurazione container PiHole
- [Traefik](docs/04-04-installazione-traefik.md) – Configurazione container Traefik  
- [Nextcloud + MariaDB](docs/05-installazione-nextcloud.md) – Configurazione container Nextcloud con database MariaDB  
- [Script utili](scripts/) – Script di backup, restore e gestione storage  

---

## Obiettivo del progetto

- Creare un ambiente Nextcloud **completamente in LAN**, senza esposizione diretta a Internet  
- Fornire documentazione chiara e completa con esempi pronti  
- Rendere semplice l’aggiunta di altri container Docker gestiti via Portainer
