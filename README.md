# 🐧 Curs Linux 

> **Audiență țintă:** Utilizatori cu cunoștințe de bază Linux (navigare în terminal, comenzi simple) care doresc să avanseze spre administrare de sistem și automatizare.
> **Durată estimată:** ~40–50 ore de studiu + practică

---

## 📚 Structura Cursului

| Modul | Titlu | Nivel | Ore est. |
|-------|-------|-------|----------|
| 1 | Recapitulare & Fundamente Solide | Mediu | 3h |
| 2 | Sistemul de Fișiere & Permisiuni Avansate | Mediu | 4h |
| 3 | Gestionarea Proceselor & Resurselor | Mediu | 4h |
| 4 | Rețelistică pe Linux | Mediu–Avansat | 5h |
| 5 | Gestionarea Utilizatorilor & Grupurilor | Mediu | 3h |
| 6 | Managementul Pachetelor & Compilare din Sursă | Mediu | 3h |
| 7 | **Bash Scripting** | Mediu–Avansat | 10h |
| 8 | Servicii & Systemd | Avansat | 4h |
| 9 | Securitate pe Linux | Avansat | 5h |
| 10 | Administrare Avansată & DevOps Intro | Avansat | 5h |

---

## 📦 Modul 1 — Recapitulare & Fundamente Solide

### Obiective
- Consolidarea cunoștințelor de bază
- Înțelegerea filozofiei Unix

### Conținut
1. **Structura sistemului Linux** — kernel, shell, userspace
2. **Comenzi esențiale recapitulate** — `ls`, `cd`, `cp`, `mv`, `rm`, `find`, `grep`
3. **Redirectări & Pipes avansate** — `>`, `>>`, `2>`, `|`, `tee`, `xargs`
4. **Editoare de text** — `vim` (moduri, comenzi esențiale), `nano`, `sed` basics
5. **Variabile de mediu** — `$PATH`, `$HOME`, `.bashrc`, `.profile`, `export`
6. **Manuale & Documentație** — `man`, `info`, `--help`, `tldr`

### Exerciții practice
- Construiește un pipeline complex cu `find | grep | xargs`
- Personalizează `.bashrc` cu aliasuri și funcții simple

---

## 📦 Modul 2 — Sistemul de Fișiere & Permisiuni Avansate

### Obiective
- Înțelegerea ierarhiei FHS (Filesystem Hierarchy Standard)
- Controlul avansat al permisiunilor

### Conținut
1. **FHS — Filesystem Hierarchy Standard** — `/etc`, `/var`, `/proc`, `/sys`, `/dev`, `/tmp`
2. **Permisiuni standard** — `chmod` (octal & simbolic), `chown`, `chgrp`
3. **Permisiuni speciale** — `SUID`, `SGID`, `Sticky Bit`
4. **ACL — Access Control Lists** — `getfacl`, `setfacl`
5. **Linkuri** — hard links vs symbolic links (`ln`, `ln -s`)
6. **Montarea sistemelor de fișiere** — `mount`, `umount`, `/etc/fstab`
7. **Tipuri de sisteme de fișiere** — ext4, xfs, btrfs, tmpfs
8. **LVM — Logical Volume Manager** — PV, VG, LV; creare și redimensionare
9. **Atribute de fișier** — `chattr`, `lsattr`

### Exerciții practice
- Configurează ACL pentru un director partajat între doi utilizatori
- Creează și montează un volum LVM

---

## 📦 Modul 3 — Gestionarea Proceselor & Resurselor

### Obiective
- Monitorizarea și controlul proceselor
- Optimizarea utilizării resurselor

### Conținut
1. **Procesele în Linux** — PID, PPID, stări ale proceselor, fork/exec
2. **Vizualizare procese** — `ps`, `pstree`, `top`, `htop`, `atop`
3. **Semnale** — `kill`, `killall`, `pkill`, `SIGTERM`, `SIGKILL`, `SIGHUP`
4. **Prioritatea proceselor** — `nice`, `renice`, valori nice (-20 → 19)
5. **Jobs & Background** — `&`, `fg`, `bg`, `jobs`, `nohup`, `disown`
6. **Monitorizare resurse** — `free`, `vmstat`, `iostat`, `sar`, `lsof`
7. **Limite de resurse** — `ulimit`, `/etc/security/limits.conf`
8. **cgroups** — introducere, limitare CPU/memorie
9. **Planificarea task-urilor** — `cron`, `crontab`, `at`, `anacron`

