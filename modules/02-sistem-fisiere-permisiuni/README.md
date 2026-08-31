# 📦 Modul 2 — Sistemul de Fișiere & Permisiuni Avansate

> **Nivel:** Mediu | **Durată estimată:** 4h

[← Înapoi la cuprins](../../README.md)

## Obiective
- Înțelegerea ierarhiei FHS (Filesystem Hierarchy Standard)
- Controlul avansat al permisiunilor

---

## 2.1 — FHS — Filesystem Hierarchy Standard

Structura de directoare din Linux nu e arbitrară — urmează un standard (FHS) astfel încât orice sistem Linux să fie predictibil.

| Director | Conține |
|----------|---------|
| `/` | root-ul ierarhiei, totul pornește de aici |
| `/bin`, `/sbin` | executabile esențiale (azi de regulă symlink către `/usr/bin`, `/usr/sbin`) |
| `/etc` | fișiere de configurare la nivel de sistem (text, editabile) |
| `/var` | date variabile: loguri (`/var/log`), cozi de mail, cache, baze de date |
| `/home` | directoare home ale utilizatorilor obișnuiți |
| `/root` | home-ul utilizatorului root |
| `/tmp` | fișiere temporare, șterse de regulă la reboot |
| `/usr` | programe și date "read-only" instalate — majoritatea software-ului sistemului |
| `/opt` | software third-party, auto-conținut (ex: aplicații comerciale) |
| `/boot` | kernel, initramfs, configurație bootloader (GRUB) |
| `/lib`, `/lib64` | biblioteci partajate necesare binarelor din `/bin`, `/sbin` |
| `/mnt`, `/media` | puncte de montare temporare (manual / dispozitive amovibile) |
| `/dev` | fișiere de dispozitiv (`/dev/sda`, `/dev/null`, `/dev/tty`) |
| `/proc` | **pseudo-filesystem** — informații live despre kernel și procese, generat în memorie |
| `/sys` | **pseudo-filesystem** — interfață cu kernel-ul pentru drivere și hardware |

`/proc` și `/sys` nu există pe disc — sunt generate dinamic de kernel la fiecare citire:

```bash
cat /proc/meminfo              # memorie disponibilă, live
cat /proc/loadavg              # load average
ls /proc/$$/                    # informații despre shell-ul curent (PID propriu = $$)
cat /sys/class/net/eth0/mtu     # MTU-ul interfeței de rețea, expus ca fișier
```

---

## 2.2 — Permisiuni standard

Fiecare fișier are trei seturi de permisiuni: **user (u)**, **group (g)**, **others (o)**, fiecare cu **read (r=4)**, **write (w=2)**, **execute (x=1)**.

```bash
ls -l fisier.txt
# -rw-r--r-- 1 alice devs 220 Aug 31 12:00 fisier.txt
#  │└┬┘└┬┘└┬┘
#  │ u  g  o
#  └─ tipul fișierului (- normal, d director, l symlink)
```

```bash
# chmod — notație octală (cea mai folosită)
chmod 755 script.sh     # rwxr-xr-x — owner: tot, grup+alții: citire+execuție
chmod 644 fisier.txt     # rw-r--r-- — owner scrie, restul doar citesc
chmod 600 id_rsa          # rw------- — doar owner, tipic pentru chei private

# chmod — notație simbolică (modificări relative)
chmod u+x script.sh       # adaugă execuție pentru owner
chmod go-w fisier.txt      # scoate write pentru grup și alții
chmod a+r fisier.txt        # adaugă read pentru toată lumea (a = all)
chmod -R o-rwx dir/          # recursiv, scoate toate drepturile "others"

# chown / chgrp
chown alice fisier.txt          # schimbă owner
chown alice:devs fisier.txt      # schimbă owner + grup într-un pas
chown -R www-data:www-data /var/www/app   # recursiv, tipic pentru web root
chgrp devs fisier.txt              # schimbă doar grupul
```

---

## 2.3 — Permisiuni speciale

Pe lângă rwx standard, mai există trei biți speciali, reprezentați ca a patra cifră octală (`chmod 4755 ...`):

