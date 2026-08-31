# 📦 Modul 3 — Gestionarea Proceselor & Resurselor

> **Nivel:** Mediu | **Durată estimată:** 4h

[← Înapoi la cuprins](../../README.md)

## Obiective
- Monitorizarea și controlul proceselor
- Optimizarea utilizării resurselor

---

## 3.1 — Procesele în Linux

Fiecare program care rulează devine un **proces**, identificat printr-un **PID** (Process ID). Fiecare proces are un **PPID** (Parent PID) — procesul care l-a creat.

```
PID 1 (systemd/init)
 ├── PID 842 (sshd)
 │    └── PID 1204 (bash)      ← PPID = 842
 │         └── PID 1310 (vim)   ← PPID = 1204
 └── PID 900 (nginx)
```

**Modelul fork/exec** — modul standard de a lansa un proces nou pe Unix:
1. **`fork()`** — procesul curent se clonează (copil = duplicat aproape identic al părintelui, cu propriul PID).
2. **`exec()`** — procesul copil își înlocuiește imaginea de memorie cu un program nou.

Dacă un proces părinte moare înaintea copilului, copilul e "adoptat" (re-parented) de `init`/`systemd` (PID 1).

**Stări ale unui proces** (vezi coloana `STAT` din `ps aux`):

| Cod | Stare | Descriere |
|-----|-------|-----------|
| `R` | Running | rulează sau e gata de rulare (în coada de scheduling) |
| `S` | Sleeping | așteaptă un eveniment (I/O, timer) — întreruptibil |
| `D` | Uninterruptible sleep | de regulă I/O pe disk — nu poate fi omorât cu semnale normale |
| `T` | Stopped | oprit (ex: prin `Ctrl+Z` sau `SIGSTOP`) |
| `Z` | Zombie | s-a terminat, dar părintele nu i-a citit încă exit code-ul (`wait()`) |

```bash
ps -o pid,ppid,stat,cmd -p $$      # informații despre shell-ul curent
```

---

## 3.2 — Vizualizare procese

```bash
ps aux                    # toate procesele, format BSD (user, %cpu, %mem, ...)
ps -ef                     # toate procesele, format System V (cu PPID vizibil)
ps -eo pid,ppid,ni,stat,%cpu,%mem,cmd --sort=-%cpu | head    # top 10 după CPU, format custom

pstree                       # arbore de procese, vizual
pstree -p                     # cu PID-uri afișate

top                            # monitor interactiv (P=sortare CPU, M=memorie, k=kill, q=exit)
htop                            # variantă modernă, colorată, cu scroll și mouse
atop                              # ca top, dar loghează istoric — util pt. postmortem
```

**Chei utile în `top`/`htop`:** `k` = kill proces selectat, `r` = renice, `1` = detaliu per-core CPU, `F6`/`>` = schimbă coloana de sortare.

---

## 3.3 — Semnale

Un semnal e un mesaj trimis unui proces, cerându-i să reacționeze (de regulă să se oprească, dar nu întotdeauna).

| Semnal | Nr. | Efect | Poate fi ignorat/capturat? |
|--------|-----|-------|------------------------------|
| `SIGHUP` | 1 | reload config (convenție) / terminal închis | da |
| `SIGINT` | 2 | întrerupere (`Ctrl+C`) | da |
| `SIGQUIT` | 3 | oprire + core dump | da |
| `SIGKILL` | 9 | oprire **imediată, necondiționată** | ❌ nu |
| `SIGTERM` | 15 | cerere de oprire "elegantă" (default la `kill`) | da |
| `SIGSTOP` | 19 | pauză procesul | ❌ nu |
| `SIGCONT` | 18 | reia un proces oprit | da |
| `SIGUSR1`/`SIGUSR2` | 10/12 | semnale custom, definite de aplicație | da |

```bash
kill 1234                  # trimite SIGTERM (implicit) — cere oprire elegantă
kill -9 1234                 # trimite SIGKILL — oprire forțată, fără cleanup
kill -SIGHUP 1234              # ex: multe daemoane reîncarcă config-ul la SIGHUP

killall nginx                   # oprește toate procesele numite exact "nginx"
pkill -f "python.*worker.py"     # oprește procese al căror command line se potrivește cu regex-ul
```

> **Best practice:** încearcă întotdeauna `SIGTERM` întâi (dă procesului șansa să facă cleanup — să închidă fișiere, conexiuni). Folosește `SIGKILL` doar dacă procesul nu răspunde — nu poate fi ignorat, dar sare peste orice cleanup.

---

## 3.4 — Prioritatea proceselor

Valoarea **nice** determină cât de "amabil" e un proces cu ceilalți în competiția pentru CPU: variază de la **-20** (prioritate maximă) la **19** (prioritate minimă). Default: `0`.

```bash
nice -n 10 ./script-greu.sh        # pornește un proces cu prioritate redusă (nice=10)
renice -n 5 -p 1234                  # schimbă prioritatea unui proces deja pornit

ps -o pid,ni,cmd -p 1234              # verifică valoarea nice curentă
```

⚠️ Doar **root** poate seta valori nice negative (prioritate mai mare) sau poate scădea nice-ul unui proces al altui utilizator. Un utilizator normal își poate doar **crește** propriul nice (poate ceda prioritate, nu poate cere mai multă).

---

## 3.5 — Jobs & Background

