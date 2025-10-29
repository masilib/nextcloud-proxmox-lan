# Installazione Docker + Portainer su Alpine Linux (LXC Proxmox)

Questa guida spiega come installare Docker su Alpine Linux e creare un container Portainer + Portainer Agent usando Docker Compose.

---

## 1️⃣ Aggiornare Alpine e installare utilità di base 

```bash
apk update && apk upgrade
apk add bash curl vim nano git docker docker-cli containerd py3-pip
```

---

## 2️⃣ Abilitare Docker all'avvio e avviare il servizio

```bash
rc-update add docker boot
service docker start
```

Verifica:

```bash
docker version
docker info
```

> Se mostra informazioni su client/server → ✅ installazione corretta.

---

## 3️⃣ Installare Docker Compose

```bash
apk add docker-compose
```

Alpine non ha sempre Docker Compose aggiornato, eventualmente quindi installiamo via `pip`:

```bash
python3 -m pip install --user docker-compose
```

Controlla versione:

```bash
docker-compose --version
```

---

## 4️⃣ Creare la cartella per Docker Compose

```bash
mkdir -p portainer
cd portainer
```

---

## 5️⃣ Creare il file `docker-compose.yml`

```bash
nano docker-compose.yml
```

Incolla dentro:

```yaml
version: "3"

services:
  portainer:
    image: portainer/portainer-ce
    container_name: portainer
    restart: always
    ports:
      - "8000:8000"
      - "9443:9443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - portainer_data:/data

  portainer-agent:
    image: portainer/agent:latest
    container_name: portainer-agent
    restart: always
    ports:
      - "9001:9001"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/docker/volumes:/var/lib/docker/volumes

volumes:
  portainer_data:
```

---

## 6️⃣ Avviare i container

```bash
docker-compose up -d
```

> Questo comando creerà i container Portainer e Portainer Agent in background.

---

## 7️⃣ Verifica

Apri il browser su:

```
https://<IP_CONTAINER>:9443
```

- Login iniziale → crea utente admin  
- Dovresti vedere sia Portainer che l’agent collegato

---

## 8️⃣ Note

- Portainer Agent serve a gestire più host Docker in remoto (utile se aggiungi altri container o host in futuro).  
- I dati di Portainer sono persistenti grazie al volume `portainer_data`.  
- Puoi arrestare o rimuovere i container in qualsiasi momento:

```bash
docker-compose down
```

---

✅ Ora Docker + Portainer sono pronti all’uso su Alpine Linux.
