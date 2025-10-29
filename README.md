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

- [Creazione Container LXC](docs/container.md) – Configurazione container su Proxmox con Alpine Linux  
- [Installazione Docker + Portainer](docs/docker.md) – Setup ambiente Docker e Portainer  
- [Nextcloud + MariaDB](docs/nextcloud.md) – Configurazione container Nextcloud con database MariaDB  
- [Script utili](scripts/) – Script di backup, restore e gestione storage  

---

## Guida rapida

1. Creare un container LXC su Proxmox con:
   - Alpine Linux
   - Non privilegiato (`unprivileged: 1`)
   - Nesting abilitato (`features: nesting=1`)
2. Accedere al container e aggiornare Alpine:
   ```bash
   apk update && apk upgrade
   ```
3. Installare Docker e Docker Compose
4. Creare la cartella per Docker Compose:
   ```bash
   mkdir -p ~/portainer
   cd ~/portainer
   ```
5. Copiare il file `docker-compose.yml` e lanciare:
   ```bash
   ~/.local/bin/docker-compose up -d
   ```
6. Aprire il browser su `https://<IP_CONTAINER>:9443` per accedere a Portainer

---

## Struttura dei file

```
nextcloud-proxmox-lan/
├─ README.md
├─ docs/
│  ├─ container.md
│  ├─ docker.md
│  └─ nextcloud.md
├─ lxc/
├─ compose/
└─ scripts/
```

---

## Obiettivo del progetto

- Creare un ambiente Nextcloud **completamente in LAN**, senza esposizione diretta a Internet  
- Fornire documentazione chiara e completa con esempi pronti  
- Rendere semplice l’aggiunta di altri container Docker gestiti via Portainer
