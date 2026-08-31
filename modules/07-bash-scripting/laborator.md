# 🧪 Laborator — Modul 7: Bash Scripting

[← Înapoi la Modul 7](README.md) · [Cuprins curs](../../README.md)

## Scenariu

Ești rugat să construiești, de la zero și incremental, un script de monitorizare de sistem — `syscheck.sh` — pe care echipa de operațiuni îl va rula prin cron. Fiecare task adaugă o funcționalitate nouă peste ce ai construit deja, exact cum ar evolua un script real în producție. La final vei avea un instrument complet, robust, cu logging, validare de argumente și cod de ieșire corect.

## Cerințe prealabile
- Bash ≥ 4 (pentru array-uri asociative)
- `shellcheck` instalat, ideal (`apt install shellcheck` / `brew install shellcheck`) — folosește-l după fiecare task

Creează un director de lucru:
```bash
mkdir -p ~/lab-bash && cd ~/lab-bash
touch syscheck.sh && chmod +x syscheck.sh
```

---

## Task 1 — Scheletul scriptului

Pornește cu un shebang portabil și modul "sigur" activat, care oprește scriptul la prima eroare, la variabile nedefinite, și propagă erorile din pipeline-uri.

<details>
<summary>✅ Soluție</summary>

```bash
#!/usr/bin/env bash
set -euo pipefail

echo "syscheck.sh — pornit"
```

Testează imediat cu `./syscheck.sh`. Rulează `shellcheck syscheck.sh` — trebuie să fie curat de la primul task.
</details>

---

## Task 2 — Verificare spațiu disc, ca funcție

Scrie o funcție `check_disk` care primește ca parametru un punct de montare (default `/` dacă nu se dă niciun argument) și afișează procentul de utilizare, extras din `df`.

<details>
<summary>💡 Hint</summary>

`df -h /` are pe a doua linie coloana `Use%`. Combină `df` cu `awk 'NR==2 {print $5}'` și `${1:-/}` pentru default.
</details>

<details>
<summary>✅ Soluție</summary>

```bash
check_disk() {
    local mount_point="${1:-/}"
    local usage
    usage=$(df -h "$mount_point" | awk 'NR==2 {print $5}' | tr -d '%')
    echo "Disk ($mount_point): ${usage}%"
}

check_disk
check_disk /tmp
```
</details>

---

## Task 3 — Praguri și cod de ieșire

Transformă `check_disk` astfel încât să primească și un prag (threshold, default 80), și să **returneze** (nu doar să afișeze) un cod: `0` = OK, `1` = peste prag. Scrie un `if` care folosește acest cod pentru a afișa `[OK]` sau `[ALERTĂ]`.

<details>
<summary>✅ Soluție</summary>

```bash
check_disk() {
    local mount_point="${1:-/}"
    local threshold="${2:-80}"
    local usage
    usage=$(df -h "$mount_point" | awk 'NR==2 {print $5}' | tr -d '%')

    if (( usage >= threshold )); then
        echo "Disk ($mount_point): ${usage}% [ALERTĂ, prag ${threshold}%]"
        return 1
    else
        echo "Disk ($mount_point): ${usage}% [OK]"
        return 0
    fi
}

if check_disk / 80; then
    echo "Totul e ok pe /"
else
    echo "Trebuie investigat!" >&2
fi
```

> Observă: `set -e` ar opri scriptul dacă `check_disk` returnează 1 direct într-un context care nu e testat de `if`. De-asta îl apelăm mereu în interiorul unui `if`/`||`.
</details>

---

## Task 4 — Verifică mai multe puncte de montare dintr-un array

Extinde scriptul: definește un array cu punctele de montare de verificat (`/`, `/tmp`, `/var` — sau ce ai disponibil), iterează cu `for`, și numără câte alerte au apărut.

<details>
<summary>✅ Soluție</summary>

```bash
mount_points=(/ /tmp)
alert_count=0

for mp in "${mount_points[@]}"; do
    if ! check_disk "$mp" 80; then
        (( alert_count++ ))
    fi
done

echo "Total alerte disk: $alert_count"
```
</details>

---

## Task 5 — Verificare proces critic + procesare text