| Bit | Valoare octală | Efect pe fișier executabil | Efect pe director |
|-----|-----------------|-----------------------------|--------------------|
| **SUID** | 4000 | rulează cu privilegiile **owner-ului**, nu ale celui care execută | (ignorat) |
| **SGID** | 2000 | rulează cu privilegiile **grupului** fișierului | fișierele noi create înăuntru moștenesc **grupul** directorului |
| **Sticky bit** | 1000 | (ignorat, istoric) | doar owner-ul unui fișier (sau root) îl poate șterge, chiar dacă directorul e writable de toți |

```bash
# SUID clasic: /usr/bin/passwd rulează ca root ca să poată scrie în /etc/shadow
ls -l /usr/bin/passwd
# -rwsr-xr-x 1 root root ... /usr/bin/passwd     ('s' în loc de 'x' = SUID activ)
chmod u+s /usr/local/bin/tool     # setează SUID
chmod 4755 /usr/local/bin/tool     # echivalent, notație octală

# SGID pe director: util pentru foldere partajate de o echipă
mkdir /srv/team-share
chgrp devs /srv/team-share
chmod 2775 /srv/team-share     # fișierele noi create aici vor aparține grupului "devs"

# Sticky bit: comportamentul lui /tmp — oricine scrie, dar nu poate șterge fișierele altora
ls -ld /tmp
# drwxrwxrwt ... /tmp     ('t' la final = sticky bit)
chmod +t /srv/shared-uploads
```

⚠️ **SUID pe scripturi shell e ignorat de kernel** (din motive de securitate) — funcționează doar pe binare compilate.

---

## 2.4 — ACL — Access Control Lists

Permisiunile standard rwx suportă un singur owner și un singur grup. Când ai nevoie de reguli pentru mai mulți utilizatori/grupuri simultan, foloseşti **ACL**.

```bash
# Vizualizare ACL curente
getfacl fisier.txt

# Acordă acces read+write unui utilizator specific, în plus față de permisiunile normale
setfacl -m u:bob:rw fisier.txt

# Acordă acces read unui grup specific
setfacl -m g:auditori:r fisier.txt

# ACL implicit pe director — se aplică automat fișierelor NOI create înăuntru
setfacl -d -m u:bob:rw /srv/team-share

# Elimină o intrare ACL specifică
setfacl -x u:bob fisier.txt

# Elimină toate ACL-urile
setfacl -b fisier.txt
```

După `setfacl`, `ls -l` arată un `+` la finalul permisiunilor (`-rw-rwx---+`), semn că există ACL suplimentare vizibile doar cu `getfacl`.

---

## 2.5 — Linkuri: hard links vs symbolic links

Fiecare fișier e reprezentat pe disc de un **inode** (metadate + pointeri către blocuri de date). Un nume de fișier e doar o etichetă care indică spre un inode.

| | Hard link (`ln`) | Symbolic link (`ln -s`) |
|---|---|---|
| Ce este | alt nume pentru **același inode** | fișier separat care conține **calea** către altul |
| Cross-filesystem | ❌ nu poate | ✅ poate |
| Link către director | ❌ interzis | ✅ permis |
| Dacă ștergi originalul | datele rămân (link-ul funcționează) | link-ul devine "broken" (dangling) |
| `ls -l` | apare ca fișier normal | apare cu `l` și `->` către țintă |

```bash
echo "date" > original.txt

ln original.txt hardlink.txt        # hard link — același inode
ln -s original.txt softlink.txt      # symbolic link — pointer către cale

ls -li original.txt hardlink.txt      # -i arată numărul de inode: identic pt hard link
rm original.txt
cat hardlink.txt     # funcționează — datele încă există
cat softlink.txt      # eroare: No such file or directory (link rupt)
```

---

## 2.6 — Montarea sistemelor de fișiere

```bash
lsblk                          # arată discuri/partiții și unde sunt montate
blkid                           # UUID-uri ale partițiilor

mount /dev/sdb1 /mnt/data       # montare manuală
umount /mnt/data                 # demontare
umount -l /mnt/data               # "lazy unmount" — dacă e ocupat (busy)
```

**`/etc/fstab`** — montări automate la boot. Format: `<device> <mount point> <fs type> <options> <dump> <pass>`

```
# /etc/fstab
UUID=1a2b3c4d-...  /data   ext4    defaults        0  2
/dev/sdb2           /backup xfs     defaults,noatime 0  2
tmpfs                /tmp    tmpfs   defaults,size=2G  0  0
```

