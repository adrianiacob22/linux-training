# 📦 Modul 7 — Bash Scripting 🔧

> **Nivel:** Mediu–Avansat | **Durată estimată:** 10h
> ⭐ **Modulul central al cursului**

[← Înapoi la cuprins](../../README.md)

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

### 🧪 Laborator practic

[👉 Laborator Modul 7](laborator.md) — construiește incremental un script real de monitorizare (`syscheck.sh`), de la schelet până la o versiune completă cu logging, `trap`, `getopts`.

### Exerciții practice Modul 7
1. Scrie un script care monitorizează un director și notifică la apariția de fișiere noi
2. Automatizează rotația certificatelor SSL cu Let's Encrypt
3. Creează un script de deployment cu rollback automat
4. Implementează un mini-framework de testing pentru scripturi Bash
