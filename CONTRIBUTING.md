# ============================================
# CONTRIBUTING.md
# ============================================

# Contribuire al progetto

Prima di tutto, grazie per l'interesse nel contribuire a questo progetto! 🎉

Questo documento fornisce le linee guida per contribuire in modo efficace.

---

## 📋 Indice

- [Come posso contribuire?](#come-posso-contribuire)
- [Segnalare bug](#segnalare-bug)
- [Proporre nuove funzionalità](#proporre-nuove-funzionalità)
- [Processo di Pull Request](#processo-di-pull-request)
- [Standard di codice](#standard-di-codice)
- [Struttura dei commit](#struttura-dei-commit)

---

## 🤝 Come posso contribuire?

Ci sono diversi modi per contribuire:

### 1. 📝 Migliorare la documentazione
- Correggere errori di battitura o grammaticali
- Aggiungere esempi pratici
- Chiarire sezioni poco chiare
- Tradurre la documentazione in altre lingue

### 2. 🐛 Segnalare bug
- Problemi nei file di configurazione
- Errori nella documentazione
- Script non funzionanti

### 3. ✨ Proporre nuove funzionalità
- Nuovi servizi Docker da integrare
- Script di automazione aggiuntivi
- Miglioramenti alla sicurezza

### 4. 🔧 Contribuire con codice
- Correggere bug
- Implementare nuove funzionalità
- Ottimizzare script esistenti

---

## 🐛 Segnalare bug

Prima di aprire una issue per un bug:

1. **Verifica** che il bug non sia già stato segnalato
2. **Assicurati** di usare l'ultima versione della documentazione
3. **Raccogli informazioni**:
   - Versione di Proxmox
   - Versione di Alpine Linux
   - Versioni dei container Docker
   - Log degli errori

### Template per segnalare un bug

```markdown
## Descrizione del problema
[Descrizione chiara e concisa del bug]

## Passi per riprodurre
1. Vai a '...'
2. Esegui '...'
3. Vedi errore

## Comportamento atteso
[Cosa ti aspettavi che succedesse]

## Comportamento attuale
[Cosa succede invece]

## Ambiente
- Proxmox VE: [versione]
- Alpine Linux: [versione]
- Docker: [versione]
- Container interessato: [nome]

## Log e screenshot
[Includi log rilevanti o screenshot]

## Note aggiuntive
[Eventuali informazioni aggiuntive]
```

---

## 💡 Proporre nuove funzionalità

Per proporre una nuova funzionalità:

1. **Apri una issue** con il tag `enhancement`
2. **Descrivi** chiaramente la funzionalità
3. **Spiega** perché sarebbe utile
4. **Proponi** un'implementazione (opzionale)

### Template per nuove funzionalità

```markdown
## Descrizione della funzionalità
[Descrizione chiara della funzionalità proposta]

## Motivazione
[Perché questa funzionalità sarebbe utile?]

## Proposta di implementazione
[Come potrebbe essere implementata? (opzionale)]

## Alternative considerate
[Hai considerato altre soluzioni?]
```

---

## 🔄 Processo di Pull Request

### Prima di iniziare

1. **Apri una issue** per discutere la modifica (per cambiamenti sostanziali)
2. **Fai un fork** del repository
3. **Crea un branch** dalla `main`:
   ```bash
   git checkout -b feature/nome-funzionalita
   # oppure
   git checkout -b fix/nome-bug
   ```

### Durante lo sviluppo

1. **Mantieni i commit atomici** (una modifica logica per commit)
2. **Testa** le tue modifiche
3. **Aggiorna** la documentazione se necessario
4. **Segui** gli standard di codice (vedi sotto)

### Inviare la Pull Request

1. **Pusha** il tuo branch:
   ```bash
   git push origin feature/nome-funzionalita
   ```

2. **Apri una Pull Request** su GitHub

3. **Descrivi** le modifiche nel template:
   ```markdown
   ## Descrizione
   [Descrizione delle modifiche]

   ## Tipo di cambiamento
   - [ ] Bug fix
   - [ ] Nuova funzionalità
   - [ ] Breaking change
   - [ ] Documentazione

   ## Checklist
   - [ ] Ho testato le modifiche
   - [ ] Ho aggiornato la documentazione
   - [ ] Ho seguito gli standard di codice
   - [ ] I miei commit hanno messaggi descrittivi

   ## Issue correlate
   Closes #[numero issue]
   ```

4. **Attendi** il review

---

## 📐 Standard di codice

### File di configurazione

- **Indentazione**: 2 spazi per YAML, 4 per altri formati
- **Naming**: usa nomi descrittivi e in minuscolo
- **Commenti**: documenta scelte non ovvie

### Script Bash

```bash
#!/bin/bash
# Descrizione dello script

set -euo pipefail  # Fail on error, undefined vars, pipe failures

# Variabili in MAIUSCOLO
BACKUP_DIR="/backup"

# Funzioni con nomi descrittivi
function backup_database() {
    local db_name="$1"
    # ...
}

# Commenta sezioni complesse
# Questo blocco gestisce...
```

### Docker Compose

```yaml
version: '3.8'

services:
  service_name:
    image: image:tag  # Usa sempre tag specifici, non 'latest'
    container_name: descriptive-name
    restart: unless-stopped
    environment:
      - VAR_NAME=value
    volumes:
      - ./data:/data  # Usa path relativi quando possibile
    networks:
      - network_name
    # Commenta scelte non standard
```

### Documentazione Markdown

- **Titoli**: Usa heading gerarchici (H1 → H2 → H3)
- **Codice**: Sempre con syntax highlighting
- **Link**: Preferisci link relativi per file interni
- **Immagini**: Salvale in `docs/images/` con nomi descrittivi

---

## 📝 Struttura dei commit

Usa commit message chiari e descrittivi seguendo questo formato:

```
<tipo>: <descrizione breve>

[corpo opzionale: spiega cosa e perché, non come]

[footer opzionale: riferimenti a issue]
```

### Tipi di commit

- `feat`: Nuova funzionalità
- `fix`: Correzione bug
- `docs`: Modifiche alla documentazione
- `style`: Formattazione, spazi, ecc.
- `refactor`: Refactoring senza cambiare funzionalità
- `test`: Aggiunta o modifica test
- `chore`: Manutenzione, aggiornamenti dipendenze

### Esempi

```bash
# Buoni
feat: aggiunge supporto per Redis
fix: corregge errore mount storage in script backup
docs: migliora sezione installazione Traefik

# Da evitare
update
fix stuff
changes
```

---

## 🧪 Testing

Prima di inviare una PR:

1. **Testa** le configurazioni su un ambiente pulito
2. **Verifica** che gli script funzionino
3. **Controlla** che la documentazione sia accurata
4. **Valida** i file YAML:
   ```bash
   docker-compose -f compose/file.yml config
   ```

---

## 📜 Licenza

Contribuendo, accetti che i tuoi contributi saranno rilasciati sotto la licenza MIT del progetto.

---

## ❓ Domande?

Se hai domande:

- 💬 Apri una [Discussion](../../discussions)
- 📧 Apri una [Issue](../../issues) con tag `question`

---

## 🙏 Grazie!

Ogni contributo, grande o piccolo, è prezioso e apprezzato!

**Happy coding!** 🚀