# 📦 Modul 5 — Gestionarea Utilizatorilor & Grupurilor

> **Nivel:** Mediu | **Durată estimată:** 3h

[← Înapoi la cuprins](../../README.md)

## Obiective
- Administrarea completă a utilizatorilor
- Configurarea autentificării

---

## 5.1 — Fișiere sistem

Toată informația despre utilizatori și grupuri e stocată în text simplu, editabilă direct (deși nu recomandat manual — există unelte dedicate).

**`/etc/passwd`** — un rând per utilizator, câmpuri separate prin `:`
```
alice:x:1001:1001:Alice Popescu:/home/alice:/bin/bash
 │    │  │    │       │            │            └─ shell implicit
 │    │  │    │       │            └─ home directory
 │    │  │    │       └─ GECOS (nume complet / info)
 │    │  │    └─ GID (grup principal)
 │    │  └─ UID
 │    └─ parola — 'x' = stocată în /etc/shadow, nu aici
 └─ username
```

**`/etc/shadow`** — parolele criptate (hash), citibil doar de root:
```
alice:$6$abc123...:19600:0:90:7:::
       │            │     │  │  │
       │            │     │  │  └─ zile de grație după expirare
       │            │     │  └─ zile de avertizare înainte de expirare
       │            │     └─ zile minime între schimbări de parolă
       │            └─ vârsta parolei (zile de la epoch la ultima schimbare)
       └─ hash-ul parolei ($6$ = SHA-512)
```

**`/etc/group`** — un rând per grup:
```
devs:x:2001:alice,bob
 │   │  │      └─ membri suplimentari (grup secundar)
 │   │  └─ GID
 │   └─ parolă grup (rar folosită azi)
 └─ nume grup
```

**`/etc/gshadow`** — parole/administratori de grup (rar folosit în practică modernă).

> **UID/GID convenții:** `0` = root, `1-999` = utilizatori/grupuri de sistem (servicii), `1000+` = utilizatori umani obișnuiți (pragul variază puțin: 500 pe unele distro RHEL vechi).

---

## 5.2 — Gestionare utilizatori

```bash
useradd -m -s /bin/bash -c "Alice Popescu" alice     # -m creează home dir, -s shell, -c comentariu/GECOS
useradd -m -u 1050 -g devs -G sudo,docker bob             # UID specific, grup principal + grupuri secundare

usermod -aG docker alice        # ADAUGĂ la un grup secundar (⚠️ -a e esențial — fără el, ÎNLOCUIEȘTE toate grupurile!)
usermod -s /usr/sbin/nologin serviciu    # schimbă shell-ul (util pentru conturi de serviciu, fără login interactiv)
usermod -L alice                  # blochează contul (lock) fără să-l șteargă
usermod -U alice                   # deblochează

userdel alice                        # șterge userul, PĂSTREAZĂ home directory-ul
userdel -r alice                       # șterge userul ȘI home directory-ul + mail spool

passwd alice                            # setează/schimbă parola interactiv
passwd -l alice                           # lock (echivalent parțial cu usermod -L)
echo "alice:ParolaNoua123" | chpasswd       # setare parolă non-interactivă (util în scripturi de provisioning)
```

⚠️ **Greșeala clasică:** `usermod -G docker alice` (fără `-a`) **înlocuiește** toate grupurile secundare ale lui alice cu doar `docker` — dacă avea și alte grupuri, le pierde silențios.

---

## 5.3 — Gestionare grupuri

```bash
groupadd devs                    # grup nou
groupadd -g 2050 qa                # cu GID specific

groupmod -n developers devs         # redenumește grupul
groupdel qa                          # șterge grupul (eșuează dacă e grup principal al vreunui user)

gpasswd -a alice devs                  # adaugă alice la grupul devs (echivalent cu usermod -aG)
gpasswd -d alice devs                    # scoate alice din grupul devs

groups alice                               # ce grupuri are alice
id alice                                     # UID, GID, toate grupurile — cel mai complet
```

---

## 5.4 — sudo & su

**`su`** — schimbă complet utilizatorul curent (necesită parola contului **țintă**):
```bash
su - alice          # "-" = simulează un login complet nou (mediu, PATH, HOME resetate)
su alice              # păstrează mediul curent (mai puțin curat, evită dacă poți)
```