Adaugă o funcție `check_process` care primește un nume de proces și verifică (prin `pgrep`, sau `ps aux | grep` dacă nu ai `pgrep`) dacă rulează cel puțin o instanță. Testeaz-o pe un proces care sigur rulează (`sshd`, `bash`, sau chiar `sysgroup` fictiv pentru a vedea cazul de eșec).

<details>
<summary>💡 Hint</summary>

`pgrep -x nume` e mai curat decât `ps aux | grep`, dar exersează și varianta cu pipeline dacă vrei să practici `grep -v grep`.
</details>

<details>
<summary>✅ Soluție</summary>

```bash
check_process() {
    local process_name="$1"
    if pgrep -x "$process_name" >/dev/null 2>&1; then
        echo "Proces '$process_name': [RULEAZĂ]"
        return 0
    else
        echo "Proces '$process_name': [OPRIT]"
        return 1
    fi
}

check_process "bash" || true
check_process "proces-inexistent-xyz" || true
```

Varianta fără `pgrep` (portabilă, dar mai fragilă):
```bash
check_process_fallback() {
    local process_name="$1"
    if ps aux | grep -v grep | grep -q "$process_name"; then
        echo "Proces '$process_name': [RULEAZĂ]"
        return 0
    fi
    echo "Proces '$process_name': [OPRIT]"
    return 1
}
```
</details>

---

## Task 6 — Logging structurat

Adaugă funcții de logging (`log_info`, `log_warn`, `log_error`) care scriu într-un fișier de log **și** pe ecran, cu timestamp, folosind `tee`. Erorile trebuie să meargă pe `stderr`.

<details>
<summary>✅ Soluție</summary>

```bash
readonly LOG_FILE="${LOG_FILE:-/tmp/syscheck.log}"

log() {
    local level="$1"; shift
    printf '[%s] [%s] %s\n' "$(date '+%Y-%m-%d %H:%M:%S')" "$level" "$*" | tee -a "$LOG_FILE"
}

log_info()  { log "INFO " "$@"; }
log_warn()  { log "WARN " "$@" >&2; }
log_error() { log "ERROR" "$@" >&2; }

log_info "Verificare pornită"
```
</details>

---

## Task 7 — Cleanup garantat cu `trap`

Scriptul creează un lock file la pornire (ca să nu ruleze de două ori simultan) și trebuie să-l șteargă la ieșire, **indiferent** dacă scriptul se termină normal, e întrerupt (`Ctrl+C`) sau eșuează.

<details>
<summary>✅ Soluție</summary>

```bash
readonly LOCK_FILE="/tmp/syscheck.lock"

cleanup() {
    local exit_code=$?
    rm -f "$LOCK_FILE"
    log_info "Cleanup efectuat (exit code: $exit_code)"
    exit "$exit_code"
}
trap cleanup EXIT INT TERM

if [[ -f "$LOCK_FILE" ]]; then
    log_error "Scriptul rulează deja (PID $(cat "$LOCK_FILE"))"
    exit 1
fi
echo $$ > "$LOCK_FILE"
```

Testează: rulează scriptul în background (`./syscheck.sh &`), apoi rulează-l din nou — al doilea lansare trebuie să refuze să pornească. Testează și `Ctrl+C` în mijlocul execuției — lock file-ul trebuie să dispară oricum.
</details>

---

## Task 8 — Argumente din linia de comandă cu `getopts`

Adaugă suport pentru flag-uri:
- `-t <prag>` — threshold custom pentru disk (default 80)
- `-v` — mod verbose (afișează și punctele de montare OK, nu doar alertele)
- `-h` — afișează help și iese

<details>
<summary>✅ Soluție</summary>

```bash
usage() {
    cat <<EOF
Usage: $(basename "$0") [-t threshold] [-v] [-h]

  -t N   Prag alertă disk, procent (default: 80)
  -v     Mod verbose
  -h     Acest mesaj
EOF
    exit "${1:-0}"
}

threshold=80
verbose=false

while getopts ":t:vh" opt; do
    case "$opt" in
        t) threshold="$OPTARG" ;;
        v) verbose=true ;;
        h) usage 0 ;;
        :) echo "Eroare: -$OPTARG necesită un argument" >&2; usage 1 ;;
        \?) echo "Opțiune necunoscută: -$OPTARG" >&2; usage 1 ;;
    esac
done
shift $(( OPTIND - 1 ))
```
</details>

---

## Task 9 — Asamblare finală

