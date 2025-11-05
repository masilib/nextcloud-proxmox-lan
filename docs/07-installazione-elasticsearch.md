# ElasticSearch per Nextcloud

In questo capitolo configuriamo **ElasticSearch**.

È il "motore di ricerca", pensa a lui come a un "Google" privato per i dati della tua azienda.
E' il componente principale per abilitare la ricerca full-text su Nextcloud. Tuttavia, da sola non è sufficiente: è necessario installare anche un'app "provider" per l'estrazione del contenuto e un'app "platform" per l'indicizzazione.

---

## 1. Prerequisiti

- **Full text search**:  Questa è l'app "motore" di base.  
- **Full text search - Elasticsearch**: Questo è il "connettore" specifico. 


Dopo averle installate, assicurati che siano entrambe attivate.

---

## 2. Scarica e attiva i moduli

- Vai nelle impostazioni di Amministrazione di Nextcloud: Impostazioni > Applicazioni > File e cerca "Full text search".
- Scarica Full text search
- Scarica Full text search - Elasticsearch Platform
- Scarica Full text search - Files : è un'estensione dell'app Full Text Search che permette di indicizzare il contenuto dei file degli utenti. È necessaria per abilitare la ricerca all'interno dei documenti

---

## 3 Creare la cartella per Docker elasticsearch
```bash
mkdir -p elasticsearch
cd elasticsearch
```

---

## 4. Creare il file `docker-compose.yml`

➡️ Esempio file docker-compose.yml pronto da usare:
[compose/elasticsearch-docker-compose](../compose/elasticsearch-docker-compose.yml)

```bash
nano docker-compose.yml
```

Incolla dentro:
```yaml
version: "3.8"

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
      - ES_JAVA_OPTS=-Xms512m -Xmx512m
    ports:
      - "9200:9200"
    networks:
      - lanufficio
    dns:
      - 172.23.0.100
    volumes:
      - es_data:/usr/share/elasticsearch/data

volumes:
  es_data:

networks:
  lanufficio:
    external: true
```

---

## 5. Avviare i container

```bash
docker-compose up -d
```

---

## 6. Verifica

Apri il browser su:

```
https://<IP_CONTAINER>:9200
```
- Dovresti vedere lo schema del tipo { "name", "cluster_name", ..... }

---

## 7. Configura la ricerca

- Vai nelle impostazioni di Amministrazione di Nextcloud: Impostazioni di amministrazione 
- Selezionare il modulo Full text search (Ricerca del testo integrale in italiano)
- In 'Generale' selezionare come piattaforma di ricerca 'Elasticsearch'

---

## 8. Comandi utili per l'indicizzazione

| Funzione | Comando |
|----------|-------------------|
| Avvia indicizzazione completa | occ fulltextsearch:index |
| Stato dei documenti | occ fulltextsearch:document:status |
| Verifica configurazione | occ fulltextsearch:check |
| Test ricerca | occ fulltextsearch:test "testo da cercare" |
| Resetta tutto | occ fulltextsearch:reset |

---

```bash
docker exec -u www-data nextcloud php occ fulltextsearch:check
```

---

## 9. Miglioriamo la ricerca

- Analyzer Italiano : migliora ricerca di testi in italiano (stemming: cane → trova anche cani)
- OCR PDF (se vuoi indicizzare PDF scannerizzati) : permette di cercare nel contenuto delle immagini
- Avviare la prima indicizzazione completa
- Live indexing


- **Dentro al container elasticsearch**

```bash
docker exec -it elasticsearch bin/elasticsearch-plugin install analysis-italian
docker restart elasticsearch
docker exec -u www-data nextcloud php occ fulltextsearch:configure '{"analyzer_tokenizer": "italian"}'

```

- **Abilitiamo OCR dei PDF (Opzionale ma Consigliato)**
Installazione Tesseract OCR (lingua italiana):

```bash
docker exec nextcloud apt update
docker exec nextcloud apt install -y tesseract-ocr tesseract-ocr-ita
```

Abilitiamo OCR nei file scannerizzati:

```bash
docker exec -u www-data nextcloud php occ files_fulltextsearch:configure '{"pdf_ocr": true}'
```

- **Indicizzazione iniziale (deve essere fatta almeno una volta!))**
```bash
docker exec -u www-data nextcloud php occ fulltextsearch:index
```
- **Live indexing**
Per evitare che gli indici restino fermi, aggiungiamo anche la live indexing:

```bash
docker exec -u www-data nextcloud php occ fulltextsearch:live
```

---

## 10. Settare un job automatico graduale per evitare carico CPU
Mettiamo in sesto un job che indicizza pian piano, senza uccidere la CPU del tuo server.
L’idea è indicizzare un piccolo numero di documenti per volta ogni tot minuti.

- **Impostiamo batch ridotto in Nextcloud**
Tu puoi scegliere quante unità, io ti propongo 20:

```bash
docker exec -u www-data nextcloud php occ fulltextsearch:configure '{"collection_indexing_list": 20}'
```


- **Imposta (in modo permanente) nano come editor predefinito (facoltativo)**
Esegui questo comando sulla tua macchina:

```bash
echo 'export EDITOR=nano' >> ~/.bashrc
```

Poi ricarica la configurazione:

```bash
source ~/.bashrc
```

Impostazione anche per root (se usi sudo su)
```bash
echo 'export EDITOR=nano' >> /root/.bashrc
source /root/.bashrc
```

Verifica
```bash
echo $EDITOR
```

- **Creiamo un cron job in Host (Proxmox)**
Apri crontab:
```bash
crontab -e
```
Inserisci questa riga:

```bash
*/10 * * * * docker exec -u www-data nextcloud php occ fulltextsearch:index --quiet
```
Ogni 10 minuti indicizza solo i nuovi/aggiornati
--quiet evita spam di output
Salva con:
- CTRL + O → Invio per confermare
- CTRL + X per uscire


## 11. Suggerimenti
- Attivare un log dedicato per controllare se qualcosa rallenta
- Fare tuning delle risorse elasticsearch (RAM, JVM)