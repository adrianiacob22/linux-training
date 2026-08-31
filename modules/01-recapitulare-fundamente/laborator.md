# 🧪 Laborator — Modul 1: Recapitulare & Fundamente Solide

[← Înapoi la Modul 1](README.md) · [Cuprins curs](../../README.md)

## Scenariu

Tocmai ai primit acces la un server Linux proaspăt instalat și urmează să-l folosești zilnic. Înainte să faci orice altceva, vrei să-ți cunoști terenul: cum arată structura sistemului, cum cauți rapid informații, și cum îți personalizezi mediul de lucru ca să fii eficient.

## Cerințe prealabile
- Acces la un shell Bash (Linux, WSL, sau un container `docker run -it ubuntu bash`)
- Drepturi de citire în `/proc`, `/var/log` (nu e nevoie de root pentru majoritatea task-urilor)

---

## Task 1 — Explorează structura sistemului

1. Află ce shell rulezi și ce versiune de kernel are sistemul.
2. Afișează câte procese rulează în acest moment, citind direct din `/proc` (fără `ps`).
3. Găsește PID-ul propriului shell folosind variabila specială potrivită.

<details>
<summary>💡 Hint</summary>

`$SHELL` sau `echo $0` pentru shell, `uname -r` pentru kernel. `/proc` conține un director numeric per proces — numără-le cu `ls` + `wc -l`. PID-ul shell-ului curent e o variabilă specială din §1.2/7.2.
</details>

<details>
<summary>✅ Soluție</summary>

```bash
echo $0                    # numele shell-ului curent
uname -r                    # versiune kernel
ls -d /proc/[0-9]* | wc -l   # câte directoare numerice (= procese) sunt în /proc
echo $$                      # PID-ul shell-ului curent
```
</details>

---

## Task 2 — Pipeline de curățenie

Simulează o curățenie de loguri: găsește toate fișierele `.log` din `/var/log` (dacă nu ai destule, creează câteva de test în `/tmp/logs`), afișează doar numele celor care conțin cuvântul `error` (case-insensitive), și numără-le.

<details>
<summary>💡 Hint</summary>

Combină `find`, `grep -il`, și `wc -l` într-un singur pipeline.
</details>

<details>
<summary>✅ Soluție</summary>

```bash
# Date de test, dacă nu ai loguri reale la îndemână
mkdir -p /tmp/logs
echo "everything fine" > /tmp/logs/app1.log
echo "ERROR: disk full" > /tmp/logs/app2.log
echo "connection ERROR timeout" > /tmp/logs/app3.log

find /tmp/logs -name "*.log" | xargs grep -il "error" | wc -l
# sau, mai robust cu nume ce pot conține spații:
find /tmp/logs -name "*.log" -print0 | xargs -0 grep -il "error" | wc -l
```
</details>

---

## Task 3 — Editare cu Vim

Creează un fișier `/tmp/server.conf` cu următorul conținut (folosind Vim, nu `echo`/`cat`):

```
host=localhost
port=8080
debug=false
timeout=30
```

Apoi, tot în Vim:
- schimbă `debug=false` în `debug=true`
- adaugă o linie nouă `retries=3` la final
- salvează și ieși

<details>
<summary>💡 Hint</summary>

`vim /tmp/server.conf`, apoi `i` pentru insert, `Esc` pentru a ieși din insert, `:wq` pentru salvare+ieșire. Pentru substituție rapidă: `:%s/debug=false/debug=true/`.
</details>

<details>
<summary>✅ Soluție (secvență de taste în Vim)</summary>

```
i
host=localhost
port=8080
debug=false
timeout=30
<Esc>
:%s/debug=false/debug=true/
G
oretries=3
<Esc>
:wq
```
</details>

---

## Task 4 — Personalizează `.bashrc`

Adaugă în `~/.bashrc`:
- alias `ll` → `ls -lah`
- funcție `mkcd` care creează un director și intră direct în el
- `$HOME/bin` la începutul lui `$PATH` (doar dacă nu e deja acolo)

Apoi reîncarcă shell-ul fără să deschizi un terminal nou și testează `mkcd /tmp/testdir`.

<details>
<summary>✅ Soluție</summary>

```bash
cat >> ~/.bashrc <<'EOF'

alias ll='ls -lah'

mkcd() {
    mkdir -p "$1" && cd "$1"
}

case ":$PATH:" in
    *":$HOME/bin:"*) ;;
    *) export PATH="$HOME/bin:$PATH" ;;
esac
EOF

source ~/.bashrc
mkcd /tmp/testdir
pwd    # ar trebui să afișeze /tmp/testdir
```
</details>

---

## Task 5 — Documentație

Fără să folosești un motor de căutare web, află:
1. Ce face flag-ul `-i` la comanda `cp` (folosind `man`).
2. În ce secțiune de manual se află `passwd` ca *fișier* (nu ca și comandă), și ce diferență e față de comanda `passwd`.

<details>
<summary>✅ Soluție</summary>

```bash
man cp | grep -A2 '^ *-i'      # sau: man cp, apoi caută "/-i" în pager

man -k passwd                    # apropos — arată toate paginile "passwd" disponibile
man 5 passwd                      # secțiunea 5 = formate de fișiere → /etc/passwd
man 1 passwd                       # secțiunea 1 = comenzi user → comanda passwd
```

`-i` la `cp` = mod interactiv, cere confirmare înainte de suprascrierea unui fișier existent.
</details>

---

## ✅ Checklist final

- [ ] Știu să navighez și să interpretez `/proc`
- [ ] Pot construi un pipeline `find | grep | xargs` din memorie
- [ ] Mă descurc să editez și să salvez un fișier în Vim fără să "rămân blocat"
- [ ] Am un `.bashrc` personalizat cu alias-uri și funcții proprii
- [ ] Știu să caut rapid documentație cu `man`/`man -k`