**`sudo`** — execută **o comandă** cu privilegii elevate (necesită **propria** parolă, nu a root-ului):
```bash
sudo systemctl restart nginx
sudo -u www-data whoami       # rulează ca alt utilizator, nu neapărat root
sudo -i                          # shell root complet interactiv (ca su - dar prin sudo)
sudo -l                            # arată ce poate rula utilizatorul curent cu sudo
```

**`/etc/sudoers`** — se editează **exclusiv** cu `visudo` (validează sintaxa înainte de salvare — o eroare aici te poate bloca din sudo complet):
```bash
visudo
```
```
# Sintaxă: user  HOST=(runas_user) comenzi
alice   ALL=(ALL:ALL) ALL                        # acces total, ca root, pe orice host
bob      ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx    # o singură comandă, fără parolă
%devs     ALL=(ALL) /usr/bin/docker                                  # tot grupul devs, doar comanda docker

# Mai bine: fișiere separate în /etc/sudoers.d/ (nu editezi direct sudoers)
```
```bash
echo 'bob ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx' > /etc/sudoers.d/bob-nginx
visudo -cf /etc/sudoers.d/bob-nginx     # validează sintaxa fișierului nou ÎNAINTE să-l lași acolo
chmod 440 /etc/sudoers.d/bob-nginx        # permisiuni obligatorii, altfel sudo îl ignoră
```

---

## 5.5 — PAM — Pluggable Authentication Modules

PAM e stratul care decide **cum** se face autentificarea (parolă, cheie, biometrie, reguli de complexitate) — independent de aplicația care o cere (login, sshd, sudo...).

```
/etc/pam.d/sshd
/etc/pam.d/sudo
/etc/pam.d/common-password   (Debian/Ubuntu — reguli comune de parolă)
```

Fiecare linie dintr-un fișier PAM: `tip   control   modul   argumente`

```
# /etc/pam.d/common-password (extras)
password  requisite  pam_pwquality.so retry=3 minlen=12 dcredit=-1 ucredit=-1
password  required   pam_unix.so obscure sha512
```

- **tip**: `auth` (cine ești), `account` (contul e valid/activ?), `password` (schimbare parolă), `session` (setup la login/logout).
- **control**: `required` (trebuie să treacă, dar continuă evaluarea altor module înainte de a refuza), `requisite` (trebuie să treacă, oprește imediat la eșec), `sufficient` (dacă trece, e suficient), `optional`.
- **module comune**: `pam_unix.so` (autentificare clasică prin `/etc/shadow`), `pam_pwquality.so` (complexitate parolă), `pam_faillock.so` (blocare după N încercări eșuate), `pam_limits.so` (aplică `/etc/security/limits.conf`).

---

## 5.6 — Politici de parole

```bash
chage -l alice                  # afișează politica curentă de expirare pentru alice
chage -M 90 alice                 # parola expiră la fiecare 90 zile
chage -m 7 alice                    # minim 7 zile între schimbări (previne "schimb de 10 ori rapid" ca să revii la vechea parolă)
chage -W 14 alice                     # avertizare cu 14 zile înainte de expirare
chage -E 2026-12-31 alice                # data expirării CONTULUI (nu doar a parolei)
chage -d 0 alice                           # forțează schimbarea parolei la următorul login
```

Complexitate parolă (via `pam_pwquality`, în `/etc/security/pwquality.conf`):
```
minlen = 12
dcredit = -1      # necesită cel puțin 1 cifră
ucredit = -1       # necesită cel puțin 1 literă mare
ocredit = -1        # necesită cel puțin 1 caracter special
```

---

## 5.7 — Logare & Audit

```bash
who                    # cine e logat acum, pe ce terminal, de când
w                        # ca who, plus ce comandă rulează fiecare (mai detaliat)
last                      # istoric de login-uri (din /var/log/wtmp)
last -n 20                 # ultimele 20 de intrări
last reboot                  # istoric de reboot-uri
lastlog                        # ultimul login al FIECĂRUI utilizator (inclusiv "never logged in")

faillock --user alice             # (RHEL/Fedora) încercări eșuate de autentificare pentru alice
faillock --user alice --reset       # resetează contorul (deblochează după prea multe eșecuri)
```

---

## 🧪 Laborator practic

[👉 Laborator Modul 5](laborator.md) — provizionează utilizatori și grupuri pentru o echipă, configurează sudo granular și o politică de parole completă.

## Exerciții practice
- Configurează sudo cu drepturi granulare pentru un utilizator de serviciu
- Implementează o politică de parole cu PAM
