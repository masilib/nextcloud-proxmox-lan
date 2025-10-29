# Installazione Docker su Alpine Linux (LXC Proxmox)

Questo container LXC verrà utilizzato per ospitare Nextcloud in container Docker.

---

## Installazione Docker

```bash
apk add docker docker-cli containerd
```

---

## Abilitare Docker all’avvio

```bash
rc-update add docker boot
service docker start
```

---

## Verifica installazione

```bash
docker version
docker info
```

Se mostra informazioni su client/server → ✅ installazione corretta.

---

## Aggiunta utente al gruppo Docker *(consigliato)*

```bash
addgroup <utente> docker
```

> Se non hai utenti creati → di default si lavora da `root`  
> Puoi creare un utente se vuoi maggiore sicurezza

---

## Test veloce

Esegui un container di prova:

```bash
docker run hello-world
```

Se ricevi un messaggio di benvenuto → tutto ok ✅