Pune totul cap la cap: scheletul, logging-ul, lock file-ul, `getopts`, verificarea de disk pe un array de mount points (cu threshold-ul din `-t`), și verificarea unui proces critic. La final, scriptul trebuie să iasă cu exit code `0` dacă nu sunt alerte, `1` dacă sunt.

<details>
<summary>✅ Soluție completă (referință)</summary>

```bash
#!/usr/bin/env bash
set -euo pipefail

readonly LOG_FILE="${LOG_FILE:-/tmp/syscheck.log}"
readonly LOCK_FILE="/tmp/syscheck.lock"
mount_points=(/ /tmp)
critical_process="bash"

threshold=80
verbose=false

usage() {
    cat <<EOF
Usage: $(basename "$0") [-t threshold] [-v] [-h]

  -t N   Prag alertă disk, procent (default: 80)
  -v     Mod verbose
  -h     Acest mesaj
EOF
    exit "${1:-0}"
}

while getopts ":t:vh" opt; do
    case "$opt" in
        t) threshold="$OPTARG" ;;
        v) verbose=true ;;
        h) usage 0 ;;
        :) echo "Eroare: -$OPTARG necesită un argument" >&2; usage 1 ;;
        \?) echo "Opțiune necunoscută: -$OPTARG" >&2; usage 1 ;;
    esac
done
shift $(( OPTIND - 1 ))

log() {
    local level="$1"; shift
    printf '[%s] [%s] %s\n' "$(date '+%Y-%m-%d %H:%M:%S')" "$level" "$*" | tee -a "$LOG_FILE"
}
log_info()  { log "INFO " "$@"; }
log_warn()  { log "WARN " "$@" >&2; }
log_error() { log "ERROR" "$@" >&2; }

cleanup() {
    local exit_code=$?
    rm -f "$LOCK_FILE"
    log_info "Cleanup efectuat (exit code: $exit_code)"
    exit "$exit_code"
}
trap cleanup EXIT INT TERM

if [[ -f "$LOCK_FILE" ]]; then
    log_error "Scriptul rulează deja (PID $(cat "$LOCK_FILE"))"
    exit 1
fi
echo $$ > "$LOCK_FILE"

check_disk() {
    local mount_point="$1" thresh="$2" usage
    usage=$(df -h "$mount_point" | awk 'NR==2 {print $5}' | tr -d '%')
    if (( usage >= thresh )); then
        log_warn "Disk ($mount_point): ${usage}% [ALERTĂ, prag ${thresh}%]"
        return 1
    fi
    $verbose && log_info "Disk ($mount_point): ${usage}% [OK]"
    return 0
}

check_process() {
    local process_name="$1"
    if pgrep -x "$process_name" >/dev/null 2>&1; then
        $verbose && log_info "Proces '$process_name': [RULEAZĂ]"
        return 0
    fi
    log_warn "Proces '$process_name': [OPRIT]"
    return 1
}

alert_count=0
log_info "syscheck pornit (threshold=${threshold}%, verbose=${verbose})"

for mp in "${mount_points[@]}"; do
    check_disk "$mp" "$threshold" || (( alert_count++ ))
done

check_process "$critical_process" || (( alert_count++ ))

log_info "Verificare completă — $alert_count alerte găsite"

if (( alert_count > 0 )); then
    exit 1
fi
exit 0
```

Testează:
```bash
./syscheck.sh -v                 # mod verbose, threshold default
./syscheck.sh -t 1                # prag foarte mic, forțează alerte
echo $?                            # verifică exit code-ul
cat /tmp/syscheck.log               # verifică logul
```
</details>

---

## ✅ Checklist final

- [ ] Scriptul trece `shellcheck` fără avertismente
- [ ] Folosește `local` peste tot în funcții, nicio variabilă globală accidentală
- [ ] Nu se poate rula de două ori simultan (lock file funcțional)
- [ ] Cleanup-ul rulează garantat, inclusiv la `Ctrl+C`
- [ ] Suportă `-h`, `-t`, `-v` prin `getopts`
- [ ] Exit code reflectă corect rezultatul (0 = ok, 1 = alerte)

### Provocare bonus
Adaugă un flag `-o <fisier>` care exportă rezultatele ca JSON (folosind `printf` sau `jq -n`), util pentru integrarea cu un sistem de monitorizare extern.
