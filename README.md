# Nextcloud su Proxmox (LAN)

<div align="center">

![Nextcloud](https://img.shields.io/badge/Nextcloud-0082C9?style=for-the-badge&logo=nextcloud&logoColor=white)
![Proxmox](https://img.shields.io/badge/Proxmox-E57000?style=for-the-badge&logo=proxmox&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Alpine Linux](https://img.shields.io/badge/Alpine_Linux-0D597F?style=for-the-badge&logo=alpine-linux&logoColor=white)

**Una soluzione completa e sicura per il tuo cloud personale in LAN**

[📖 Documentazione](#documentazione) • [🚀 Quick Start](#quick-start) • [✨ Caratteristiche](#caratteristiche) • [🤝 Contribuire](#contribuire)

</div>

---

## 📋 Panoramica

Questa repository fornisce una guida completa e dettagliata per implementare un server **Nextcloud** professionale nella tua rete locale (LAN), utilizzando le migliori pratiche di containerizzazione e orchestrazione.

### 🏗️ Stack Tecnologico

- **Proxmox VE** - Piattaforma di virtualizzazione enterprise
- **LXC Container** (non privilegiato) - Massima sicurezza e isolamento
- **Alpine Linux** - Sistema operativo leggero e sicuro
- **Docker + Docker Compose** - Orchestrazione container
- **Portainer** - Interfaccia web per gestione container
- **Traefik** - Reverse proxy e load balancer
- **MariaDB** - Database performante e affidabile

---

## ✨ Caratteristiche

### 🔒 Sicurezza
- ✅ Container LXC non privilegiato
- ✅ Isolamento completo dei servizi
- ✅ Rete locale senza esposizione Internet
- ✅ HTTPS con certificati self-signed o Let's Encrypt (LAN)

### 🎯 Funzionalità
- ☁️ **Nextcloud** - Cloud storage completo con sincronizzazione file
- 📝 **Collabora Online** - Editor online per documenti Office
- 🔍 **ElasticSearch** - Ricerca full-text nei documenti
- 🛡️ **Pi-hole** - DNS filtering e ad-blocking
- 🔄 **Traefik** - Reverse proxy automatico con dashboard

### 🚀 Vantaggi
- 📦 Configurazione modulare e scalabile
- 🔧 Setup semplificato con script automatizzati
- 📚 Documentazione completa passo-passo
- 💾 Script di backup e restore inclusi
- 🎨 Gestione visuale tramite Portainer

---

## 📖 Documentazione

La documentazione è organizzata in moduli progressivi per una configurazione guidata:

| Step | Componente | Descrizione |
|------|-----------|-------------|
| 1️⃣ | [**Container LXC**](docs/01-creazione-container.md) | Creazione e configurazione del container Proxmox con Alpine Linux |
| 2️⃣ | [**Docker + Portainer**](docs/02-installazione-docker.md) | Installazione ambiente Docker e interfaccia di gestione |
| 3️⃣ | [**Pi-hole**](docs/03-installazione-pihole.md) | Setup DNS filtering per la rete locale |
| 4️⃣ | [**Traefik**](docs/04-installazione-traefik.md) | Configurazione reverse proxy con routing automatico |
| 5️⃣ | [**Nextcloud + MariaDB**](docs/05-installazione-nextcloud.md) | Installazione cloud storage con database |
| 6️⃣ | [**Collabora Online**](docs/06-installazione-collabora.md) | Integrazione editor documenti Office |
| 7️⃣ | [**ElasticSearch**](docs/07-installazione-elasticsearch.md) | Motore di ricerca full-text per i contenuti |

---

## 🚀 Quick Start

### Prerequisiti

- Server Proxmox VE (versione 7.0+)
- Almeno 4GB RAM e 20GB storage disponibili
- Rete LAN configurata
- Conoscenze base di Linux e Docker

### Installazione Rapida

```bash
# 1. Clona la repository
git clone https://github.com/tuousername/nextcloud-proxmox-lan.git
cd nextcloud-proxmox-lan

# 2. Segui la documentazione nell'ordine
# Inizia da docs/01-creazione-container.md

# 3. Usa i file di esempio forniti
# - lxc/*.conf → Configurazioni LXC
# - compose/*.yml → Stack Docker Compose
# - scripts/*.sh → Script di automazione
```

---

## 📁 Struttura Repository

```
nextcloud-proxmox-lan/
├── 📄 README.md                          # Questo file
├── 📂 docs/                              # Documentazione completa
│   ├── 01-creazione-container.md
│   ├── 02-installazione-docker.md
│   ├── 03-installazione-pihole.md
│   ├── 04-installazione-traefik.md
│   ├── 05-installazione-nextcloud.md
│   ├── 06-installazione-collabora.md
│   └── 07-installazione-elasticsearch.md
├── 📂 lxc/                               # Configurazioni LXC
│   └── nextcloud-server.conf
├── 📂 compose/                           # Docker Compose files
│   ├── nextcloud.yml
│   ├── traefik.yml
│   ├── collabora.yml
│   │── pihole.yml
│   │── portainer.yml
│   └── elasticsearch.yml
└── 📂 scripts/                           # Script di utilità

```

---

## 🎯 Obiettivi del Progetto

### Principali
- 🏠 Creare un ambiente Nextcloud **completamente in LAN**, senza esposizione diretta a Internet
- 📖 Fornire documentazione chiara, completa e accessibile anche a utenti intermedi
- 🔧 Offrire esempi pronti all'uso e facilmente personalizzabili
- 🔄 Permettere l'espansione semplice con altri servizi Docker

### Casi d'Uso Ideali
- 🏢 Piccole aziende che necessitano di cloud privato
- 🏠 Utenti avanzati per home server
- 🎓 Ambienti educativi e laboratori
- 👨‍💻 Sviluppatori per ambienti di test

---

## 🤝 Contribuire

I contributi sono benvenuti! Se vuoi migliorare la documentazione o aggiungere nuove funzionalità:

1. 🍴 Fai un fork del progetto
2. 🌿 Crea un branch per la tua feature (`git checkout -b feature/NuovaFunzionalita`)
3. 💾 Committa le modifiche (`git commit -m 'Aggiunge NuovaFunzionalita'`)
4. 📤 Pusha sul branch (`git push origin feature/NuovaFunzionalita`)
5. 🔄 Apri una Pull Request

---

## 📝 Licenza

Questo progetto è distribuito sotto licenza MIT. Vedi il file `LICENSE` per maggiori dettagli.

---

## 💡 Supporto

- 📧 Apri una [Issue](../../issues) per bug o richieste
- 💬 Consulta le [Discussions](../../discussions) per domande generali
- ⭐ Lascia una stella se il progetto ti è utile!

---

## 🙏 Ringraziamenti

Progetti e risorse che hanno ispirato questo lavoro:

- [Nextcloud](https://nextcloud.com/) - La piattaforma cloud open source
- [Proxmox VE](https://www.proxmox.com/) - Virtualizzazione enterprise
- [Traefik](https://traefik.io/) - Reverse proxy moderno
- [Portainer](https://www.portainer.io/) - Gestione container semplificata

---

<div align="center">

**Realizzato con ❤️ per la community self-hosting**

[⬆ Torna su](#nextcloud-su-proxmox-lan)

</div>