### Exerciții practice
- Monitorizează un server sub load cu `htop` + `iostat`
- Configurează un job cron complex cu logging

---

## 📦 Modul 4 — Rețelistică pe Linux

### Obiective
- Configurarea și diagnosticarea rețelei
- Securizarea conexiunilor

### Conținut
1. **Modelul OSI & TCP/IP recap**
2. **Configurare interfețe de rețea** — `ip addr`, `ip link`, `ip route`, `nmcli`, `netplan`
3. **Diagnosticare rețea** — `ping`, `traceroute`, `mtr`, `dig`, `nslookup`, `host`
4. **Analiza traficului** — `ss`, `netstat`, `tcpdump`, `wireshark` (intro)
5. **SSH avansat** — chei SSH, `ssh-keygen`, `ssh-copy-id`, SSH config, tunneling, jump hosts
6. **SCP & RSYNC** — transfer securizat de fișiere, sincronizare
7. **Firewall cu iptables/nftables** — lanțuri, reguli, NAT, logging
8. **UFW & firewalld** — interfețe simplificate
9. **VPN basics** — OpenVPN, WireGuard introducere

### Exerciții practice
- Configurează SSH cu autentificare prin cheie și dezactivează parola
- Creează reguli de firewall pentru un server web

---

## 📦 Modul 5 — Gestionarea Utilizatorilor & Grupurilor

### Obiective
- Administrarea completă a utilizatorilor
- Configurarea autentificării

### Conținut
1. **Fișiere sistem** — `/etc/passwd`, `/etc/shadow`, `/etc/group`, `/etc/gshadow`
2. **Gestionare utilizatori** — `useradd`, `usermod`, `userdel`, `passwd`
3. **Gestionare grupuri** — `groupadd`, `groupmod`, `groupdel`, `gpasswd`
4. **sudo & su** — `/etc/sudoers`, `visudo`, politici sudo granulare
5. **PAM — Pluggable Authentication Modules** — structură, module comune
6. **Politici de parole** — `chage`, complexitate, expirare
7. **Logare & Audit** — `who`, `w`, `last`, `lastlog`, `faillock`

### Exerciții practice
- Configurează sudo cu drepturi granulare pentru un utilizator de serviciu
- Implementează o politică de parole cu PAM

---

## 📦 Modul 6 — Managementul Pachetelor & Compilare din Sursă

### Obiective
- Managementul avansat al pachetelor software
- Compilarea și instalarea manuală

### Conținut
1. **APT avansat (Debian/Ubuntu)** — `apt`, `apt-get`, `dpkg`, `apt-cache`, repository-uri, pinning
2. **YUM/DNF (RHEL/CentOS/Fedora)** — `dnf`, `rpm`, repo management, modular repos
3. **Compilare din sursă** — `./configure`, `make`, `make install`, dependențe de compilare
4. **Snap & Flatpak** — avantaje, dezavantaje, utilizare
5. **Crearea pachetelor simple** — `.deb` basics, spec file RPM basics
6. **Gestiunea bibliotecilor** — `ldconfig`, `LD_LIBRARY_PATH`, `ldd`

### Exerciții practice
- Compilează și instalează Nginx din sursă cu module custom
- Creează un pachet `.deb` simplu

---

## 📦 Modul 7 — Bash Scripting 🔧

> ⭐ **Modulul central al cursului**

### Obiective
- Scrierea de scripturi Bash profesionale și robuste
- Automatizarea taskurilor de administrare
- Debugging și best practices

---

### 7.1 — Introducere în Bash Scripting
- Ce este Bash? Diferența față de sh, dash, zsh
- Shebang line — `#!/bin/bash` vs `#!/usr/bin/env bash`
- Executarea scripturilor — `bash script.sh`, `./script.sh`, `source`
- Comentarii, stil și formatare
- `set -e`, `set -u`, `set -o pipefail` — scripturi sigure

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "Hello, Linux!"
```

---

### 7.2 — Variabile & Tipuri de Date
- Declararea variabilelor — convenții de numire
- **Variabile speciale** — `$0`, `$1`-`$9`, `$#`, `$@`, `$*`, `$?`, `$$`, `$!`
- **Variabile de mediu** — `export`, `env`, `printenv`
- **Arrays (indexate)** — declarare, acces, iterare, lungime
- **Arrays asociative (dicționare)** — `declare -A`
- **readonly & local** — scoping în funcții
- **Substituție de parametri** — `${var:-default}`, `${var:=value}`, `${var:+alt}`, `${var:?error}`
- **Manipulare string** — `${#var}`, `${var#prefix}`, `${var%suffix}`, `${var/find/replace}`

