# 🧪 Laborator — Modul 6: Managementul Pachetelor & Compilare din Sursă

[← Înapoi la Modul 6](README.md) · [Cuprins curs](../../README.md)

## Scenariu

Ai nevoie de o versiune de Nginx compilată manual, cu module specifice care nu vin în pachetul standard din distro. După ce o ai funcțională, vrei să distribui un mic utilitar intern echipei — nu ca sursă, ci ca pachet `.deb` instalabil cu o comandă.

## Cerințe prealabile
- O mașină/VM Debian/Ubuntu cu drepturi sudo (task-urile `.deb` sunt specifice acestei familii de distro)
- Conexiune la internet pentru a descărca sursele Nginx

---

## Task 1 — Pregătește dependențele de compilare

Instalează tot ce e necesar pentru a compila Nginx cu suport SSL și gzip din sursă.

<details>
<summary>✅ Soluție</summary>

```bash
sudo apt update
sudo apt install -y build-essential libpcre3-dev zlib1g-dev libssl-dev wget
```
</details>

---

## Task 2 — Descarcă și configurează sursele

Descarcă o versiune stabilă de Nginx, dezarhiveaz-o, și rulează `./configure` cu:
- prefix de instalare `/opt/nginx-custom`
- modulul SSL activat
- modulul HTTP/2 activat

<details>
<summary>✅ Soluție</summary>

```bash
cd /tmp
wget https://nginx.org/download/nginx-1.25.3.tar.gz
tar -xzf nginx-1.25.3.tar.gz
cd nginx-1.25.3

./configure \
    --prefix=/opt/nginx-custom \
    --with-http_ssl_module \
    --with-http_v2_module
```

Dacă `./configure` se oprește cu o eroare despre o bibliotecă lipsă, citește mesajul cu atenție — de regulă îți spune exact ce pachet `-dev` lipsește.
</details>

---

## Task 3 — Compilează și instalează

Compilează folosind toate core-urile disponibile, apoi instalează.

<details>
<summary>✅ Soluție</summary>

```bash
make -j"$(nproc)"
sudo make install
```
</details>

---

## Task 4 — Verifică build-ul

Confirmă că binarul rezultat chiar are modulele cerute compilate, și că toate bibliotecile de care depinde sunt găsite corect.

<details>
<summary>💡 Hint</summary>

`nginx -V` (majusculă) afișează opțiunile de compilare folosite, nu doar versiunea.
</details>

<details>
<summary>✅ Soluție</summary>

```bash
/opt/nginx-custom/sbin/nginx -V 2>&1 | tr -- '--' '\n--' | grep -i ssl     # confirmă modulul SSL prezent

ldd /opt/nginx-custom/sbin/nginx      # toate liniile ar trebui să arate o cale, nu "not found"
```

Dacă ai instalat vreo bibliotecă manual într-o cale non-standard (nu e cazul aici, dar exersează sintaxa):
```bash
echo "/opt/nginx-custom/lib" | sudo tee /etc/ld.so.conf.d/nginx-custom.conf
sudo ldconfig
```
</details>

---

## Task 5 — Rulează și testează

Pornește serverul compilat pe un port necritic (nginx default e 80, care poate cere root — ajustează în `conf/nginx.conf` la 8080 dacă rulezi ca user normal) și confirmă că răspunde.

<details>
<summary>✅ Soluție</summary>

```bash
sudo sed -i 's/listen       80;/listen       8080;/' /opt/nginx-custom/conf/nginx.conf

sudo /opt/nginx-custom/sbin/nginx
curl -I http://localhost:8080

sudo /opt/nginx-custom/sbin/nginx -s stop     # oprire elegantă
```
</details>

---

## Task 6 — Creează un pachet `.deb` simplu

Împachetează un mic script util (ex: `syscheck.sh` din laboratorul Modulului 7, sau orice script simplu) ca pachet `.deb`, instalabil în `/usr/local/bin`.

<details>
<summary>✅ Soluție</summary>

```bash
mkdir -p ~/mytool-1.0/DEBIAN
mkdir -p ~/mytool-1.0/usr/local/bin

cat > ~/mytool-1.0/usr/local/bin/mytool <<'EOF'
#!/usr/bin/env bash
echo "mytool v1.0 — instalat prin .deb custom"
EOF
chmod 755 ~/mytool-1.0/usr/local/bin/mytool

cat > ~/mytool-1.0/DEBIAN/control <<EOF
Package: mytool
Version: 1.0
Architecture: amd64
Maintainer: Adrian <adrian@example.com>
Depends: bash
Description: Utilitar de test pentru curs Linux
EOF

chmod 755 ~/mytool-1.0/DEBIAN

dpkg-deb --build ~/mytool-1.0 ~/mytool-1.0.deb

sudo dpkg -i ~/mytool-1.0.deb
mytool                              # testează comanda instalată

dpkg -L mytool                        # confirmă ce fișiere a instalat
sudo dpkg -r mytool                     # dezinstalare, verifică că fișierul dispare
```
</details>

---

## Task 7 — Investighează cu unelte low-level

Fără să dezinstalezi nimic, răspunde:
1. Cărui pachet îi aparține fișierul `/bin/bash`?
2. Ce versiune de `curl` e instalată și din ce repository provine (dacă e disponibil mai mult de unul)?

<details>
<summary>✅ Soluție</summary>

```bash
dpkg -S /bin/bash

apt-cache policy curl
```
</details>

---

## ✅ Checklist final

- [ ] Pot compila un proiect C clasic din sursă (`./configure && make && make install`) cu module custom
- [ ] Știu să citesc erori de `./configure` și să identific pachetul `-dev` lipsă
- [ ] Pot verifica un binar compilat manual cu `ldd` și îi înțeleg dependențele
- [ ] Pot construi și instala un pachet `.deb` minimal, de la zero
- [ ] Știu să investighez proveniența unui fișier/pachet cu `dpkg -S` / `apt-cache policy`
