# 📦 Modul 2 — Sistemul de Fișiere & Permisiuni Avansate

> **Nivel:** Mediu | **Durată estimată:** 4h
> 🚧 *Conținut detaliat în lucru*

[← Înapoi la cuprins](../../README.md)

## Obiective
- Înțelegerea ierarhiei FHS (Filesystem Hierarchy Standard)
- Controlul avansat al permisiunilor

## Conținut
1. **FHS — Filesystem Hierarchy Standard** — `/etc`, `/var`, `/proc`, `/sys`, `/dev`, `/tmp`
2. **Permisiuni standard** — `chmod` (octal & simbolic), `chown`, `chgrp`
3. **Permisiuni speciale** — `SUID`, `SGID`, `Sticky Bit`
4. **ACL — Access Control Lists** — `getfacl`, `setfacl`
5. **Linkuri** — hard links vs symbolic links (`ln`, `ln -s`)
6. **Montarea sistemelor de fișiere** — `mount`, `umount`, `/etc/fstab`
7. **Tipuri de sisteme de fișiere** — ext4, xfs, btrfs, tmpfs
8. **LVM — Logical Volume Manager** — PV, VG, LV; creare și redimensionare
9. **Atribute de fișier** — `chattr`, `lsattr`

## Exerciții practice
- Configurează ACL pentru un director partajat între doi utilizatori
- Creează și montează un volum LVM
