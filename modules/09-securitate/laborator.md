# 🧪 Laborator — Modul 9: Securitate pe Linux

[← Înapoi la Modul 9](README.md) · [Cuprins curs](../../README.md)

## Scenariu

Ai primit un server Ubuntu proaspăt instalat, care va fi expus pe internet. Înainte să pui orice aplicație pe el, trebuie să-l securizezi: SSH doar prin chei, firewall strict, protecție împotriva brute-force, auditare a fișierelor critice, și o scanare care confirmă că nu ai lăsat nimic evident deschis.

⚠️ **Rulează acest laborator pe o VM de test, nu pe o mașină de producție** — schimbările la SSH pot bloca accesul dacă sunt greșite; ține la îndemână consola VM-ului (nu doar SSH) ca plasă de siguranță.

## Cerințe prealabile
- VM Ubuntu (sau Debian) cu drepturi sudo
- Acces la consola VM-ului (nu doar SSH), pentru cazul în care te blochezi afară

---

## Task 1 — SSH hardening

Aplică următoarele în `/etc/ssh/sshd_config`: `PermitRootLogin no`, `PasswordAuthentication no`, `MaxAuthTries 3`. **Înainte** de restart, asigură-te că ai deja o cheie SSH funcțională configurată (altfel te blochezi afară — vezi laboratorul Modulului 4, Task 3).

<details>
<summary>✅ Soluție</summary>

```bash
# confirmă ÎNTÂI că autentificarea prin cheie funcționează
ssh -o PasswordAuthentication=no localhost "echo ok"

sudo sed -i \
    -e 's/^#\?PermitRootLogin.*/PermitRootLogin no/' \
    -e 's/^#\?PasswordAuthentication.*/PasswordAuthentication no/' \
    -e 's/^#\?MaxAuthTries.*/MaxAuthTries 3/' \
    /etc/ssh/sshd_config

sudo sshd -t              # validează sintaxa ÎNAINTE de restart
sudo systemctl restart sshd

sshd -T | grep -E "permitrootlogin|passwordauthentication|maxauthtries"    # confirmă efectiv aplicat
```
</details>

---

## Task 2 — fail2ban

Instalează `fail2ban`, activează jail-ul pentru SSH cu prag de 3 încercări/10 minute și ban de 1 oră. Provoacă deliberat câteva autentificări eșuate și confirmă că IP-ul e banat.

<details>
<summary>✅ Soluție</summary>

```bash
sudo apt install -y fail2ban

sudo tee /etc/fail2ban/jail.local > /dev/null <<'EOF'
[sshd]
enabled = true
maxretry = 3
findtime = 600
bantime = 3600
EOF

sudo systemctl restart fail2ban
sudo fail2ban-client status sshd

# provoacă eșecuri (de pe altă mașină/IP, sau localhost cu user inexistent)
for i in 1 2 3 4; do ssh -o PreferredAuthentications=password baduser@localhost 2>/dev/null; done

sudo fail2ban-client status sshd     # ar trebui să arate IP-ul în "Banned IP list"

# deblocare manuală (util pentru a nu te bloca singur în timpul testelor!)
sudo fail2ban-client set sshd unbanip 127.0.0.1
```
</details>

---

## Task 3 — Firewall deny-by-default cu logging

Configurează `iptables` cu politică `DROP` implicită pe `INPUT`, permite doar SSH/HTTP/HTTPS și trafic pe loopback + conexiuni deja stabilite, și loghează tot ce e refuzat.

<details>
<summary>✅ Soluție</summary>

```bash
sudo iptables -F INPUT
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
sudo iptables -A INPUT -j LOG --log-prefix "IPTABLES-DROP: " --log-level 4
sudo iptables -P INPUT DROP

sudo iptables -L -n -v

# testează un port neautorizat, apoi verifică logul
nc -zv localhost 9999 2>&1 || true
sudo dmesg | grep "IPTABLES-DROP" | tail -3
```
</details>

---

## Task 4 — AppArmor

Verifică profilurile AppArmor active. Alege unul (sau `nginx` dacă e instalat) și pune-l temporar în modul `complain`, generează niște activitate, și verifică ce ar fi fost blocat în modul `enforce`.

<details>
<summary>✅ Soluție</summary>

