# 🚀 Guida Ottimizzazione Nextcloud - Produzione

Documento completo per configurazione avanzata Nextcloud con Docker + Traefik + Redis + MariaDB


---

## 📋 Indice

1. [Ottimizzazione Performance](#1-ottimizzazione-performance)
2. [Configurazione Email](#2-configurazione-email)
3. [Installazione App Essenziali](#3-installazione-app-essenziali)
4. [Limiti Upload e Timeout](#4-limiti-upload-e-timeout)
5. [Verifiche e Test](#5-verifiche-e-test)
6. [Manutenzione Periodica](#6-manutenzione-periodica)
7. [Troubleshooting](#7-troubleshooting)

---

## 1. Ottimizzazione Performance

### 1.1 Configurazione Redis (Cache)

Redis è un database in-memory che velocizza Nextcloud del 30-50%.

```bash
# Configura connessione Redis
docker exec -u www-data nextcloud php occ config:system:set redis host --value="redis"
docker exec -u www-data nextcloud php occ config:system:set redis port --value="6379"

# Memory caching locale (APCu)
docker exec -u www-data nextcloud php occ config:system:set memcache.local --value="\OC\Memcache\APCu"

# Memory caching distribuita (Redis)
docker exec -u www-data nextcloud php occ config:system:set memcache.distributed --value="\OC\Memcache\Redis"

# File locking con Redis (evita conflitti multi-utente)
docker exec -u www-data nextcloud php occ config:system:set memcache.locking --value="\OC\Memcache\Redis"

# Verifica configurazione Redis
docker exec -u www-data nextcloud php occ config:list system | grep -A 10 redis
```

**Benefici:**
- ✅ Caricamento pagine 30-50% più veloce
- ✅ Nessun conflitto file con utenti simultanei
- ✅ Query database ridotte del 70%

---

### 1.2 Configurazione PHP Ottimizzata

Crea file di configurazione PHP personalizzato:

```bash
cd ~/nextcloud-docker
mkdir -p config/php

cat > config/php/custom.ini << 'EOF'
# Limiti memoria e upload
memory_limit = 512M
upload_max_filesize = 10G
post_max_size = 10G

# Timeout
max_execution_time = 3600
max_input_time = 3600

# Regione telefoni
default_phone_region = IT

# OPcache (cache codice PHP compilato)
opcache.enable = 1
opcache.interned_strings_buffer = 16
opcache.max_accelerated_files = 10000
opcache.memory_consumption = 256
opcache.save_comments = 1
opcache.revalidate_freq = 60

# APCu
apc.enable_cli = 1
EOF
```

**Aggiungi al docker-compose.yml:**

```yaml
nextcloud:
  # ... configurazione esistente ...
  volumes:
    - /mnt/nextcloud-data:/var/www/html
    - ./config/php/custom.ini:/usr/local/etc/php/conf.d/zzz-custom.ini:ro  # ← AGGIUNGI
```

**Riavvia Nextcloud:**

```bash
docker compose restart nextcloud
```

**Verifica configurazione applicata:**

```bash
docker exec nextcloud php -i | grep -E "upload_max_filesize|memory_limit|opcache.enable"
```

**Benefici:**
- ✅ Upload file fino a 10 GB
- ✅ Esecuzione PHP 50-70% più veloce (OPcache)
- ✅ Nessun timeout durante operazioni lunghe

---

### 1.3 Ottimizzazione Database MariaDB

Manutenzione database per prestazioni ottimali:

```bash
# Aggiungi indici mancanti (10-100x più veloce)
docker exec -u www-data nextcloud php occ db:add-missing-indices

# Aggiungi colonne mancanti
docker exec -u www-data nextcloud php occ db:add-missing-columns

# Aggiungi chiavi primarie
docker exec -u www-data nextcloud php occ db:add-missing-primary-keys

# Converti a BIGINT (supporta miliardi di file)
docker exec -u www-data nextcloud php occ db:convert-filecache-bigint

# Ottimizza tabelle (rimuove spazi vuoti)
docker exec mariadb mariadb -u root -p${MYSQL_ROOT} nextcloud -e "OPTIMIZE TABLE oc_filecache;"
```

**Quando eseguire:**
- ✅ Subito dopo installazione
- ✅ Dopo ogni aggiornamento major di Nextcloud
- ✅ Ogni 3-6 mesi (OPTIMIZE TABLE)

---

### 1.4 Background Jobs con Cron

Passa da Ajax a Cron per esecuzione affidabile task in background:

```bash
# Abilita modalità Cron
docker exec -u www-data nextcloud php occ background:cron

# Verifica attivazione
docker exec -u www-data nextcloud php occ config:system:get backgroundjobs_mode
# Output atteso: cron
```

**Aggiungi servizio cron al docker-compose.yml:**

```yaml
services:
  # ... servizi esistenti ...

  nextcloud-cron:
    image: nextcloud:latest
    container_name: nextcloud-cron
    restart: unless-stopped
    networks:
      lanufficio:
    volumes:
      - /mnt/nextcloud-data:/var/www/html
    entrypoint: /cron.sh
    depends_on:
      - nextcloud
```

**Riavvia stack:**

```bash
docker compose up -d
```

**Verifica cron funzionante:**

```bash
# Vedi log cron
docker logs nextcloud-cron --tail 20

# Dovrebbe mostrare esecuzioni ogni 5 minuti
```

**Benefici:**
- ✅ Task eseguiti anche senza utenti collegati
- ✅ Timing preciso (ogni 5 minuti esatti)
- ✅ Anteprime generate in background
- ✅ Email notifiche inviate regolarmente

---

### 1.5 File Locking

Abilita locking per prevenire conflitti file:

```bash
# Abilita file locking
docker exec -u www-data nextcloud php occ config:system:set filelocking.enabled --value=true --type=boolean

# Imposta TTL (scadenza lock automatica dopo 1 ora)
docker exec -u www-data nextcloud php occ config:system:set filelocking.ttl --value=3600 --type=integer

# Verifica
docker exec -u www-data nextcloud php occ config:system:get filelocking.enabled
```

**Benefici:**
- ✅ Nessun file "conflitto" quando utenti collaborano
- ✅ Lock automatici scadono per evitare blocchi infiniti

---

## 2. Configurazione Email

### 2.1 Configurazione SMTP

**Provider comuni:**

| Provider | SMTP Host | Port | Security |
|----------|-----------|------|----------|
| Gmail | smtp.gmail.com | 587 | STARTTLS |
| Outlook/Hotmail | smtp-mail.outlook.com | 587 | STARTTLS |
| Office 365 | smtp.office365.com | 587 | STARTTLS |
| PEC Aruba | smtps.pec.aruba.it | 465 | SSL |

**Configurazione via comando:**

```bash
# Modalità SMTP
docker exec -u www-data nextcloud php occ config:system:set mail_smtpmode --value="smtp"

# Server SMTP (esempio Gmail, adatta al tuo provider)
docker exec -u www-data nextcloud php occ config:system:set mail_smtphost --value="smtp.gmail.com"
docker exec -u www-data nextcloud php occ config:system:set mail_smtpport --value="587"
docker exec -u www-data nextcloud php occ config:system:set mail_smtpsecure --value="tls"

# Autenticazione
docker exec -u www-data nextcloud php occ config:system:set mail_smtpauth --value=true --type=boolean
docker exec -u www-data nextcloud php occ config:system:set mail_smtpname --value="tua-email@gmail.com"
docker exec -u www-data nextcloud php occ config:system:set mail_smtppassword --value="PASSWORD_APP_GOOGLE"

# Mittente email
docker exec -u www-data nextcloud php occ config:system:set mail_from_address --value="noreply"
docker exec -u www-data nextcloud php occ config:system:set mail_domain --value="ufficio.local"
```

**⚠️ IMPORTANTE per Gmail:**

1. Attiva verifica in 2 passaggi su account Google
2. Vai su: https://myaccount.google.com/apppasswords
3. Genera "Password per le app" per Nextcloud
4. Usa QUELLA password nel comando `mail_smtppassword`

**Test invio email:**

```bash
# Imposta email admin (sostituisci con tua email)
docker exec -u www-data nextcloud php occ user:setting admin settings email "admin@ufficio.local"

# Invia email di test dalla web UI:
# Settings → Administration → Basic settings → Send email
```

---

### 2.2 Configurazione alternativa via Web UI

1. Login come admin
2. **Settings** → **Administration** → **Basic settings**
3. Sezione **Email server**:
   - Send mode: `SMTP`
   - Encryption: `STARTTLS`
   - From address: `noreply@ufficio.local`
   - Server address: `smtp.gmail.com:587`
   - Authentication: ✓ Required
   - Credentials: inserisci email e password app
4. Clicca **Send email** per testare

---


## 3. Limiti Upload e Timeout

### 3.1 Configurazione Apache Timeout

```bash
cd ~/nextcloud-docker
mkdir -p config/apache

cat > config/apache/timeout.conf << 'EOF'
Timeout 3600
ProxyTimeout 3600
EOF
```

**Aggiungi al docker-compose.yml:**

```yaml
nextcloud:
  volumes:
    # ... volumi esistenti ...
    - ./config/apache/timeout.conf:/etc/apache2/conf-available/timeout.conf:ro
```

**Attiva configurazione:**

```bash
docker compose restart nextcloud
docker exec nextcloud a2enconf timeout
docker exec nextcloud apache2ctl graceful
```

---

### 3.2 Chunk Upload (file grandi)

```bash
# Imposta chunk size 10 MB (divide file grandi in pezzi)
docker exec -u www-data nextcloud php occ config:system:set chunk_size --value=10485760

# Upload timeout 1 ora
docker exec -u www-data nextcloud php occ config:system:set uploadtimeout --value=3600 --type=integer

# Abilita bulk upload
docker exec -u www-data nextcloud php occ config:system:set bulkupload.enabled --value=true --type=boolean
```

**Benefici:**
- ✅ Upload file fino a 10 GB affidabili
- ✅ Resume automatico se connessione si interrompe
- ✅ Progress bar precisa

---

### 3.3 Verifica limiti attivi

```bash
# Verifica configurazione PHP
docker exec nextcloud php -i | grep -E "upload_max_filesize|post_max_size|max_execution_time"

# Output atteso:
# upload_max_filesize => 10G
# post_max_size => 10G
# max_execution_time => 3600
```


---