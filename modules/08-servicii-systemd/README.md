# 📦 Modul 8 — Servicii & Systemd

> **Nivel:** Avansat | **Durată estimată:** 4h

[← Înapoi la cuprins](../../README.md)

## Obiective
- Administrarea completă a serviciilor
- Crearea de unit-uri personalizate

---

## 8.1 — Systemd arhitectură

Systemd e sistemul de inițializare (PID 1) al majorității distribuțiilor moderne. Gestionează pornirea/oprirea serviciilor, dependențele dintre ele, și paralelizează boot-ul cât mai mult posibil.

Totul în systemd e un **unit** — o entitate gestionabilă, cu un tip declarat prin extensie:

| Extensie | Rol |
|----------|-----|
| `.service` | un proces/daemon (ex: `nginx.service`) |
| `.timer` | programare temporală (înlocuitor cron) |
| `.socket` | activare la conexiune pe un socket (socket activation) |
| `.mount` | punct de montare filesystem |
| `.target` | grupează alte units, punct de sincronizare (ex: `multi-user.target`) |
| `.path` | declanșează la modificarea unui fișier/director |

**Dependințe** — declarate în secțiunea `[Unit]`: `Requires=` (dependință hard — dacă eșuează, eșuează și unit-ul curent), `Wants=` (dependință soft — încearcă, dar continuă chiar dacă eșuează), `After=`/`Before=` (doar ordine, NU implică dependință de pornire).

```bash
systemctl list-dependencies nginx.service     # arată arborele de dependențe
```

---

## 8.2 — Gestionarea serviciilor

```bash
systemctl start nginx           # pornește acum
systemctl stop nginx              # oprește acum
systemctl restart nginx             # oprește + pornește
systemctl reload nginx                # re-citește config fără downtime (dacă serviciul suportă)
systemctl reload-or-restart nginx       # reload dacă suportă, altfel restart complet

systemctl enable nginx                    # pornește automat la boot
systemctl disable nginx                     # nu mai pornește automat la boot
systemctl enable --now nginx                  # enable + start într-un singur pas

systemctl status nginx                          # stare curentă, ultimele linii de log, PID
systemctl is-active nginx                          # doar activ/inactiv (util în scripturi)
systemctl is-enabled nginx                           # doar pornește-la-boot da/nu

systemctl list-units --type=service --state=running     # toate serviciile active acum
systemctl list-unit-files --state=enabled                  # toate unit-urile setate să pornească la boot
```

---

## 8.3 — Unit files

Un fișier `.service` are trei secțiuni principale:

```ini
# /etc/systemd/system/myapp.service
[Unit]
Description=My Custom Application
After=network.target
Wants=network-online.target

[Service]
Type=simple
User=myapp
Group=myapp
WorkingDirectory=/opt/myapp
ExecStart=/opt/myapp/venv/bin/python /opt/myapp/app.py
ExecReload=/bin/kill -HUP $MAINPID
Restart=on-failure
RestartSec=5
Environment=APP_ENV=production
EnvironmentFile=-/etc/myapp/env

[Install]
WantedBy=multi-user.target
```

- **`[Unit]`** — metadate + dependințe (`Description`, `After`, `Requires`, `Wants`).
- **`[Service]`** — cum rulează procesul (`ExecStart`, `User`, `Restart`, variabile de mediu).
- **`[Install]`** — sub ce `target` se activează la `enable` (unde ajunge simlink-ul).

```bash
systemctl daemon-reload      # OBLIGATORIU după orice editare manuală a unui unit file, altfel systemd folosește versiunea veche cache-uită
```

---

## 8.4 — Tipuri de service

| Type | Comportament |
|------|--------------|
| `simple` (default) | procesul din `ExecStart` E procesul principal; systemd îl consideră "pornit" imediat ce e lansat |
| `forking` | procesul face fork și părintele iese; systemd așteaptă ca părintele să iasă înainte să considere serviciul "pornit" (daemoane clasice, ex: vechiul Apache) |
| `oneshot` | rulează, se termină, gata — util pentru scripturi de inițializare (`RemainAfterExit=yes` dacă vrei să apară "activ" după) |
| `notify` | procesul anunță explicit systemd (`sd_notify()`) când e gata — cel mai precis, dar necesită suport în aplicație |
| `dbus` | consideră serviciul pornit când apare pe D-Bus cu un nume specific |

Alegerea greșită de `Type` cauzează probleme clasice: cu `forking` setat greșit ca `simple`, systemd poate crede că serviciul a pornit înainte ca acesta să fie de fapt gata.

---

## 8.5 — Timers — înlocuitor modern pentru cron

Un timer e format din **două** unit-uri: `.timer` (programarea) + `.service` (ce se execută), cu același nume de bază.