```bash
sudo aa-status

# dacă ai nginx instalat:
sudo aa-complain /etc/apparmor.d/usr.sbin.nginx
sudo systemctl restart nginx
curl -s localhost >/dev/null

sudo journalctl -k | grep -i apparmor | tail -20     # evenimente ce AR FI fost blocate în enforce

sudo aa-enforce /etc/apparmor.d/usr.sbin.nginx        # revino la enforce după ce ai analizat
```
</details>

---

## Task 5 — Audit fișiere critice cu `auditd`

Instalează `auditd`, adaugă o regulă care loghează orice scriere/schimbare de atribute pe `/etc/passwd`, apoi modifică fișierul (ex: adaugă un comentariu inofensiv) și caută evenimentul în audit log.

<details>
<summary>✅ Soluție</summary>

```bash
sudo apt install -y auditd audispd-plugins
sudo auditctl -w /etc/passwd -p wa -k passwd-changes

sudo cp /etc/passwd /etc/passwd.bak
echo "# test audit $(date)" | sudo tee -a /etc/passwd > /dev/null

sudo ausearch -k passwd-changes | tail -20

sudo cp /etc/passwd.bak /etc/passwd     # revert modificarea de test
```

Pentru persistență peste reboot, adaugă regula într-un fișier în `/etc/audit/rules.d/`.
</details>

---

## Task 6 — Scanare de vulnerabilități cu Lynis

Rulează un audit complet cu Lynis și identifică cel puțin 3 recomandări pe care le poți aplica imediat, pe baza a ce ai învățat în acest modul.

<details>
<summary>✅ Soluție</summary>

```bash
sudo apt install -y lynis
sudo lynis audit system

# raportul complet e salvat și în:
sudo cat /var/log/lynis-report.dat | grep "^suggestion" | head -10
```

Recomandările tipice pentru o VM proaspătă includ: setare `umask` mai restrictiv, dezactivare protocoale de rețea nefolosite, configurare politică de parole, restricționare acces la `/boot`.
</details>

---

## Task 7 — Baseline SUID/SGID

Creează un baseline al tuturor fișierelor SUID din sistem. Simulează apariția unui binar SUID nou (suspect), apoi detectează-l prin comparație cu baseline-ul.

<details>
<summary>✅ Soluție</summary>

```bash
sudo find / -xdev -perm -4000 -type f 2>/dev/null | sort > ~/suid-baseline.txt

# simulează un binar SUID nou-apărut (ex: cineva a copiat /bin/bash cu SUID — clasic semn de compromitere)
sudo cp /bin/bash /tmp/suspicious-shell
sudo chmod 4755 /tmp/suspicious-shell

sudo find / -xdev -perm -4000 -type f 2>/dev/null | sort > ~/suid-current.txt
diff ~/suid-baseline.txt ~/suid-current.txt     # ar trebui să arate /tmp/suspicious-shell ca nou

sudo rm /tmp/suspicious-shell     # curățenie
```
</details>

---

## Task 8 — Kernel hardening persistent

Aplică minim 3 parametri `sysctl` de securitate din §9.9, persistent, într-un fișier dedicat.

<details>
<summary>✅ Soluție</summary>

```bash
sudo tee /etc/sysctl.d/99-hardening.conf > /dev/null <<'EOF'
net.ipv4.conf.all.accept_source_route = 0
net.ipv4.conf.all.accept_redirects = 0
net.ipv4.icmp_echo_ignore_broadcasts = 1
net.ipv4.tcp_syncookies = 1
kernel.dmesg_restrict = 1
EOF

sudo sysctl -p /etc/sysctl.d/99-hardening.conf
sysctl net.ipv4.tcp_syncookies      # confirmă că valoarea s-a aplicat
```
</details>

---

## ✅ Checklist final

- [ ] SSH acceptă doar chei, root login e dezactivat, și am testat ÎNAINTE de a restrânge accesul
- [ ] `fail2ban` banează automat IP-uri cu brute-force pe SSH
- [ ] Firewall-ul e deny-by-default, cu excepții explicite și logging pentru refuzuri
- [ ] Știu să pun un profil AppArmor în `complain` pentru debugging, fără să dezactivez protecția complet
- [ ] Pot audita modificări pe un fișier critic cu `auditd`/`ausearch`
- [ ] Am rulat un scan Lynis și am aplicat cel puțin 3 recomandări
- [ ] Pot detecta un binar SUID nou-apărut prin comparație cu un baseline
- [ ] Am parametri `sysctl` de hardening persistenți, nu doar aplicați live