```bash
# Array indexat
fruits=("mar" "para" "cireasa")
echo "${fruits[0]}"        # mar
echo "${#fruits[@]}"       # 3

# Array asociativ
declare -A config
config["host"]="localhost"
config["port"]="8080"
echo "${config[host]}"

# Substituție de parametri
name=""
echo "${name:-Guest}"      # Guest
echo "${name:?'Numele este obligatoriu!'}"
```

---

### 7.3 — Operatori & Expresii
- **Operatori aritmetici** — `$(( ))`, `let`, `expr`, `bc` pentru float
- **Operatori de comparație (numere)** — `-eq`, `-ne`, `-lt`, `-le`, `-gt`, `-ge`
- **Operatori de comparație (string)** — `=`, `!=`, `<`, `>`, `-z`, `-n`
- **Operatori pentru fișiere** — `-f`, `-d`, `-e`, `-r`, `-w`, `-x`, `-s`, `-L`
- **Operatori logici** — `&&`, `||`, `!`
- **`[[ ]]` vs `[ ]`** — diferențe, avantaje

```bash
# Aritmetică
x=10
y=3
echo $(( x * y ))          # 30
echo $(( x % y ))          # 1
result=$(echo "scale=2; $x / $y" | bc)
echo "$result"             # 3.33

# Test fișier
if [[ -f "/etc/passwd" && -r "/etc/passwd" ]]; then
    echo "Fișierul există și este citibil"
fi
```

---

### 7.4 — Structuri de Control

#### Condiții
```bash
# if / elif / else
if [[ $USER == "root" ]]; then
    echo "Ești root!"
elif [[ $EUID -eq 0 ]]; then
    echo "Rulezi cu drepturi root"
else
    echo "Utilizator normal: $USER"
fi

# case / esac
case "$1" in
    start)   echo "Pornire serviciu..." ;;
    stop)    echo "Oprire serviciu..." ;;
    restart) echo "Restart serviciu..." ;;
    *)       echo "Usage: $0 {start|stop|restart}" ; exit 1 ;;
esac
```

#### Bucle
```bash
# for clasic
for i in {1..10}; do
    echo "Iteratia $i"
done

# for C-style
for (( i=0; i<5; i++ )); do
    echo "$i"
done

# for pe array
servers=("web01" "web02" "db01")
for server in "${servers[@]}"; do
    echo "Conectare la: $server"
    ssh "$server" "uptime"
done

# while cu contor
count=0
while [[ $count -lt 5 ]]; do
    echo "Count: $count"
    (( count++ ))
done

# while citire fișier linie cu linie
while IFS= read -r line; do
    echo "Linie: $line"
done < "/etc/hosts"

# until
until ping -c1 google.com &>/dev/null; do
    echo "Aștept conexiune..."
    sleep 5
done

# break & continue
for i in {1..20}; do
    (( i % 2 == 0 )) && continue
    [[ $i -gt 10 ]] && break
    echo "Număr impar: $i"
done
```

---

### 7.5 — Funcții
- Declararea funcțiilor — sintaxă, scoping
- **Parametri** — `$1`, `$2`, `$@`
- **Valori de retur** — `return` (exit code), `echo` (output capture)
- **Variabile locale** — `local`
- **Funcții recursive**
- **Biblioteci de funcții** — sourcing

```bash
#!/usr/bin/env bash

# Funcție cu validare
create_user() {
    local username="$1"
    local uid="${2:-}"
    
    if [[ -z "$username" ]]; then
        echo "Eroare: numele utilizatorului este obligatoriu" >&2
        return 1
    fi
    
    if id "$username" &>/dev/null; then
        echo "Utilizatorul $username există deja"
        return 0
    fi
    
    if [[ -n "$uid" ]]; then
        useradd -u "$uid" "$username"
    else
        useradd "$username"
    fi
    
    echo "Utilizatorul $username a fost creat"
    return 0
}

# Captură output din funcție
get_ip() {
    local interface="${1:-eth0}"
    ip addr show "$interface" | grep 'inet ' | awk '{print $2}' | cut -d'/' -f1
}

ip_addr=$(get_ip "lo")
echo "IP loopback: $ip_addr"
```

---

