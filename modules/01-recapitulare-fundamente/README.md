# 📦 Modul 1 — Recapitulare & Fundamente Solide

> **Nivel:** Mediu | **Durată estimată:** 3h

[← Înapoi la cuprins](../../README.md)

## Obiective
- Consolidarea cunoștințelor de bază
- Înțelegerea filozofiei Unix

---

## 1.1 — Structura sistemului Linux

Linux este alcătuit din trei straturi principale, fiecare cu un rol clar:

```
┌─────────────────────────────────────┐
│   Userspace (aplicații, utilitare)   │  ← bash, ls, firefox, nginx...
├─────────────────────────────────────┤
│   Shell (interpretor de comenzi)     │  ← bash, zsh, dash, sh
├─────────────────────────────────────┤
│   Kernel (nucleul sistemului)        │  ← gestionează CPU, memorie, I/O
├─────────────────────────────────────┤
│   Hardware                           │
└─────────────────────────────────────┘
```

- **Kernel** — programul central care rulează în kernel space, cu acces direct la hardware. Gestionează procese, memorie, drivere, sistemul de fișiere și rețeaua. Comunicarea cu userspace se face prin **system calls** (`syscalls`).
- **Shell** — interpretorul de comenzi care stă între utilizator și kernel. Cele mai comune: `bash` (default pe majoritatea distribuțiilor), `sh`/`dash` (POSIX, folosit pentru scripturi de sistem — mai rapid, mai puține funcționalități), `zsh` (interactiv, foarte configurabil).
- **Userspace** — tot ce rulează în afara kernel-ului: utilitare GNU (`coreutils`), aplicații, servicii. Rulează izolat de kernel; un crash aici nu (ar trebui să) doboare sistemul.

**Filozofia Unix** — câteva principii care explică de ce Linux arată așa cum arată:
1. *"Do one thing and do it well"* — fiecare comandă are un scop restrâns (`grep` caută, `sort` sortează, `cut` extrage câmpuri).
2. *"Everything is a file"* — dispozitive (`/dev/sda`), procese (`/proc/1234`), socket-uri, chiar și kernel-ul expune interfețe ca fișiere (`/sys`).
3. **Compunerea prin pipe-uri** — programe mici combinate cu `|` rezolvă probleme complexe fără să reinventezi funcționalitatea.
4. **Text ca interfață universală** — output-ul unei comenzi e text simplu, ușor de procesat de următoarea comandă din pipeline.

```bash
# "Everything is a file" în practică
cat /proc/cpuinfo          # informații CPU expuse ca fișier text
cat /proc/1/comm           # numele procesului cu PID 1
echo 1 > /sys/class/leds/led0/brightness   # controlezi hardware scriind într-un fișier
```

---

## 1.2 — Comenzi esențiale recapitulate

| Comandă | Scop | Exemple utile |
|---------|------|----------------|
| `ls` | listare fișiere | `ls -lah` (detaliat + human-readable + ascunse), `ls -lt` (sortat după dată) |
| `cd` | schimbare director | `cd -` (director anterior), `cd ~user` (home altui user) |
| `cp` | copiere | `cp -r dir1/ dir2/` (recursiv), `cp -a` (păstrează permisiuni/timestamp) |
| `mv` | mutare/redenumire | `mv fisier.txt{,.bak}` (trick redenumire rapidă) |
| `rm` | ștergere | `rm -rf dir/` (⚠️ recursiv, forțat — folosește cu grijă) |
| `find` | căutare fișiere | `find /var -name "*.log" -mtime -7` (loguri din ultimele 7 zile) |
| `grep` | căutare text | `grep -rn "TODO" --include="*.py" .` (recursiv, cu nr. linie) |

```bash
# find: combinații utile
find / -type f -size +100M 2>/dev/null       # fișiere > 100MB, ignoră erorile de acces
find . -name "*.tmp" -delete                  # șterge toate fișierele .tmp găsite
find /home -maxdepth 1 -type d                # doar subdirectoare directe

# grep: combinații utile
grep -i "error" app.log                       # case-insensitive
grep -v "^#" config.conf                      # exclude liniile de comentariu
grep -c "warning" app.log                     # doar numărul de potriviri
grep -B2 -A2 "Exception" app.log               # 2 linii context înainte/după
```

---

## 1.3 — Redirectări & Pipes avansate

Fiecare proces are trei "canale" implicite: **stdin** (0), **stdout** (1), **stderr** (2).

```bash
comanda > fisier.txt      # redirectează stdout, suprascrie fișierul
comanda >> fisier.txt     # redirectează stdout, adaugă la final
comanda 2> erori.txt      # redirectează doar stderr
comanda > out.txt 2>&1    # stdout ȘI stderr în același fișier (ordinea contează!)
comanda &> out.txt        # scurtătură bash pentru linia de mai sus
comanda 2>/dev/null       # aruncă erorile, le ignoră complet
```

- **`|` (pipe)** — leagă stdout-ul unei comenzi de stdin-ul următoarei.
- **`tee`** — "despică" stdout-ul: îl scrie într-un fișier ȘI îl trimite mai departe pe pipeline.
- **`xargs`** — transformă output-ul unei comenzi (linii de text) în argumente pentru o altă comandă.

```bash
# tee: vezi output-ul live ȘI îl salvezi
ping -c5 google.com | tee ping-log.txt

# xargs: șterge toate fișierele .log găsite cu find
find /var/log -name "*.log" | xargs rm -f

# xargs -I{} pentru control fin asupra poziției argumentului
find . -name "*.txt" | xargs -I{} mv {} {}.bak

# xargs -P pentru paralelism
cat urls.txt | xargs -P4 -I{} curl -sO {}

# Pipeline complex: procese care consumă cea mai multă memorie
ps aux | sort -k4 -rn | head -5
```

