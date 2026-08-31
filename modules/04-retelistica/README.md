# 📦 Modul 4 — Rețelistică pe Linux

> **Nivel:** Mediu–Avansat | **Durată estimată:** 5h

[← Înapoi la cuprins](../../README.md)

## Obiective
- Configurarea și diagnosticarea rețelei
- Securizarea conexiunilor

---

## 4.1 — Modelul OSI & TCP/IP recap

| Layer OSI | Exemplu | Unelte Linux relevante |
|-----------|---------|--------------------------|
| 7 — Application | HTTP, DNS, SSH | `curl`, `dig`, `ssh` |
| 4 — Transport | TCP, UDP | `ss`, porturi |
| 3 — Network | IP, routare | `ip`, `ping`, `traceroute` |
| 2 — Data Link | Ethernet, MAC | `ip link`, `arp` |
| 1 — Physical | cabluri, semnal | (hardware) |

Modelul **TCP/IP** (practic, cel folosit real) comprimă asta în 4 straturi: Link → Internet (IP) → Transport (TCP/UDP) → Application. Când diagnostichezi o problemă de rețea, mergi de jos în sus: interfața are IP? poți ajunge la gateway? poți rezolva DNS? portul răspunde?

---

## 4.2 — Configurare interfețe de rețea

```bash
ip addr show                    # (sau: ip a) — IP-uri pe toate interfețele
ip addr add 192.168.1.10/24 dev eth0     # adaugă un IP (temporar, se pierde la reboot)
ip link show                       # stare interfețe (UP/DOWN), MAC
ip link set eth0 up                  # activează interfața
ip route show                          # tabela de rutare
ip route add default via 192.168.1.1      # setează gateway-ul implicit
```

- **`nmcli`** — control NetworkManager (majoritatea desktop-urilor și multe servere moderne):
  ```bash
  nmcli device status
  nmcli connection show
  nmcli connection add type ethernet ifname eth0 ipv4.addresses 192.168.1.10/24 ipv4.gateway 192.168.1.1 ipv4.method manual
  ```
- **`netplan`** (Ubuntu Server) — configurare declarativă în YAML, `/etc/netplan/*.yaml`:
  ```yaml
  network:
    version: 2
    ethernets:
      eth0:
        addresses: [192.168.1.10/24]
        routes:
          - to: default
            via: 192.168.1.1
        nameservers:
          addresses: [8.8.8.8, 1.1.1.1]
  ```
  ```bash
  netplan apply
  ```

---

## 4.3 — Diagnosticare rețea

```bash
ping -c4 8.8.8.8              # conectivitate la nivel IP (4 pachete, apoi se oprește)
traceroute google.com           # ruta (hop-uri) până la destinație
mtr google.com                   # traceroute + ping continuu, combinate, live

dig google.com                    # interogare DNS detaliată
dig +short google.com              # doar IP-ul rezultat
dig @8.8.8.8 google.com               # interoghează un server DNS specific
nslookup google.com                    # variantă mai veche, interactivă
host google.com                          # variantă simplă, rapidă
```

**Ordinea firească de diagnostic** pentru "nu pot accesa serviciul X":
1. `ip addr` — am IP?
2. `ping <gateway>` — văd rețeaua locală?
3. `ping 8.8.8.8` — am ieșire la internet (IP direct, fără DNS)?
4. `dig google.com` — funcționează rezoluția DNS?
5. `curl -v http://serviciu:port` — răspunde aplicația la nivel de layer 7?

---

## 4.4 — Analiza traficului

```bash
ss -tulpn                    # socket-uri TCP/UDP, listening, cu proces (necesită root pentru -p complet)
#   -t TCP  -u UDP  -l listening  -p proces  -n fără rezolvare DNS/nume servicii

netstat -tulpn                 # echivalent mai vechi al lui ss (unele distro nu-l mai includ implicit)

tcpdump -i eth0                  # captură live pe interfața eth0
tcpdump -i eth0 port 443            # doar trafic HTTPS
tcpdump -i eth0 -w capture.pcap       # salvează într-un fișier, analizabil ulterior cu Wireshark
tcpdump -i eth0 host 10.0.0.5           # doar trafic de la/către un IP specific
```

**Wireshark** — versiunea grafică/interactivă a lui `tcpdump`: deschide fișiere `.pcap`, filtre vizuale (`http`, `tcp.port==443`), urmărire de stream TCP complet.

---

## 4.5 — SSH avansat

```bash
ssh-keygen -t ed25519 -C "adrian@laptop"     # generează o pereche de chei (ed25519 = recomandat azi)
ssh-copy-id user@server                         # copiază cheia publică pe server, în ~/.ssh/authorized_keys
ssh user@server                                   # autentificare prin cheie (dacă e configurată corect)
```

