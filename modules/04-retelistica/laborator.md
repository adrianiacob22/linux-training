# 🧪 Laborator — Modul 4: Rețelistică pe Linux

[← Înapoi la Modul 4](README.md) · [Cuprins curs](../../README.md)

## Scenariu

Ai primit un server nou, proaspăt provizionat, pe care urmează să găzduiești o aplicație web. Trebuie să-i configurezi rețeaua, să securizezi accesul SSH (fără parole, doar chei), să transferi codul aplicației, și să pui un firewall care lasă trecerea doar la ce e strict necesar.

## Cerințe prealabile
- Ideal: **două** mașini/VM-uri Linux (sau două containere Docker cu SSH instalat) — una joacă rol de "server", una de "client/admin"
- Dacă ai o singură mașină, majoritatea task-urilor merg simulate pe `localhost` (SSH către `127.0.0.1`)
- Drepturi root/sudo pe server pentru task-urile de firewall

---

## Task 1 — Verifică și configurează IP-ul

Pe server, afișează interfețele de rețea și IP-urile curente. Dacă interfața principală nu are IP static, adaugă unul temporar (nu contează dacă se pierde la reboot, e doar exercițiu).

<details>
<summary>✅ Soluție</summary>

```bash
ip addr show
ip route show     # confirmă gateway-ul implicit

# exemplu — adaptează la interfața ta reală (vezi output-ul de mai sus)
sudo ip addr add 192.168.100.10/24 dev eth0
sudo ip link set eth0 up
```
</details>

---

## Task 2 — Lanțul de diagnostic

De pe client, urmează în ordine pașii de diagnostic din §4.3 pentru a confirma conectivitatea completă către server: rezoluție DNS (dacă folosești un hostname), conectivitate IP, și că portul SSH (22) răspunde.

<details>
<summary>✅ Soluție</summary>

```bash
ping -c3 <ip-server>
dig +short <hostname-server>     # doar dacă folosești DNS, sare peste dacă ai IP direct
ss -tulpn | grep :22               # rulat PE server — confirmă că sshd ascultă
nc -zv <ip-server> 22                 # rulat de pe client — confirmă că portul e accesibil (netcat)
# sau, fără netcat:
timeout 3 bash -c "echo > /dev/tcp/<ip-server>/22" && echo "port deschis"
```
</details>

---

## Task 3 — SSH doar prin chei, fără parolă

De pe client, generează o pereche de chei SSH dedicată acestui server, copiaz-o pe server, apoi editează configurația SSH a serverului pentru a **dezactiva** autentificarea prin parolă. Testează că autentificarea prin cheie funcționează **înainte** de a dezactiva parola (ca să nu te blochezi afară!).

<details>
<summary>💡 Hint</summary>

Ordinea contează: testează cheia funcțională întâi, abia apoi editează `/etc/ssh/sshd_config` și dă restart la `sshd`. Ține un al doilea terminal conectat, dacă poți, cât timp faci schimbarea — plasă de siguranță în caz că ceva nu merge.
</details>

<details>
<summary>✅ Soluție</summary>

```bash
# pe client
ssh-keygen -t ed25519 -C "lab-server" -f ~/.ssh/id_ed25519_lab
ssh-copy-id -i ~/.ssh/id_ed25519_lab.pub user@<ip-server>

# testează ÎNAINTE de a dezactiva parola
ssh -i ~/.ssh/id_ed25519_lab user@<ip-server> "echo conectat cu succes"
```

```bash
# pe server
sudo sed -i 's/^#\?PasswordAuthentication.*/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo sshd -t              # verifică sintaxa înainte de restart — foarte important!
sudo systemctl restart sshd
```

```bash
# pe client — confirmă că parola nu mai e acceptată (dintr-o mașină fără cheia configurată, sau cu -o):
ssh -o PubkeyAuthentication=no user@<ip-server>
# ar trebui să eșueze / să nu ofere prompt de parolă
```
</details>

---

## Task 4 — Profil SSH persistent

Creează o intrare în `~/.ssh/config` (pe client) numită `labserver` care folosește automat cheia și userul corecte, astfel încât să te poți conecta doar cu `ssh labserver`.

<details>
<summary>✅ Soluție</summary>

```bash
cat >> ~/.ssh/config <<EOF

Host labserver
    HostName <ip-server>
    User user
    IdentityFile ~/.ssh/id_ed25519_lab
    IdentitiesOnly yes
EOF

chmod 600 ~/.ssh/config
ssh labserver "hostname"
```
</details>

---

## Task 5 — Deploy cu rsync

Simulează deployul aplicației: creează local un mic director `app/` cu 2-3 fișiere, apoi sincronizează-l pe server în `/tmp/app`, folosind profilul SSH de la Task 4. Rulează întâi un dry-run.

<details>
<summary>✅ Soluție</summary>

```bash
mkdir -p app && echo "<h1>Hello</h1>" > app/index.html && echo "v1.0" > app/VERSION

rsync -avzn app/ labserver:/tmp/app/     # dry-run — vezi ce s-ar copia, fără să copieze
rsync -avz app/ labserver:/tmp/app/       # transfer real

ssh labserver "ls -la /tmp/app"
```
</details>

---

## Task 6 — Firewall restrictiv

Pe server, configurează un firewall (UFW dacă e Debian/Ubuntu, sau `iptables` direct dacă preferi practica low-level) care:
- permite SSH (22), HTTP (80), HTTPS (443)
- refuză implicit orice alt trafic de intrare
- permite tot traficul de ieșire

<details>
<summary>✅ Soluție — UFW</summary>

```bash
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw enable
sudo ufw status verbose
```
</details>

<details>
<summary>✅ Soluție — iptables (echivalent, mai low-level)</summary>

```bash
sudo iptables -A INPUT -i lo -j ACCEPT
sudo iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
sudo iptables -P INPUT DROP

sudo iptables -L -n -v
```

⚠️ Testează regulile de firewall dintr-un al doilea terminal conectat deja prin SSH — dacă greșești portul SSH, te poți bloca afară din server.
</details>

---

## Task 7 — Confirmă cu captură de trafic

Pe server, pornește o captură scurtă `tcpdump` filtrată pe portul 22, apoi de pe client fă o nouă conexiune SSH și confirmă vizual, în captură, handshake-ul TCP + traficul SSH.

<details>
<summary>✅ Soluție</summary>

```bash
# pe server, într-un terminal
sudo tcpdump -i any port 22 -c 20 -n

# pe client, în alt terminal
ssh labserver "echo test"
```

Ar trebui să vezi în output pachetele SYN/SYN-ACK/ACK ale handshake-ului TCP, urmate de trafic criptat SSH (`Encrypted Packet`).
</details>

---

## ✅ Checklist final

- [ ] Pot configura și verifica un IP static și rutarea implicită
- [ ] Urmez sistematic lanțul de diagnostic (IP → DNS → port) în loc să ghicesc
- [ ] Pot activa autentificare SSH doar prin chei, în siguranță (fără să mă blochez afară)
- [ ] Am un `~/.ssh/config` funcțional cu profiluri per-host
- [ ] Pot sincroniza fișiere eficient cu `rsync` (inclusiv dry-run)
- [ ] Pot construi reguli de firewall restrictive (deny-by-default + excepții explicite)
- [ ] Știu să confirm comportamentul rețelei cu o captură `tcpdump`, nu doar "cred că merge"

### Provocare bonus
Adaugă o regulă de firewall care permite acces la portul 5432 (PostgreSQL) **doar** dintr-un subnet specific (ex: `10.0.0.0/24`), nu de oriunde.
