# Configurazione Container LXC per Nextcloud

Questa guida descrive la configurazione di un container LXC su Proxmox per ospitare Nextcloud in una rete locale (LAN).

---

## Parametri del Container

| Parametro | Valore consigliato |
|----------|-------------------|
| Nome | `nextcloud-server` |
| VMTYPE | Container |
| Privilegi | ✅ Non privilegiato |
| OS | Alpine Linux |
| CPU | 2 core |
| RAM | 4 GB |
| Storage | 32 GB (o più) |
| Firewall | ❌ Disattivato |
| Features | `nesting=1` |

---

## Abilitazione Nesting (obbligatorio per Docker)

Da shell di Proxmox (host):

```bash
pct set <ID> -features nesting=1
```

> 🔁 Sostituire `<ID>` con l’ID reale del container, es: `pct set 121 -features nesting=1`

---

## Avvio del Container

```bash
pct start <ID>
pct console <ID>
```

---

## Aggiornamento dei pacchetti

Dentro il container:

```bash
apk update && apk upgrade
```

---

## Installazione utilità base

```bash
apk add nano vim bash curl wget htop
```

✅ Pronto per installare Docker

➡️ Esempio file LXC pronto da usare:
[lxc/nextcloud-server.conf](../lxc/nextcloud-server.conf)

```ini
# Configurazione container LXC...
features: nesting=1
 unprivileged: 1
```