```bash
./script-lung.sh &          # pornește în background, shell-ul rămâne liber
jobs                          # listează job-urile din shell-ul curent
fg %1                          # aduce job-ul 1 în foreground
bg %1                           # reia (dacă e oprit) job-ul 1 în background

# Ctrl+Z în timp ce un proces rulează în foreground → îl OPREȘTE (SIGSTOP), nu îl omoară
# apoi: bg %1 pentru a-l continua în background, sau fg %1 pentru a reveni la el
```

**`nohup` vs `disown`** — ambele previn oprirea unui proces când închizi terminalul (care trimite `SIGHUP`), dar în momente diferite:

```bash
nohup ./script-lung.sh &        # pornit imediat "imun" la SIGHUP, output redirectat în nohup.out
disown -h %1                      # un job deja pornit → îl scoate din tabela de joburi a shell-ului,
                                    # astfel încât shell-ul să nu-i mai trimită SIGHUP la ieșire
```

---

## 3.6 — Monitorizare resurse

```bash
free -h                       # memorie RAM + swap, human-readable
vmstat 1                        # statistici sistem la fiecare 1s (procese, memorie, I/O, CPU)
iostat -x 1                      # utilizare disk detaliată (necesită pachetul sysstat)
sar -u 1 5                        # utilizare CPU, 5 mostre la interval de 1s (necesită sysstat + configurare)

lsof -p 1234                        # ce fișiere/socket-uri are deschise procesul 1234
lsof -i :80                          # ce proces ascultă pe portul 80
lsof -u alice                         # toate fișierele deschise de utilizatorul alice
lsof +D /var/log                       # ce procese au fișiere deschise în /var/log (util înainte de umount)
```

---

## 3.7 — Limite de resurse

```bash
ulimit -a                    # toate limitele curente ale shell-ului
ulimit -n                     # limita curentă de file descriptors deschise
ulimit -n 4096                 # setează limita "soft" pentru sesiunea curentă
ulimit -Hn                       # afișează limita "hard" (plafonul absolut, doar root o poate crește)
```

Pentru limite **persistente**, per utilizator/grup, editează `/etc/security/limits.conf` (aplicat de PAM):

```
# /etc/security/limits.conf
# <domain>   <type>   <item>   <value>
alice         soft      nofile    4096
alice         hard      nofile    8192
@developers    soft      nproc     100
```

- **soft** = limita implicită, utilizatorul o poate crește singur până la valoarea hard.
- **hard** = plafonul absolut, doar root îl poate schimba.

---

## 3.8 — cgroups — introducere

**Control groups (cgroups)** permit limitarea și contabilizarea resurselor (CPU, memorie, I/O) pentru grupuri de procese — stau la baza containerelor (Docker, systemd services).

Pe sisteme cu systemd, cel mai simplu mod de a folosi cgroups e prin unit-uri systemd, fără să atingi manual filesystem-ul `/sys/fs/cgroup`:

```bash
# Rulează o comandă într-un scope cgroup temporar, limitat
systemd-run --scope -p CPUQuota=20% -p MemoryMax=200M ./script-greu.sh

# Pentru un serviciu permanent, în unit file (/etc/systemd/system/app.service):
# [Service]
# CPUQuota=50%
# MemoryMax=512M
# MemorySwapMax=0
```

```bash
systemctl set-property nginx.service CPUQuota=30% MemoryMax=300M    # ajustare live, fără restart
systemd-cgtop                                                          # monitorizare consum per cgroup, live
```

---

## 3.9 — Planificarea task-urilor

```bash
crontab -e                  # editează crontab-ul utilizatorului curent
crontab -l                   # listează job-urile curente
crontab -r                    # șterge tot crontab-ul (⚠️ ireversibil, fără confirmare)
```

**Sintaxă crontab:** `minut oră zi_lună lună zi_săptămână comandă`

```
# ┌───────────── minut (0-59)
# │ ┌───────────── oră (0-23)
# │ │ ┌───────────── zi a lunii (1-31)
# │ │ │ ┌───────────── lună (1-12)
# │ │ │ │ ┌───────────── zi a săptămânii (0-6, 0=duminică)
# │ │ │ │ │
  0 3 * * *   /usr/local/bin/backup.sh >> /var/log/backup.log 2>&1     # zilnic la 03:00
  */15 * * * * /usr/local/bin/healthcheck.sh                            # la fiecare 15 minute
  0 9 * * 1-5   /usr/local/bin/report.sh                                  # 09:00, luni-vineri
  @reboot         /usr/local/bin/startup-check.sh                          # la fiecare boot
```

- `/etc/cron.d/` — fișiere crontab la nivel de sistem, cu o coloană în plus pentru user.
- `at "now + 1 hour"` — rulează o comandă **o singură dată**, la un moment viitor (interactiv, `Ctrl+D` să salvezi).
- `anacron` — pentru mașini care nu rulează 24/7 (laptopuri) — rulează job-uri "restante" la următorul boot dacă mașina a fost oprită la momentul programat.

---

## 🧪 Laborator practic

[👉 Laborator Modul 3](laborator.md) — depanează un "proces scăpat de sub control", ajustează priorități și limite, și automatizează un job de monitorizare cu cron.

## Exerciții practice
- Monitorizează un server sub load cu `htop` + `iostat`
- Configurează un job cron complex cu logging