```ini
# /etc/systemd/system/backup.timer
[Unit]
Description=Rulează backup.service zilnic

[Timer]
OnCalendar=daily
Persistent=true    # dacă mașina era oprită la momentul programat, rulează la următorul boot (ca anacron)

[Install]
WantedBy=timers.target
```
```ini
# /etc/systemd/system/backup.service
[Unit]
Description=Backup job

[Service]
Type=oneshot
ExecStart=/usr/local/bin/backup.sh
```
```bash
systemctl enable --now backup.timer
systemctl list-timers                 # arată toate timer-ele active + următoarea rulare
```

**Avantaje față de cron:** logging integrat în journald, `Persistent=` (catch-up automat), poți exprima dependințe (`After=`), randomizare (`RandomizedDelaySec=`), și `OnCalendar=` acceptă expresii mai flexibile (`Mon..Fri *-*-* 09:00:00`).

---

## 8.6 — Journald

```bash
journalctl                       # toate logurile, cronologic
journalctl -u nginx                # doar logurile unui unit specific
journalctl -f                        # "follow" — live, ca tail -f
journalctl --since "1 hour ago"        # filtrare temporală
journalctl --since "2026-08-01" --until "2026-08-02"
journalctl -p err                        # doar nivel err și mai grav (emerg, alert, crit, err)
journalctl -b                              # doar de la boot-ul curent
journalctl -b -1                             # boot-ul anterior
journalctl --disk-usage                        # cât spațiu ocupă logurile pe disk
```

**Persistență** — implicit, unele distro păstrează journal-ul doar în RAM (`/run/log/journal`), pierdut la reboot:
```bash
mkdir -p /var/log/journal          # existența acestui director face journald să persiste pe disc
systemctl restart systemd-journald
```

**Limitare dimensiune** — în `/etc/systemd/journald.conf`:
```ini
[Journal]
SystemMaxUse=500M
```

---

## 8.7 — Targets

Echivalentul modern al vechilor "runlevels" SysV:

| Target | Echivalent runlevel | Descriere |
|--------|----------------------|-----------|
| `poweroff.target` | 0 | oprire sistem |
| `rescue.target` | 1 | mod single-user, minimal |
| `multi-user.target` | 3 | sistem complet, fără GUI (tipic pentru servere) |
| `graphical.target` | 5 | multi-user + interfață grafică |
| `reboot.target` | 6 | restart |

```bash
systemctl get-default                      # target-ul implicit la boot
systemctl set-default multi-user.target       # setează default (util pt. servere fără GUI)
systemctl isolate rescue.target                 # comută LIVE la alt target (atenție — oprește tot ce nu aparține lui)
```

---

## 8.8 — Socket activation & Path units

**Socket activation** — systemd ascultă pe un port/socket ÎN LOCUL aplicației, și pornește aplicația doar la prima conexiune (economisește resurse, permite restart fără să pierzi conexiuni în tranzit):

```ini
# myapp.socket
[Socket]
ListenStream=8080

[Install]
WantedBy=sockets.target
```
Serviciul `myapp.service` corespunzător nu mai are nevoie de `ExecStart` care face bind pe port — systemd îi pasează socket-ul deja deschis.

**Path units** — declanșează un serviciu când un fișier/director se modifică (alternativă la polling):
```ini
# watch-config.path
[Path]
PathModified=/etc/myapp/config.yaml

[Install]
WantedBy=multi-user.target
```

---

## 8.9 — Crearea unui serviciu custom (exemplu Python)

```bash
# 1. Aplicația
mkdir -p /opt/myapp
cat > /opt/myapp/app.py <<'EOF'
import time
while True:
    print("myapp rulează...", flush=True)
    time.sleep(10)
EOF

# 2. Utilizator dedicat (niciodată root pentru aplicații custom!)
useradd -r -s /usr/sbin/nologin myapp
chown -R myapp:myapp /opt/myapp

# 3. Unit file
cat > /etc/systemd/system/myapp.service <<'EOF'
[Unit]
Description=My Python App
After=network.target

[Service]
Type=simple
User=myapp
ExecStart=/usr/bin/python3 /opt/myapp/app.py
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF

# 4. Activare
systemctl daemon-reload
systemctl enable --now myapp

# 5. Verificare
systemctl status myapp
journalctl -u myapp -f
```

---

## 🧪 Laborator practic

[👉 Laborator Modul 8](laborator.md) — construiește un serviciu systemd complet pentru o aplicație custom, cu restart automat, apoi înlocuiește un cron job cu un timer echivalent.

## Exerciții practice
- Creează un serviciu systemd pentru o aplicație web custom
- Înlocuiește un cron job cu un systemd timer