**`~/.ssh/config`** — profiluri per-host, evită să repeți flag-uri:
```
Host prod
    HostName 203.0.113.10
    User deploy
    Port 2222
    IdentityFile ~/.ssh/id_ed25519_prod

Host jump
    HostName bastion.example.com
    User adrian

Host internal-db
    HostName 10.0.0.5
    User admin
    ProxyJump jump          # ← trece prin "jump" ca bastion host
```
Cu acest config, `ssh internal-db` se conectează automat prin bastion, fără comenzi manuale suplimentare.

**Tunneling SSH:**
```bash
# Local port forwarding: portul 8080 local → portul 80 de pe server, prin tunel SSH
ssh -L 8080:localhost:80 user@server

# Remote port forwarding: expune un port local PE server
ssh -R 9000:localhost:3000 user@server

# Dynamic forwarding (SOCKS proxy) — tot traficul prin tunel, ca un mini-VPN
ssh -D 1080 user@server
```

**Jump hosts** (fără să fie deja în config): `ssh -J bastion user@internal-server`.

---

## 4.6 — SCP & RSYNC

```bash
scp fisier.txt user@server:/tmp/                  # copiere simplă peste SSH
scp -r dir/ user@server:/tmp/                        # recursiv
scp -P 2222 fisier.txt user@server:/tmp/               # port custom (atenție: -P majusculă la scp, -p la ssh)

rsync -avz dir/ user@server:/backup/dir/                 # -a arhivare (permisiuni+timestamp), -v verbose, -z compresie
rsync -avz --delete dir/ user@server:/backup/dir/          # sincronizare EXACTĂ — șterge la destinație ce nu mai există la sursă
rsync -avzn dir/ user@server:/backup/dir/                    # -n = dry-run, arată ce s-ar face fără să facă nimic
rsync -avz -e "ssh -p 2222" dir/ user@server:/backup/            # port SSH custom
```

`rsync` transferă doar diferențele (delta), mult mai eficient decât `scp` pentru sincronizări repetate.

---

## 4.7 — Firewall cu iptables/nftables

**iptables** — lucrează cu **lanțuri** (chains): `INPUT` (trafic către mașină), `OUTPUT` (trafic dinspre mașină), `FORWARD` (trafic rutat prin mașină).

```bash
iptables -L -n -v                      # listează regulile curente, cu contoare

iptables -A INPUT -p tcp --dport 22 -j ACCEPT     # permite SSH
iptables -A INPUT -p tcp --dport 80 -j ACCEPT       # permite HTTP
iptables -A INPUT -i lo -j ACCEPT                     # permite tot pe loopback
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT   # permite răspunsuri la conexiuni deja inițiate
iptables -P INPUT DROP                                   # politică implicită: refuză tot restul

iptables -A INPUT -j LOG --log-prefix "IPTABLES-DROP: "     # logging înainte de DROP final (pentru audit)

iptables-save > /etc/iptables/rules.v4     # persistă regulile (variază după distro)
```

**nftables** — succesorul modern, sintaxă unificată IPv4/IPv6:
```bash
nft list ruleset
nft add table inet filter
nft add chain inet filter input { type filter hook input priority 0 \; policy drop \; }
nft add rule inet filter input tcp dport 22 accept
nft add rule inet filter input ct state established,related accept
```

**NAT** (ex: masquerading pentru a partaja o conexiune internet):
```bash
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
```

---

## 4.8 — UFW & firewalld

Interfețe simplificate peste iptables/nftables — pentru cazuri obișnuite, nu mai scrii reguli low-level.

```bash
# UFW (Ubuntu/Debian)
ufw allow 22/tcp
ufw allow from 192.168.1.0/24 to any port 5432    # restricționează accesul la DB doar din LAN
ufw default deny incoming
ufw default allow outgoing
ufw enable
ufw status verbose

# firewalld (RHEL/CentOS/Fedora)
firewall-cmd --add-service=http --permanent
firewall-cmd --add-port=8080/tcp --permanent
firewall-cmd --reload
firewall-cmd --list-all
```

---

## 4.9 — VPN basics

- **OpenVPN** — matur, foarte flexibil, folosește TLS pentru autentificare/criptare. Config text (`.ovpn`), certificate per client.
- **WireGuard** — modern, mult mai simplu de configurat, performanță mai bună, bazat pe chei publice (similar SSH):
  ```bash
  wg genkey | tee privatekey | wg pubkey > publickey     # generare pereche de chei

  # /etc/wireguard/wg0.conf (server)
  # [Interface]
  # Address = 10.8.0.1/24
  # PrivateKey = <cheia-privata-server>
  # ListenPort = 51820
  #
  # [Peer]
  # PublicKey = <cheia-publica-client>
  # AllowedIPs = 10.8.0.2/32

  wg-quick up wg0
  ```

---

## 🧪 Laborator practic

[👉 Laborator Modul 4](laborator.md) — configurează accesul de rețea securizat pentru un server nou: IP static, SSH doar prin chei, tunel, și firewall restrictiv.

## Exerciții practice
- Configurează SSH cu autentificare prin cheie și dezactivează parola
- Creează reguli de firewall pentru un server web
