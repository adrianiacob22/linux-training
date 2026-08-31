# 🧪 Laborator — Modul 3: Gestionarea Proceselor & Resurselor

[← Înapoi la Modul 3](README.md) · [Cuprins curs](../../README.md)

## Scenariu

Un coleg raportează că serverul de test "e lent". Rolul tău: găsește procesul vinovat, oprește-l corect, apoi pune în loc mecanisme care să prevină și să detecteze din timp astfel de situații — priorități corecte, limite de resurse, și monitorizare automată prin cron.

## Cerințe prealabile
- O mașină/VM/container Linux cu drepturi root sau sudo
- Pachetul `sysstat` pentru `iostat`/`sar` (opțional): `apt install sysstat` / `yum install sysstat`

---

## Task 1 — Creează și găsește procesul "vinovat"

Pornește în background un proces care consumă CPU intens (simulăm situația reală):

```bash
yes > /dev/null &
```

Găsește PID-ul acestui proces în **două** moduri diferite: o dată cu `ps`, o dată cu `pgrep`. Confirmă apoi vizual cu `top`/`htop` că apare printre procesele cu cel mai mare `%CPU`.

<details>
<summary>✅ Soluție</summary>

```bash
yes > /dev/null &
echo "PID pornit direct din shell: $!"     # $! = PID-ul ultimului job pornit în background

ps aux | grep '[y]es'         # trucul [y]es evită ca grep să se auto-matcheze pe sine
pgrep -x yes

top       # apasă 'q' pentru a ieși
```
</details>

---

## Task 2 — Oprire elegantă, apoi forțată

Trimite mai întâi `SIGTERM` procesului `yes`. Verifică dacă s-a oprit. Dacă din vreun motiv nu se oprește (proces "blocat"/care ignoră semnalul), trimite `SIGKILL`.

<details>
<summary>💡 Hint</summary>

`yes` răspunde normal la `SIGTERM`, deci se va opri imediat — e important totuși să exersezi secvența "încearcă elegant, apoi forțat", pentru că multe daemoane reale ignoră `SIGTERM` dacă sunt blocate.
</details>

<details>
<summary>✅ Soluție</summary>

```bash
pid=$(pgrep -x yes | head -1)

kill "$pid"                    # SIGTERM implicit
sleep 1
if kill -0 "$pid" 2>/dev/null; then    # kill -0 = doar verifică dacă procesul mai există, nu trimite semnal real
    echo "Nu a răspuns la SIGTERM, forțez cu SIGKILL"
    kill -9 "$pid"
else
    echo "Procesul s-a oprit elegant"
fi
```
</details>

---

## Task 3 — Prioritate: nice și renice

Pornește din nou `yes > /dev/null &`, dar de data asta cu prioritate scăzută de la început (nice=15). Apoi, fără să-l oprești, schimbă-i prioritatea live la nice=19 (minimul posibil pentru un utilizator normal).

<details>
<summary>✅ Soluție</summary>

```bash
nice -n 15 yes > /dev/null &
pid=$!
ps -o pid,ni,cmd -p "$pid"     # confirmă NI=15

renice -n 19 -p "$pid"
ps -o pid,ni,cmd -p "$pid"      # confirmă NI=19

kill "$pid"     # curățenie
```

Încearcă și `renice -n -5 -p "$pid"` ca utilizator non-root — ar trebui să primești o eroare de tip "Permission denied", pentru că doar root poate crește prioritatea (scădea valoarea nice).
</details>

---

## Task 4 — Background, nohup, disown

Pornește un "task lung" simulat, asigură-te că supraviețuiește dacă închizi terminalul, apoi verifică jobs-urile curente ale shell-ului.

<details>
<summary>✅ Soluție</summary>

```bash
# Varianta 1: nohup de la început
nohup sleep 300 &
disown            # scoate-l și din tabela de jobs a shell-ului curent (opțional, dar complet)

# Varianta 2: proces deja pornit, îl faci "imun" la SIGHUP ulterior
sleep 300 &
jobs                # arată job-ul, ex: [1]+ Running   sleep 300 &
disown -h %1          # nu mai primește SIGHUP la închiderea shell-ului, dar rămâne vizibil ca proces normal

ps -ef | grep '[s]leep 300'    # confirmă că procesul încă rulează, independent de shell
```
</details>

---