- Folosește **UUID** în loc de `/dev/sdX` — numele de dispozitiv se pot schimba între boot-uri.
- `mount -a` testează `/etc/fstab` fără reboot (montează tot ce nu e deja montat).
- `mount -fav` face un "dry run" ca să verifici sintaxa înainte de reboot.

---

## 2.7 — Tipuri de sisteme de fișiere

| FS | Caracteristici |
|----|-----------------|
| **ext4** | default istoric pe majoritatea distro-urilor, stabil, jurnalizat, matur |
| **xfs** | performant pentru fișiere mari, folosit des pe RHEL/CentOS, dificil de micșorat |
| **btrfs** | copy-on-write, snapshots native, subvolume-uri, checksums pentru date |
| **tmpfs** | trăiește în RAM, foarte rapid, volatil (se pierde la reboot) |

```bash
mkfs.ext4 /dev/sdb1
mkfs.xfs /dev/sdc1
df -T                    # tipul de filesystem pentru fiecare montare
```

---

## 2.8 — LVM — Logical Volume Manager

LVM adaugă un strat de abstractizare între discuri fizice și partiții, permițând redimensionare flexibilă.

```
Physical Volume (PV)  →  Volume Group (VG)  →  Logical Volume (LV)
   /dev/sdb1                  vg_data              lv_home, lv_data...
```

```bash
# 1. Creează un Physical Volume dintr-o partiție/disc
pvcreate /dev/sdb1

# 2. Grupează PV-uri într-un Volume Group
vgcreate vg_data /dev/sdb1

# 3. Creează un Logical Volume din VG (aici: 20GB)
lvcreate -L 20G -n lv_app vg_data

# 4. Formatează și montează LV-ul ca orice partiție normală
mkfs.ext4 /dev/vg_data/lv_app
mount /dev/vg_data/lv_app /srv/app

# Extindere LV + filesystem (fără downtime, dacă mai e spațiu în VG)
lvextend -L +10G /dev/vg_data/lv_app
resize2fs /dev/vg_data/lv_app        # pentru ext4
xfs_growfs /srv/app                    # pentru xfs (se aplică pe mount point, nu pe device)

# Vizualizare
pvs; vgs; lvs
```

---

## 2.9 — Atribute de fișier — `chattr` / `lsattr`

Atribute la nivel de filesystem, independente de permisiunile rwx — chiar și root trebuie să le scoată explicit.

```bash
chattr +i fisier_critic.conf     # immutable — nici root nu poate șterge/modifica până nu scoate flagul
lsattr fisier_critic.conf         # verifică atributele
chattr -i fisier_critic.conf       # scoate immutable

chattr +a logfile.log               # append-only — se poate scrie doar la final, nu se poate șterge conținut
```

Util pentru fișiere de configurare critice sau loguri de audit care nu trebuie modificate retroactiv.

---

## 🧪 Laborator practic

[👉 Laborator Modul 2](laborator.md) — un scenariu ghidat, pas cu pas, cu soluții ascunse pentru autoverificare.

## Exerciții practice

1. **ACL pentru un director partajat între doi utilizatori** — `alice` trebuie să aibă read+write, `bob` doar read, iar fișierele noi create de oricine trebuie să moștenească aceste reguli automat:
   ```bash
   mkdir /srv/proiect
   chown alice:alice /srv/proiect
   setfacl -m u:alice:rwx /srv/proiect
   setfacl -m u:bob:rx /srv/proiect
   setfacl -d -m u:alice:rwx /srv/proiect     # ACL implicit pentru fișiere noi
   setfacl -d -m u:bob:rx /srv/proiect
   getfacl /srv/proiect                          # verificare
   ```

2. **Creează și montează un volum LVM** de 5GB, formatat ext4, montat persistent în `/mnt/lvtest`:
   ```bash
   pvcreate /dev/sdb1
   vgcreate vg_test /dev/sdb1
   lvcreate -L 5G -n lv_test vg_test
   mkfs.ext4 /dev/vg_test/lv_test
   mkdir -p /mnt/lvtest
   blkid /dev/vg_test/lv_test     # copiază UUID-ul
   # adaugă în /etc/fstab:
   # UUID=<uuid-ul-de-mai-sus>  /mnt/lvtest  ext4  defaults  0  2
   mount -a
   ```

3. **Bonus** — explică (în comentarii) diferența practică dintre a șterge un hard link și a șterge un symbolic link către același fișier, demonstrând cu `ls -li`.