**Atenție la `xargs` cu nume de fișiere ce conțin spații** — folosește `-print0` / `-0`:
```bash
find . -name "*.txt" -print0 | xargs -0 rm -f
```

---

## 1.4 — Editoare de text

### Vim — moduri și comenzi esențiale

Vim funcționează pe baza mai multor **moduri**:

| Mod | Intrare | Ieșire |
|-----|---------|--------|
| Normal | implicit / `Esc` | — |
| Insert | `i`, `a`, `o`, `O` | `Esc` |
| Visual | `v` (caracter), `V` (linie), `Ctrl+v` (bloc) | `Esc` |
| Command | `:` din Normal | `Enter` sau `Esc` |

```
i          intră în Insert înainte de cursor      dd     șterge linia curentă
a          intră în Insert după cursor             yy     copiază linia curentă
o          linie nouă sub, intră în Insert          p      lipește după cursor
Esc        revino în Normal                        u      undo
dw         șterge cuvântul curent                   Ctrl+r redo
x          șterge caracterul de sub cursor          /text  caută "text"
:w         salvează                                 n      următoarea potrivire
:q         ieșire                                    :%s/a/b/g  înlocuiește a→b în tot fișierul
:wq / :x   salvează și ieșire                        gg / G  început / sfârșit fișier
:q!        ieșire fără salvare                       dG      șterge de la cursor până la final
```

### Nano
Mai simplu, comenzile apar jos pe ecran (`^` = `Ctrl`): `Ctrl+O` salvează, `Ctrl+X` iese, `Ctrl+K` taie linia, `Ctrl+U` lipește, `Ctrl+W` caută.

### Sed — bazele
`sed` procesează text linie cu linie, aplicând comenzi tip editor:

```bash
sed 's/vechi/nou/' fisier.txt        # înlocuiește prima apariție pe fiecare linie
sed 's/vechi/nou/g' fisier.txt       # înlocuiește toate aparițiile (global)
sed -n '5,10p' fisier.txt            # afișează doar liniile 5-10
sed '3d' fisier.txt                  # șterge linia 3
sed -i 's/vechi/nou/g' fisier.txt    # modifică fișierul in-place
```

---

## 1.5 — Variabile de mediu

- **`$PATH`** — listă de directoare (separate prin `:`) în care shell-ul caută executabile. Ordinea contează — primul director care conține comanda "câștigă".
- **`$HOME`** — directorul home al utilizatorului curent (`~`).
- **`export`** — face o variabilă vizibilă și pentru procesele copil, nu doar în shell-ul curent.

```bash
echo $PATH
# /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin

# Adaugă un director custom la PATH (persistent, în .bashrc)
export PATH="$HOME/bin:$PATH"

which python3        # arată din ce director din PATH se ia comanda
type -a python3       # arată toate locațiile posibile (inclusiv alias-uri)
```

**Fișiere de configurare — când se încarcă fiecare:**

| Fișier | Când se citește |
|--------|------------------|
| `/etc/profile` | shell login, pentru toți utilizatorii |
| `~/.bash_profile` / `~/.profile` | shell login, per utilizator |
| `~/.bashrc` | shell interactiv non-login (ex: terminal nou dintr-un GUI) |
| `~/.bash_logout` | la ieșirea dintr-un shell login |

> În practică: `.bash_profile` de obicei doar face `source ~/.bashrc`, ca să ai aceleași setări indiferent cum ai deschis shell-ul.

```bash
# ~/.bashrc — exemple utile
export EDITOR=vim
export HISTSIZE=10000
alias ll='ls -lah'
alias gs='git status'
gcd() { cd "$1" && ls; }   # funcție simplă: cd + ls într-un pas
```

---

## 1.6 — Manuale & Documentație

```bash
man ls              # manual pagină standard
man 5 passwd         # secțiunea 5 (format fișiere) — vezi man man pentru toate secțiunile
man -k partition      # caută în toate manualele care conțin "partition" (= apropos)
info coreutils        # documentație GNU info, mai detaliată decât man pentru unele tool-uri
ls --help             # rezumat rapid al opțiunilor, fără să deschizi man
tldr tar              # exemple practice, concise (necesită instalare: apt install tldr)
```

**Secțiunile man:** 1 = comenzi user, 2 = system calls, 3 = biblioteci C, 5 = formate fișiere/config, 8 = comenzi admin.

---

## 🧪 Laborator practic

[👉 Laborator Modul 1](laborator.md) — un scenariu ghidat, pas cu pas, cu soluții ascunse pentru autoverificare.

## Exerciții practice

1. **Pipeline complex** — găsește toate fișierele `.log` mai vechi de 30 zile din `/var/log`, filtrează-le pe cele care conțin cuvântul `error`, apoi arhivează-le cu `tar`:
   ```bash
   find /var/log -name "*.log" -mtime +30 -print0 \
     | xargs -0 grep -l "error" \
     | xargs tar -czf old-error-logs.tar.gz
   ```

2. **Personalizează `.bashrc`** — adaugă:
   - un alias `ll` pentru `ls -lah`
   - o funcție `mkcd` care creează un director și intră direct în el (`mkdir -p "$1" && cd "$1"`)
   - `PATH` extins cu `$HOME/bin`

3. **Bonus** — scrie o comandă cu `sed` care șterge toate liniile goale și comentariile (`#...`) dintr-un fișier de configurare, salvând rezultatul într-un fișier nou (fără `-i`).
