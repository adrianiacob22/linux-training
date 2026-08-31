# 📦 Modul 9 — Securitate pe Linux

> **Nivel:** Avansat | **Durată estimată:** 5h

[← Înapoi la cuprins](../../README.md)

## Obiective
- Hardening sistem Linux
- Detecția și răspunsul la incidente

---

## 9.1 — Principiul privilegiului minim

Un cont, proces sau serviciu trebuie să aibă **doar** drepturile minime necesare funcției sale — nimic în plus, "pentru orice eventualitate".

Aplicat concret în practică:
- servicii rulează sub conturi dedicate, nu root (§8.9)
- `sudo` granular, nu acces total (§5.4)
- SUID doar unde e absolut necesar (§2.3), auditat periodic
- firewall deny-by-default, excepții explicite (§4.7)
- chei SSH cu scop limitat (`command=` forțat în `authorized_keys`) pentru automatizări

---

## 9.2 — SSH Hardening

```
# /etc/ssh/sshd_config
PermitRootLogin no                # root nu se poate loga direct prin SSH — folosește sudo după login normal
PasswordAuthentication no            # doar chei, nicio parolă acceptată
Port 2222                              # schimbă portul implicit (reduce zgomotul de scanări automate, NU e securitate reală)
MaxAuthTries 3                           # limitează încercările per conexiune
AllowUsers alice bob deploy                # whitelist explicit de utilizatori permiși prin SSH
ClientAliveInterval 300                      # deconectează sesiuni inactive
X11Forwarding no                               # dezactivează dacă nu ai nevoie de GUI remote
```
```bash
sshd -t                    # ALWAYS validează sintaxa înainte de restart
systemctl restart sshd
```

**fail2ban** — banează automat IP-uri cu prea multe încercări eșuate de autentificare:
```bash
apt install fail2ban
# /etc/fail2ban/jail.local
# [sshd]
# enabled = true
# maxretry = 3
# bantime = 3600
# findtime = 600

fail2ban-client status sshd            # vezi IP-urile banate curent
fail2ban-client set sshd unbanip 1.2.3.4    # deblochează manual un IP
```

**2FA (TOTP)** — via `libpam-google-authenticator`:
```bash
apt install libpam-google-authenticator
google-authenticator     # generează secretul, per utilizator, interactiv
```
```
# /etc/pam.d/sshd — adaugă
auth required pam_google_authenticator.so
```
```
# /etc/ssh/sshd_config
KbdInteractiveAuthentication yes
AuthenticationMethods publickey,keyboard-interactive
```

---

## 9.3 — Firewall avansat

Politică minimă recomandată pentru un server public: deny-by-default pe `INPUT`, permite doar strict ce e nevoie, loghează ce e refuzat (pentru analiză ulterioară de incidente).

```bash
iptables -P INPUT DROP
iptables -P FORWARD DROP
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --set                       # rate limiting SSH
iptables -A INPUT -p tcp --dport 22 -m state --state NEW -m recent --update --seconds 60 --hitcount 4 -j DROP   # max 4 conexiuni noi/min
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -j LOG --log-prefix "DROPPED: " --log-level 4
iptables -A INPUT -j DROP
```

Vezi §4.7 pentru sintaxa completă iptables/nftables.

---

## 9.4 — SELinux / AppArmor

Ambele adaugă **Mandatory Access Control (MAC)** peste permisiunile Unix clasice — chiar dacă un proces are permisiuni rwx pe un fișier, politica MAC îl poate bloca oricum.

**SELinux** (RHEL/CentOS/Fedora):
```bash
getenforce                  # Enforcing / Permissive / Disabled
setenforce 0                  # comută temporar în Permissive (nu blochează, doar loghează) — util pt. debugging
sestatus                        # detalii complete

ls -Z /var/www/html               # context SELinux al fișierelor (user:role:type:level)
chcon -t httpd_sys_content_t /var/www/html/index.html    # schimbă contextul (temporar, se pierde la relabel)
semanage fcontext -a -t httpd_sys_content_t "/srv/web(/.*)?"    # regulă PERSISTENTĂ
restorecon -Rv /srv/web

audit2allow -a                     # sugerează reguli noi pe baza refuzurilor recente din audit log
```

**AppArmor** (Ubuntu/Debian) — profiluri per-aplicație, mai simplu conceptual decât SELinux:
```bash
aa-status                        # profiluri active, enforce/complain
aa-complain /etc/apparmor.d/usr.sbin.nginx     # mod "complain" — loghează, nu blochează (pt. debugging)
aa-enforce /etc/apparmor.d/usr.sbin.nginx        # mod "enforce" — blochează activ
```

> Regulă de aur: **nu dezactiva SELinux/AppArmor** ca să "rezolvi" o eroare — pune-l în modul permisiv/complain, identifică regula lipsă din loguri, adaug-o, revino la enforcing.

---

## 9.5 — Auditare sistem cu `auditd`

`auditd` loghează la nivel de kernel evenimente de securitate: accesări de fișiere, apeluri de sistem, autentificări.