### 7.6 — Input/Output & Redirectări Avansate
- **stdin, stdout, stderr** — file descriptori 0, 1, 2
- **Redirectări** — `>`, `>>`, `<`, `2>`, `2>&1`, `&>`, `/dev/null`
- **Here Documents** — `<<EOF`
- **Here Strings** — `<<<`
- **Process Substitution** — `<(cmd)`, `>(cmd)`
- **Named Pipes (FIFOs)** — `mkfifo`
- **`read`** — citire input interactiv, timeout, silent (parole)

```bash
# Here Document
cat <<EOF > /tmp/config.txt
server=localhost
port=8080
debug=true
EOF

# Here String
grep "root" <<< "$(cat /etc/passwd)"

# Process substitution
diff <(ls /dir1) <(ls /dir2)

# Read avansat
read -r -p "Introdu numele: " -t 30 username || {
    echo "Timeout! Folosim valoarea default"
    username="guest"
}

# Citire parolă (fără echo)
read -r -s -p "Parolă: " password
echo ""  # newline după parolă

# IFS custom pentru parsing CSV
IFS=',' read -r -a fields <<< "ana,ion,maria"
echo "${fields[1]}"  # ion
```

---

### 7.7 — Procesarea Textului în Bash
- **`grep` avansat** — regex POSIX, PCRE (`-P`), `-o`, `-v`, `-c`, `-l`, `-r`
- **`sed` avansat** — substitutions, delete, insert, range, multi-line, in-place
- **`awk` avansat** — câmpuri, patterns, BEGIN/END, variabile built-in, funcții
- **`cut`, `tr`, `sort`, `uniq`, `wc`**
- **`jq`** — procesare JSON din scripturi Bash
- **`paste`, `join`, `comm`**

```bash
# grep cu regex avansat
grep -P '^\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}' /var/log/nginx/access.log

# sed - înlocuire în-place cu backup
sed -i.bak 's/old_value/new_value/g' /etc/app/config.conf

# sed - delete linii care match un pattern
sed -i '/^#/d; /^$/d' config.txt  # șterge comentarii și linii goale

# awk - raport disk usage
df -h | awk 'NR>1 {printf "%-20s %s\n", $6, $5}'

# awk - suma valorilor din coloana 3
awk '{sum += $3} END {print "Total:", sum}' data.txt

# jq - extrage valori din JSON
curl -s "https://api.example.com/data" | jq '.users[] | .name'

# Pipeline complex
ps aux | grep nginx | grep -v grep | awk '{print $2}' | xargs kill -HUP
```

---

### 7.8 — Gestionarea Erorilor & Debugging
- **Exit codes** — convenții, `$?`, `exit`
- **Trap** — `trap`, semnale, cleanup la ieșire
- **`set -e`, `set -u`, `set -x`, `set -o pipefail`**
- **Logging** — niveluri de log, timestamps, syslog (`logger`)
- **Debugging** — `bash -x`, `bash -n` (syntax check), `shellcheck`
- **Defensive scripting** — validare input, lock files

```bash
#!/usr/bin/env bash
set -euo pipefail

# ===== CONSTANTE =====
readonly SCRIPT_NAME="$(basename "$0")"
readonly LOG_FILE="/var/log/${SCRIPT_NAME%.sh}.log"
readonly LOCK_FILE="/var/run/${SCRIPT_NAME%.sh}.lock"

# ===== LOGGING =====
log() {
    local level="$1"
    shift
    printf '[%s] [%s] [%s] %s\n' \
        "$(date '+%Y-%m-%d %H:%M:%S')" \
        "$level" \
        "$SCRIPT_NAME" \
        "$*" | tee -a "$LOG_FILE"
}

log_info()  { log "INFO " "$@"; }
log_warn()  { log "WARN " "$@" >&2; }
log_error() { log "ERROR" "$@" >&2; }

# ===== CLEANUP =====
cleanup() {
    local exit_code=$?
    log_info "Cleanup în curs... (exit code: $exit_code)"
    rm -f "$LOCK_FILE"
    # Adaugă alte operații de cleanup aici
    exit "$exit_code"
}
trap cleanup EXIT INT TERM

# ===== LOCK FILE =====
if [[ -f "$LOCK_FILE" ]]; then
    log_error "Scriptul rulează deja! (PID: $(cat "$LOCK_FILE"))"
    exit 1
fi
echo $$ > "$LOCK_FILE"

# ===== VALIDARE ARGUMENTE =====
usage() {
    cat <<EOF
Usage: $SCRIPT_NAME [OPTIONS] <argument>

Options:
  -h, --help     Afișează acest mesaj
  -v, --verbose  Mod verbose
  -n, --dry-run  Simulare fără modificări
EOF
    exit "${1:-0}"
}

if [[ $# -eq 0 ]]; then
    usage 1
fi
```