## Task 5 — Diagnosticare resurse

Răspunde la aceste întrebări folosind uneltele potrivite (nu presupune, verifică):
1. Câtă memorie liberă (nu doar RAM total) are sistemul, în format citibil de om?
2. Ce proces ascultă (dacă există vreunul) pe portul 22?
3. Ce fișiere are deschise procesul curent al shell-ului tău (`$$`)?

<details>
<summary>✅ Soluție</summary>

```bash
free -h                    # coloana "available" e cea relevantă, nu "free"
lsof -i :22                 # de regulă sshd
lsof -p $$                   # fișierele deschise de shell-ul curent
```
</details>

---

## Task 6 — Limite de resurse

Verifică limita curentă de file descriptors a sesiunii tale, apoi scade-o temporar la o valoare foarte mică (ex: 10) și observă ce se întâmplă când încerci să deschizi mai multe fișiere decât atât.

<details>
<summary>💡 Hint</summary>

`ulimit -n 10` afectează doar shell-ul curent și procesele pornite din el după acest punct — nu afectează restul sistemului. Poți testa deschiderea multor fișiere cu un mic loop care redirectează în file descriptori custom, sau doar observând că o comandă care deschide multe fișiere (ex: un `find` masiv combinat cu ceva ce ține fișiere deschise) eșuează.
</details>

<details>
<summary>✅ Soluție</summary>

```bash
ulimit -n                   # valoarea curentă, de regulă 1024 sau mai mare

(
    ulimit -n 10              # limitează DOAR acest subshell
    ulimit -n                  # confirmă
    for i in $(seq 1 15); do
        exec {fd}>"/tmp/testfile_$i" || { echo "Eșec la fișierul $i — limita a fost atinsă"; break; }
    done
)
rm -f /tmp/testfile_*
```
</details>

---

## Task 7 — Limitare cu cgroups (opțional, necesită systemd)

Rulează `yes > /dev/null` într-un scope cgroup limitat la 10% CPU și confirmă cu `top`/`systemd-cgtop` că limita e respectată.

<details>
<summary>✅ Soluție</summary>

```bash
systemd-run --scope -p CPUQuota=10% -- bash -c 'yes > /dev/null' &

# într-un alt terminal:
systemd-cgtop     # observă consumul real de CPU al scope-ului, ar trebui plafonat în jur de 10%
top                # coloana %CPU pentru procesul "yes" ar trebui să fie ~10, nu 100

pkill -f "yes > /dev/null"
```
</details>

---

## Task 8 — Automatizare cu cron

Creează un job cron care rulează la fiecare 5 minute, verifică load average-ul sistemului (`uptime` sau `/proc/loadavg`), și scrie într-un fișier de log un rând cu timestamp și valoarea curentă. Loghează atât output-ul normal, cât și erorile.

<details>
<summary>✅ Soluție</summary>

```bash
cat > ~/check-load.sh <<'EOF'
#!/usr/bin/env bash
set -euo pipefail
load=$(cut -d' ' -f1-3 /proc/loadavg)
echo "$(date '+%Y-%m-%d %H:%M:%S') — load average: $load"
EOF
chmod +x ~/check-load.sh

crontab -e
# adaugă linia (ajustează calea):
# */5 * * * * $HOME/check-load.sh >> $HOME/load.log 2>&1

crontab -l          # confirmă că s-a salvat
```

Bonus: testează imediat, fără să aștepți 5 minute, cu `at`:
```bash
echo "$HOME/check-load.sh >> $HOME/load.log 2>&1" | at now + 1 minute
atq                  # listează job-urile "at" programate
```
</details>

---

## ✅ Checklist final

- [ ] Pot identifica un proces problematic din mai multe unelte (`ps`, `pgrep`, `top`)
- [ ] Știu secvența corectă: `SIGTERM` întâi, `SIGKILL` doar dacă nu răspunde
- [ ] Pot seta și schimba live prioritatea unui proces (`nice`/`renice`)
- [ ] Înțeleg diferența `nohup` vs `disown`
- [ ] Știu ce unealtă să folosesc pentru fiecare tip de diagnostic (memorie, rețea, fișiere deschise)
- [ ] Pot seta limite de resurse temporare (`ulimit`) și, opțional, cu cgroups
- [ ] Pot scrie și verifica un job cron cu logging complet
