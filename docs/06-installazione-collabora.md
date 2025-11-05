# Collabora per Nextcloud

In questo capitolo configuriamo **Collabora**. 

Il modulo Collabora per Nextcloud (chiamato anche Collabora Online o Nextcloud Office) è l’integrazione tra Nextcloud e Collabora Online, una suite di office collaborativo basata su LibreOffice.

Serve per creare, modificare e collaborare in tempo reale su documenti di testo, fogli di calcolo e presentazioni direttamente all’interno dell’interfaccia web di Nextcloud, senza bisogno di scaricare i file.
https://collabora.local



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
version: '3.8'
services:
  collabora:
    image: collabora/code:latest
    container_name: collabora
    restart: unless-stopped
    networks:
      - lanufficio
    dns:
      - 172.23.0.100
    environment:
      - domain=ufficio\\.local
      - username=admin
      - password=laTuaPassword
      # Abilita SSL
      - extra_params=--o:ssl.enable=false --o:ssl.termination=true --o:net.frame_ancestors=ufficio.local
      - dictionaries=it_IT en_US
    cap_add: 
      - MKNOD
    labels:
      - "traefik.enable=true"
      # Router HTTPS
      - "traefik.http.routers.collabora.rule=Host(`collabora.local`)"
      - "traefik.http.routers.collabora.entrypoints=websecure"
      # - "traefik.http.routers.collabora.tls.certresolver=myresolver" per certificati tipo Let's Encrypt
      - "traefik.http.routers.collabora.tls=true"
      # Servizio
      - "traefik.http.services.collabora.loadbalancer.server.port=9980"
      - "traefik.http.services.collabora.loadbalancer.server.scheme=http"
      # Middleware per headers (Collabora richiede X-Forwarded-Proto e X-Frame-Options)
      - "traefik.http.middlewares.collabora-headers.headers.customrequestheaders.X-Forwarded-Proto=https"
      - "traefik.http.middlewares.collabora-headers.headers.customresponseheaders.X-Frame-Options=SAMEORIGIN"
      - "traefik.http.routers.collabora.middlewares=collabora-headers"
      # Network Traefik
      - "traefik.docker.network=lanufficio"
networks:
  lanufficio:
    external: true
```

## 5. Avviare Collabora

- Avviare il docker

```bash
cd collabora
docker-compose up -d
```

- Dashboard visibile su `https://collabora.local`  
- Assicurati che Traefik abbia o possa generare il certificato per collabora.local. Se usi un'autorità certificativa locale (come mkcert), genera anche il certificato per Collabora.

---

## 6. Verifiche

Collabora sarà disponibile su HTTPS all’indirizzo:

```bash
https://collabora.local
```

---

## 7. Configurazione in nextcloud

- **Installa modulo su NextCloud** : nella sezione Applicazioni, abilitare l’app “Collabora Online” oppure "Nextcloud Office" dal Nextcloud App Store.
- **Impostazioni di Amministrazione** : Andare nella sezione Ufficio e impostare URL (e porta) del server di Collabora Online (https://collabora.local).
- **Disabilitare la verifica del certificato (non sicuro)**

---

## 8. Note

Per evitare problemi con certificati installati sui browser si consiglia di aprire almeno una volta la pagina https://collabora.local
Dovrebbe comparire un messaggio di 'OK'.
Da questo momento in poi il browser può essere utilizzato per creare/modificare nuovi documenti direttamente da nextcloud. 
