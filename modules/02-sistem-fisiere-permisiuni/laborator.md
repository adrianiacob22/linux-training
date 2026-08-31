# 🧪 Laborator — Modul 2: Sistemul de Fișiere & Permisiuni Avansate

[← Înapoi la Modul 2](README.md) · [Cuprins curs](../../README.md)

## Scenariu

Ești administratorul unei echipe mici de dezvoltare. Trebuie să pregătești infrastructura de fișiere pentru proiectul lor: un director de lucru partajat cu permisiuni corecte pentru mai mulți utilizatori, un volum de stocare dedicat, și protecție pentru fișierele critice.

> 💡 Pentru task-urile cu `useradd`, `pvcreate`, `mount` etc. ai nevoie de root/sudo și, ideal, de o mașină virtuală sau un container privilegiat — nu rula aceste comenzi pe un sistem de producție. Un container Docker cu `--privileged` sau o VM Vagrant/VirtualBox e ideal pentru partea de LVM.

## Cerințe prealabile
- O mașină Linux (VM recomandat) cu drepturi root
- Pentru task-ul LVM: un disc/partiție liberă (sau un disc virtual atașat unei VM)

---

## Task 1 — Pregătește utilizatorii de test

Creează doi utilizatori, `alice` și `bob`, și un grup `proiect-x` din care fac parte amândoi.

<details>
<summary>✅ Soluție</summary>

```bash
groupadd proiect-x
useradd -m -G proiect-x alice
useradd -m -G proiect-x bob
```
</details>

---

## Task 2 — Director partajat cu permisiuni standard + SGID

Creează `/srv/proiect-x`, owner `alice`, grup `proiect-x`, astfel încât:
- membrii grupului să poată citi/scrie/naviga
- alții să nu aibă niciun acces
- orice fișier nou creat de oricine în acest director să moștenească automat grupul `proiect-x` (nu grupul principal al celui care îl creează)

<details>
<summary>💡 Hint</summary>

Al treilea cerință = SGID pe director (§2.3).
</details>

<details>
<summary>✅ Soluție</summary>

```bash
mkdir -p /srv/proiect-x
chown alice:proiect-x /srv/proiect-x
chmod 2770 /srv/proiect-x    # 2 = SGID, 7 = rwx owner, 7 = rwx grup, 0 = nimic pt alții

# Verificare
ls -ld /srv/proiect-x
# drwxrws--- ... alice proiect-x /srv/proiect-x   (s = SGID activ)
```
</details>

---

## Task 3 — ACL pentru un colaborator extern

Un al treilea coleg, `carol`, nu face parte din grupul `proiect-x`, dar are nevoie de acces **doar-citire** la director, fără să-l adaugi în grup. Fișierele noi trebuie să rămână, în continuare, accesibile lui `carol` automat.

<details>
<summary>✅ Soluție</summary>

```bash
useradd -m carol
setfacl -m u:carol:rx /srv/proiect-x       # acces curent
setfacl -d -m u:carol:rx /srv/proiect-x     # ACL implicit → moștenit de fișiere noi

getfacl /srv/proiect-x    # verifică ambele intrări (efective + default)
```
</details>

---

## Task 4 — Volum LVM dedicat

Echipa are nevoie de spațiu de stocare separat, redimensionabil ulterior. Creează un volum LVM de 2GB, formatează-l ext4, și montează-l **peste** `/srv/proiect-x` (persistent, prin `/etc/fstab`), păstrând permisiunile setate anterior.

> Dacă nu ai un disc liber, poți simula unul cu un fișier loopback:
> ```bash
> dd if=/dev/zero of=/tmp/disk.img bs=1M count=2200
> losetup -fP /tmp/disk.img
> losetup -a    # notează device-ul, ex: /dev/loop0
> ```

<details>
<summary>✅ Soluție</summary>

```bash
# înlocuiește /dev/loop0 cu discul/partiția ta reală
pvcreate /dev/loop0
vgcreate vg_proiectx /dev/loop0
lvcreate -L 2G -n lv_proiectx vg_proiectx
mkfs.ext4 /dev/vg_proiectx/lv_proiectx

blkid /dev/vg_proiectx/lv_proiectx    # copiază UUID-ul

# adaugă în /etc/fstab (înlocuiește UUID-ul):
echo 'UUID=<uuid-copiat>  /srv/proiect-x  ext4  defaults  0  2' >> /etc/fstab

mount -a

# re-aplică permisiunile — un mount nou "acoperă" conținutul anterior al directorului
chown alice:proiect-x /srv/proiect-x
chmod 2770 /srv/proiect-x
setfacl -m u:carol:rx /srv/proiect-x
setfacl -d -m u:carol:rx /srv/proiect-x
```

> ⚠️ Observație importantă: montarea unui filesystem nou peste un director existent "ascunde" conținutul vechi (nu îl șterge, doar nu mai e vizibil) — de asta permisiunile trebuie reaplicate pe noul filesystem gol.
</details>

---

## Task 5 — Protejează un fișier de configurare critic

În `/srv/proiect-x`, creează un fișier `config.prod` cu conținut arbitrar și fă-l **immutable** — nici root nu trebuie să-l poată șterge sau modifica accidental fără să scoată explicit protecția.

<details>
<summary>✅ Soluție</summary>

```bash
su - alice -c "echo 'db_host=prod.internal' > /srv/proiect-x/config.prod"
chattr +i /srv/proiect-x/config.prod

rm /srv/proiect-x/config.prod    # ar trebui să eșueze: "Operation not permitted"

lsattr /srv/proiect-x/config.prod    # confirmă flagul 'i'

# pentru a-l face din nou editabil:
chattr -i /srv/proiect-x/config.prod
```
</details>

---

## Task 6 — Hard link vs symbolic link, pe scurt

În `/srv/proiect-x`, creează `notes.txt` cu un text oarecare. Creează apoi un hard link `notes-hard.txt` și un symbolic link `notes-soft.txt` către el. Șterge `notes.txt` și explică (în comentarii) ce se întâmplă cu fiecare link.

<details>
<summary>✅ Soluție</summary>

```bash
su - alice -c '
cd /srv/proiect-x
echo "draft de notite" > notes.txt
ln notes.txt notes-hard.txt
ln -s notes.txt notes-soft.txt
rm notes.txt
cat notes-hard.txt   # funcționează — inode-ul, deci datele, încă există
cat notes-soft.txt    # eșuează — linkul simbolic pointa către numele "notes.txt", care nu mai există
'
```
</details>

---

## ✅ Checklist final

- [ ] Pot crea un director partajat cu owner/grup/SGID corecte din prima încercare
- [ ] Știu să adaug ACL pentru un utilizator fără să-l bag în grup
- [ ] Pot crea, formata și monta persistent un volum LVM
- [ ] Înțeleg de ce trebuie reaplicate permisiunile după un mount nou
- [ ] Știu diferența practică hard link / symbolic link când originalul dispare
- [ ] Pot proteja un fișier critic cu `chattr +i`