---

### 7.9 — Scripturi Avansate & Patterns

#### Parsarea argumentelor cu `getopts`
```bash
verbose=false
dry_run=false
output_file=""

while getopts ":vno:h" opt; do
    case "$opt" in
        v) verbose=true ;;
        n) dry_run=true ;;
        o) output_file="$OPTARG" ;;
        h) usage 0 ;;
        :) echo "Eroare: -$OPTARG necesită un argument" >&2; exit 1 ;;
        \?) echo "Opțiune necunoscută: -$OPTARG" >&2; exit 1 ;;
    esac
done
shift $(( OPTIND - 1 ))
```

#### Paralelism în Bash
```bash
# Rulare paralelă cu background jobs
max_jobs=4
job_count=0

for server in "${servers[@]}"; do
    (
        ssh "$server" "apt-get update -qq"
        echo "$server: actualizat"
    ) &
    
    (( ++job_count ))
    if (( job_count >= max_jobs )); then
        wait -n 2>/dev/null || wait
        (( job_count-- ))
    fi
done
wait
echo "Toate serverele au fost actualizate"

# GNU Parallel (dacă este instalat)
printf '%s\n' "${servers[@]}" | parallel -j4 ssh {} "apt-get update -qq"
```

#### Configurare & Profil pattern
```bash
# config.sh — fișier de configurare
declare -A CONFIG=(
    [db_host]="localhost"
    [db_port]="5432"
    [db_name]="myapp"
    [log_level]="INFO"
)

# Suprascrierea cu fișier extern
config_file="${HOME}/.myapp/config"
if [[ -f "$config_file" ]]; then
    # shellcheck source=/dev/null
    source "$config_file"
fi
```

---

### 7.10 — Proiecte Practice Bash

#### Proiect 1: Script de Backup Avansat
```
backup.sh
├── Backup incremental cu rsync
├── Rotire backup-uri (păstrare N copii)
├── Compresie și criptare opțională (gpg)
├── Notificare email la succes/eșec
└── Logging complet
```

#### Proiect 2: Monitor de Sistem
```
sysmon.sh
├── Verificare CPU, memorie, disk (threshold-uri configurabile)
├── Verificare procese critice (restart automat)
├── Verificare conectivitate (ping, port check)
├── Generare raport HTML
└── Alertare prin email/webhook
```

#### Proiect 3: Deploy Script
```
deploy.sh
├── Pull din Git repository
├── Build aplicație
├── Rulare teste automate
├── Backup versiune curentă
├── Deploy cu zero-downtime (blue-green sau rolling)
└── Rollback automat la eșec
```

#### Proiect 4: User Provisioning
```
provision_users.sh
├── Citire utilizatori din CSV/JSON
├── Creare conturi cu parole temporare
├── Configurare directoare home și permisiuni
├── Trimitere credențiale prin email
└── Audit log al modificărilor
```

### Exerciții practice Modul 7
1. Scrie un script care monitorizează un director și notifică la apariția de fișiere noi
2. Automatizează rotația certificatelor SSL cu Let's Encrypt
3. Creează un script de deployment cu rollback automat
4. Implementează un mini-framework de testing pentru scripturi Bash

---

## 📦 Modul 8 — Servicii & Systemd

### Obiective
- Administrarea completă a serviciilor
- Crearea de unit-uri personalizate

### Conținut
1. **Systemd arhitectură** — units, targets, dependencies
2. **Gestionarea serviciilor** — `systemctl start/stop/restart/reload/enable/disable/status`
3. **Unit files** — `[Unit]`, `[Service]`, `[Install]` sections
4. **Tipuri de service** — `simple`, `forking`, `oneshot`, `notify`, `dbus`
5. **Timers** — înlocuitor modern pentru cron
6. **Journald** — `journalctl`, filtrare, persistență, dimensiune
7. **Targets** — runlevels moderne, schimbarea target-ului
8. **Socket activation & Path units**
9. **Crearea unui serviciu custom** — exemplu aplicație Python/Node.js

### Exerciții practice
- Creează un serviciu systemd pentru o aplicație web custom
- Înlocuiește un cron job cu un systemd timer

---

## 📦 Modul 9 — Securitate pe Linux

### Obiective
- Hardening sistem Linux
- Detecția și răspunsul la incidente

