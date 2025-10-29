# Creazione del Container su Proxmox

Guida passo passo per creare un container ottimizzato per Docker.

FASE 1 — Creazione Container LXC in Proxmox


✅ FASE 1 — Creazione Container LXC in Proxmox

📌 Incolla questo contenuto in docs/container.md
(se non esiste ancora il file → crealo tu)

Creazione del Container LXC per Nextcloud

Questa guida descrive la creazione del container LXC tramite GUI Proxmox.

Parametri del Container
Parametro	Valore
Nome	nextcloud-server
Tipo	Container non privilegiato
OS	Alpine Linux (versione stabile)
CPU	2 core
RAM	4 GB (consigliato)
Storage	32 GB (minimo consigliato)
Networking	Statico su LAN locale
Firewall	❌ Disattivato
Abilitare il Nesting

📍 Necessario per Docker dentro LXC

Da shell Proxmox (host):

pct set <ID> -features nesting=1


Sostituire <ID> con l’ID del container, es: pct set 105 -features nesting=1

Avvio del Container
pct start <ID>
pct console <ID>

Post-installazione base

Aggiornamento pacchetti:

apk update && apk upgrade


Installazione strumenti utili:

apk add nano vim bash curl wget htop