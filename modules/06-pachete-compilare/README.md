# 📦 Modul 6 — Managementul Pachetelor & Compilare din Sursă

> **Nivel:** Mediu | **Durată estimată:** 3h

[← Înapoi la cuprins](../../README.md)

## Obiective
- Managementul avansat al pachetelor software
- Compilarea și instalarea manuală

---

## 6.1 — APT avansat (Debian/Ubuntu)

```bash
apt update                       # actualizează lista de pachete disponibile (nu instalează nimic)
apt upgrade                        # upgradează pachetele instalate la versiuni noi
apt install nginx=1.18.0-0ubuntu1     # instalează o versiune SPECIFICĂ
apt remove nginx                        # dezinstalează, păstrează fișierele de config
apt purge nginx                           # dezinstalează + șterge fișierele de config
apt autoremove                              # curăță dependențele orfane, nemaifolosite

apt-cache search "web server"        # caută pachete după descriere
apt-cache policy nginx                  # ce versiuni sunt disponibile și din ce repo
apt-cache show nginx                      # detalii complete despre pachet

dpkg -l | grep nginx                # listează pachetele instalate (dpkg = unealta low-level, apt e wrapper-ul)
dpkg -L nginx                         # ce fișiere a instalat pachetul nginx
dpkg -S /usr/sbin/nginx                 # cărui pachet îi aparține un fișier
dpkg -i pachet.deb                        # instalează un .deb local
```

**Repository-uri** — definite în `/etc/apt/sources.list` și `/etc/apt/sources.list.d/*.list`:
```
deb http://archive.ubuntu.com/ubuntu jammy main restricted universe
deb https://download.docker.com/linux/ubuntu jammy stable
```
```bash
apt-key add docker.gpg      # (deprecat) — modern: cheile merg în /etc/apt/keyrings/, referite explicit în sources.list
```

**APT Pinning** — controlează ce sursă/versiune are prioritate când același pachet e disponibil din mai multe locuri, în `/etc/apt/preferences.d/`:
```
Package: nginx
Pin: release a=jammy-backports
Pin-Priority: 500
```

---

## 6.2 — YUM/DNF (RHEL/CentOS/Fedora)

`dnf` e succesorul modern al `yum` (sintaxă aproape identică, rezolvare de dependențe mai bună).

```bash
dnf install nginx
dnf remove nginx
dnf update                      # echivalentul lui apt upgrade
dnf search "web server"

dnf info nginx                    # detalii pachet
dnf list installed | grep nginx     # ce e instalat
dnf provides */nginx                  # cărui pachet îi aparține un fișier/binar

rpm -qa | grep nginx                 # listare low-level (rpm = echivalentul lui dpkg)
rpm -ql nginx                          # ce fișiere a instalat
rpm -qf /usr/sbin/nginx                  # cărui pachet îi aparține un fișier
rpm -ivh pachet.rpm                        # instalare directă dintr-un fișier .rpm local

dnf repolist                        # repo-urile active
dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

**Modular repos** (DNF, RHEL 8+) — permit mai multe versiuni ale aceluiași software, activate explicit:
```bash
dnf module list nodejs
dnf module enable nodejs:18
dnf install nodejs
```

---

## 6.3 — Compilare din sursă

Când nu există pachet pre-compilat sau ai nevoie de opțiuni custom (module specifice, optimizări), compilezi manual — fluxul clasic **GNU Autotools**:

```bash
# 1. Dependențe de compilare (exemplu Debian/Ubuntu)
apt install build-essential libpcre3-dev zlib1g-dev libssl-dev

# 2. Descarcă și dezarhivează sursele
tar -xzf nginx-1.25.0.tar.gz
cd nginx-1.25.0

# 3. Configurează build-ul (verifică dependențe, generează Makefile)
./configure --prefix=/usr/local/nginx --with-http_ssl_module --with-http_v2_module

# 4. Compilează (folosește toate core-urile disponibile pentru viteză)
make -j$(nproc)