### Conținut
1. **Principiul privilegiului minim**
2. **SSH Hardening** — dezactivare root login, schimbare port, fail2ban, 2FA
3. **Firewall avansat** — iptables/nftables, politici stricte, logging
4. **SELinux / AppArmor** — moduri, politici, troubleshooting
5. **Auditare sistem** — `auditd`, reguli audit, analiza log-urilor
6. **Scanare vulnerabilități** — `lynis`, `rkhunter`, `chkrootkit`, `nmap`
7. **Criptare** — LUKS (disk encryption), GPG (fișiere), SSL/TLS
8. **Umask, SUID/SGID audit**
9. **Kernel hardening** — `sysctl`, parametri de securitate

### Exerciții practice
- Hardening complet al unui server Ubuntu fresh install
- Configurare SELinux pentru o aplicație web

---

## 📦 Modul 10 — Administrare Avansată & DevOps Intro

### Obiective
- Administrarea la scară largă
- Introducere în tooling-ul modern DevOps

### Conținut
1. **Performanță & Tuning** — kernel parameters, I/O schedulers, network tuning
2. **Troubleshooting metodic** — USE Method, RED Method, instrumente de diagnosticare
3. **Containere — Docker basics** — concepte, comenzi esențiale, Dockerfile
4. **Ansible basics** — playbooks, inventory, roluri, ad-hoc commands
5. **Git pentru sysadmin** — Git flow, hooks, version control pentru config (GitOps)
6. **CI/CD basics** — GitHub Actions / GitLab CI, pipeline simplu
7. **Monitoring Stack** — Prometheus + Grafana, Node Exporter, alerting
8. **Loguri centralizate** — ELK Stack / Loki + Grafana basics

### Exerciții practice
- Deploy aplicație containerizată cu Docker Compose
- Scrie un playbook Ansible care provisionează un server complet

---

## 🛠️ Resurse Recomandate

### Cărți
- **"The Linux Command Line"** — William E. Shotts Jr. *(gratuit online)*
- **"Linux Bible"** — Christopher Negus
- **"How Linux Works"** — Brian Ward
- **"The Practice of System and Network Administration"** — Limoncelli et al.
- **"Linux Shell Scripting Cookbook"** — Clif Flynt & Sarath Lakshman

### Documentație Online
- [man7.org](https://man7.org/linux/man-pages/) — Manual pages Linux
- [tldp.org](https://tldp.org) — Linux Documentation Project
- [explainshell.com](https://explainshell.com) — Explicații comenzi
- [shellcheck.net](https://www.shellcheck.net) — Linter Bash online
- [devhints.io/bash](https://devhints.io/bash) — Bash cheat sheet

### Platforme de Practică
- **OverTheWire: Bandit** — provocări Linux (wargames)
- **TryHackMe** — module Linux
- **HackTheBox** — mașini Linux
- **KodeKloud** — cursuri interactive Linux/DevOps
- **Linux Foundation Training** — cursuri oficiale

### Tooling Recomandat
| Tool | Scop |
|------|------|
| `shellcheck` | Linter pentru scripturi Bash |
| `shfmt` | Formatare automată scripturi Bash |
| `bat` | `cat` cu syntax highlighting |
| `ripgrep (rg)` | `grep` modern și rapid |
| `fd` | `find` modern |
| `tmux` | Terminal multiplexer |
| `htop` / `btop` | Monitor sistem interactiv |

---

## 📋 Proiecte Finale (Capstone)

### Opțiunea A — Server de Producție
Configurează de la zero un server LEMP (Linux, Nginx, MySQL, PHP) complet securizat:
- SSL/TLS cu Let's Encrypt
- Firewall strict
- Monitoring cu alerting
- Backup automatizat
- Toate configurate prin Bash scripts + Ansible

### Opțiunea B — Pipeline CI/CD
Construiește un pipeline complet:
- Server Git (Gitea sau GitLab CE)
- CI/CD cu GitHub Actions sau GitLab CI
- Deploy automat cu zero-downtime
- Monitoring și logging

### Opțiunea C — Infrastructure as Code
- Provision complet al unui mediu multi-server cu Ansible
- Configurare Prometheus + Grafana + Alertmanager
- Scripturi de DR (Disaster Recovery)

---

## 📊 Evaluare & Certificare

| Tip | Pondere |
|-----|---------|
| Quiz-uri per modul | 20% |
| Exerciții practice | 40% |
| Proiect final (Capstone) | 40% |

**Prag de promovare:** 70% din total

---

*Curs creat: Mai 2026 | Versiune: 1.0*
# linux-training