```bash
auditctl -w /etc/passwd -p wa -k passwd-changes     # loghează Write+Attribute-change pe /etc/passwd
auditctl -w /etc/shadow -p wa -k shadow-changes
auditctl -a always,exit -F arch=b64 -S execve -k exec-log     # loghează toate execuțiile de programe

ausearch -k passwd-changes                # caută evenimente cu tag-ul "passwd-changes"
ausearch -ua 1001                           # evenimente asociate unui UID specific
aureport --summary                            # sumar general de activitate

# Reguli persistente: /etc/audit/rules.d/audit.rules
```

---

## 9.6 — Scanare vulnerabilități

```bash
lynis audit system              # audit general de securitate, scor + recomandări detaliate
rkhunter --check                  # caută rootkit-uri cunoscute, comparat cu baseline
rkhunter --propupd                  # actualizează baseline-ul după modificări legitime de sistem
chkrootkit                            # alternativă la rkhunter, verificări similare

nmap -sV localhost                       # ce porturi/servicii sunt expuse, cu versiuni (rulează din exterior, nu doar local!)
nmap -sV -p- 192.168.1.10                  # scanare completă (toate 65535 porturi) a unei ținte
```

⚠️ Rulează `nmap` doar pe sisteme pe care ai autorizație explicită să le scanezi.

---

## 9.7 — Criptare

**LUKS** — criptare de disk/partiție completă:
```bash
cryptsetup luksFormat /dev/sdb1        # ⚠️ distructiv — șterge tot ce era pe partiție
cryptsetup luksOpen /dev/sdb1 secure_data      # deschide, mapat la /dev/mapper/secure_data
mkfs.ext4 /dev/mapper/secure_data
mount /dev/mapper/secure_data /mnt/secure

cryptsetup luksClose secure_data
```

**GPG** — criptare/semnare fișiere individuale:
```bash
gpg --gen-key                              # generează o pereche de chei
gpg --encrypt --recipient adrian@example.com fisier.txt     # criptează pentru un destinatar
gpg --decrypt fisier.txt.gpg > fisier.txt                     # decriptează
gpg --symmetric fisier.txt                                       # criptare simplă cu parolă (fără chei)
gpg --sign fisier.txt                                               # semnătură digitală (integritate + autenticitate)
```

**SSL/TLS** — pentru servicii web, folosește Let's Encrypt / `certbot` (automatizat, gratuit):
```bash
certbot --nginx -d exemplu.com          # obține + configurează automat certificatul pentru Nginx
certbot renew --dry-run                    # testează reînnoirea automată
```

---

## 9.8 — Umask, SUID/SGID audit

**`umask`** — determină permisiunile implicite ale fișierelor/directoarelor nou create (se scade din 666/777):
```bash
umask                # valoarea curentă, ex: 0022
umask 0027              # fișiere noi: 640, directoare noi: 750 — mai restrictiv, potrivit pentru servere
```

**Audit periodic SUID/SGID** — orice binar SUID nou-apărut poate fi un semn de compromitere:
```bash
find / -perm -4000 -type f 2>/dev/null > /tmp/suid-current.txt      # toate fișierele SUID
find / -perm -2000 -type f 2>/dev/null > /tmp/sgid-current.txt        # toate fișierele SGID
diff /tmp/suid-baseline.txt /tmp/suid-current.txt                        # compară cu un baseline salvat anterior
```

---

## 9.9 — Kernel hardening

```bash
sysctl -a | grep ip_forward              # vezi valoarea curentă a unui parametru
sysctl -w net.ipv4.ip_forward=0             # setare LIVE (se pierde la reboot dacă nu e persistată)
```

Persistent, în `/etc/sysctl.d/99-hardening.conf`:
```ini
net.ipv4.ip_forward = 0                   # nu ruta pachete (dezactivează dacă mașina nu e router)
net.ipv4.conf.all.accept_source_route = 0    # refuză source routing (previne spoofing de rută)
net.ipv4.conf.all.accept_redirects = 0         # ignoră ICMP redirects (previne atacuri MITM la nivel de rutare)
net.ipv4.icmp_echo_ignore_broadcasts = 1         # previne participarea la Smurf attacks
net.ipv4.tcp_syncookies = 1                        # mitigare SYN flood
kernel.dmesg_restrict = 1                            # restricționează citirea dmesg la root (previne info leak)
fs.suid_dumpable = 0                                   # nu genera core dumps pentru binare SUID (previne leak de memorie sensibilă)
```
```bash
sysctl -p /etc/sysctl.d/99-hardening.conf      # aplică fișierul imediat
```

---

## 🧪 Laborator practic

[👉 Laborator Modul 9](laborator.md) — hardening complet, pas cu pas, al unui server Ubuntu proaspăt instalat: SSH, firewall, AppArmor, audit, scanare de vulnerabilități.

## Exerciții practice
- Hardening complet al unui server Ubuntu fresh install
- Configurare SELinux pentru o aplicație web