# 5. Instalează (de regulă necesită root, scrie în /usr/local/...)
sudo make install
```

- `./configure --help` — listează toate opțiunile disponibile pentru acel pachet specific (module, căi custom).
- Dacă `./configure` eșuează cu "missing dependency", instalează pachetul `-dev`/`-devel` corespunzător (header-e C, nu doar binarul).
- `make uninstall` — funcționează doar dacă `Makefile`-ul îl suportă explicit (nu toate proiectele îl oferă).

---

## 6.4 — Snap & Flatpak

Formate de pachete **universale**, containerizate, independente de distribuție.

| | Snap | Flatpak |
|---|---|---|
| Dezvoltator | Canonical (Ubuntu) | comunitate (freedesktop.org) |
| Sandboxing | da (confinement) | da (portals) |
| Store implicit | Snap Store | Flathub |
| Update | automat, în fundal | manual/automat, configurabil |

```bash
snap install code --classic       # --classic = fără sandboxing complet, necesar pt. anumite aplicații (IDE-uri)
snap list
snap remove code

flatpak install flathub org.videolan.VLC
flatpak list
flatpak update
```

**Avantaje:** instalare identică pe orice distro, dependențe incluse (nu mai ai conflicte de versiuni de biblioteci).
**Dezavantaje:** dimensiune mai mare (fiecare pachet aduce propriile dependențe), pornire ceva mai lentă, mai puțină integrare cu tema/sistemul.

---

## 6.5 — Crearea pachetelor simple

**`.deb` — structura minimă:**
```
myapp-1.0/
├── DEBIAN/
│   └── control          # metadate: nume, versiune, dependențe, descriere
└── usr/
    └── local/bin/
        └── myapp          # binarul/scriptul propriu-zis
```
```
# DEBIAN/control
Package: myapp
Version: 1.0
Architecture: amd64
Maintainer: Adrian <adrian@example.com>
Depends: bash
Description: Aplicație de test pentru curs
```
```bash
chmod 755 myapp-1.0/DEBIAN     # DEBIAN/ trebuie 755, nu writable de grup/alții
dpkg-deb --build myapp-1.0 myapp-1.0.deb
dpkg -i myapp-1.0.deb
```

**RPM — schița unui spec file** (`myapp.spec`):
```
Name:       myapp
Version:    1.0
Release:    1
Summary:    Aplicație de test pentru curs
License:    MIT

%description
Aplicație de test.

%files
/usr/local/bin/myapp

%install
mkdir -p %{buildroot}/usr/local/bin
cp myapp %{buildroot}/usr/local/bin/
```
```bash
rpmbuild -bb myapp.spec
```

---

## 6.6 — Gestiunea bibliotecilor

```bash
ldd /usr/sbin/nginx              # ce biblioteci partajate (.so) folosește un binar, și dacă sunt găsite
ldconfig                           # reconstruiește cache-ul de biblioteci (/etc/ld.so.cache) după instalare manuală
ldconfig -p | grep libssl            # caută o bibliotecă specifică în cache

echo $LD_LIBRARY_PATH                  # directoare suplimentare căutate ÎNAINTEA celor standard
export LD_LIBRARY_PATH=/usr/local/nginx/lib:$LD_LIBRARY_PATH    # temporar, pentru sesiunea curentă
```

Pentru biblioteci compilate manual în locații custom (ex: `/usr/local/lib`), adaugă calea persistent:
```bash
echo "/usr/local/lib" > /etc/ld.so.conf.d/local.conf
ldconfig       # aplică imediat, fără reboot
```

Dacă `ldd` arată `=> not found` pentru o bibliotecă, binarul nu va putea porni — fie lipsește pachetul, fie calea nu e în `ldconfig`/`LD_LIBRARY_PATH`.

---

## 🧪 Laborator practic

[👉 Laborator Modul 6](laborator.md) — compilează un server web din sursă cu module custom, apoi împachetează-l ca `.deb` instalabil.

## Exerciții practice
- Compilează și instalează Nginx din sursă cu module custom
- Creează un pachet `.deb` simplu
