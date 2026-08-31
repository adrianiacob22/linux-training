# 🧪 Laborator — Modul 8: Servicii & Systemd

[← Înapoi la Modul 8](README.md) · [Cuprins curs](../../README.md)

## Scenariu

Echipa de dezvoltare ți-a dat o mică aplicație (un script care rulează continuu) și îți cere să o transformi într-un serviciu de sistem: pornește automat la boot, se repornește singură dacă pică, și are logging corect. În plus, un cron job vechi de curățenie trebuie migrat la un systemd timer modern.

## Cerințe prealabile
- O mașină/VM Linux cu systemd și drepturi root/sudo

---

## Task 1 — Aplicația și utilizatorul dedicat

Creează o aplicație minimală în `/opt/heartbeat/heartbeat.sh` care scrie un timestamp la fiecare 5 secunde, și un utilizator de sistem dedicat (fără login interactiv) care o va rula.

<details>
<summary>✅ Soluție</summary>

```bash
sudo mkdir -p /opt/heartbeat
sudo tee /opt/heartbeat/heartbeat.sh > /dev/null <<'EOF'
#!/usr/bin/env bash
while true; do
    echo "heartbeat: $(date '+%Y-%m-%d %H:%M:%S')"
    sleep 5
done
EOF
sudo chmod +x /opt/heartbeat/heartbeat.sh

sudo useradd -r -s /usr/sbin/nologin heartbeat 2>/dev/null || true
sudo chown -R heartbeat:heartbeat /opt/heartbeat
```
</details>

---

## Task 2 — Unit file și activare

Creează `/etc/systemd/system/heartbeat.service`: rulează ca userul `heartbeat`, tip `simple`, cu `Restart=on-failure`. Activează-l să pornească la boot și pornește-l acum.

<details>
<summary>✅ Soluție</summary>

```bash
sudo tee /etc/systemd/system/heartbeat.service > /dev/null <<'EOF'
[Unit]
Description=Heartbeat demo service
After=network.target

[Service]
Type=simple
User=heartbeat
ExecStart=/opt/heartbeat/heartbeat.sh
Restart=on-failure
RestartSec=3

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now heartbeat

systemctl status heartbeat
```
</details>

---

## Task 3 — Verifică logging-ul

Urmărește logurile serviciului live timp de câteva cicluri, apoi filtrează doar ultimele 2 minute de log.

<details>
<summary>✅ Soluție</summary>

```bash
journalctl -u heartbeat -f      # Ctrl+C pentru a ieși după câteva rânduri

journalctl -u heartbeat --since "2 minutes ago"
```
</details>

---

## Task 4 — Testează recovery-ul automat

"Omoară" procesul serviciului direct (simulează un crash) și confirmă că systemd îl repornește singur, conform `Restart=on-failure`.

<details>
<summary>💡 Hint</summary>

Găsește PID-ul din `systemctl status heartbeat`, apoi `kill -9` pe el. Un `kill` normal (SIGTERM) ar putea fi tratat ca oprire "curată", în funcție de cum răspunde scriptul — pentru un test clar de crash, folosește `-9`.
</details>

<details>
<summary>✅ Soluție</summary>

```bash
pid=$(systemctl show -p MainPID --value heartbeat)
echo "PID curent: $pid"

sudo kill -9 "$pid"

sleep 5
systemctl status heartbeat     # ar trebui să arate un PID NOU, serviciul repornit automat
journalctl -u heartbeat --since "1 minute ago" | tail -20
```
</details>

---

## Task 5 — Capcana `daemon-reload`

Editează unit file-ul (schimbă `RestartSec=3` în `RestartSec=10`), dar **nu** rula `daemon-reload`. Repornește serviciul cu `systemctl restart` și observă că systemd încă folosește configurația veche. Apoi repară corect.

<details>
<summary>✅ Soluție</summary>

```bash
sudo sed -i 's/RestartSec=3/RestartSec=10/' /etc/systemd/system/heartbeat.service

sudo systemctl restart heartbeat     # nu ridică eroare, dar...
systemctl show -p RestartUSec --value heartbeat     # arată încă valoarea veche (systemd nu a recitit fișierul)

sudo systemctl daemon-reload            # ACUM recitește unit file-ul de pe disc
sudo systemctl restart heartbeat
systemctl show -p RestartUSec --value heartbeat     # acum arată 10s
```

> Concluzie de reținut: `daemon-reload` e obligatoriu după orice editare manuală a unui fișier `.service`/`.timer` — `restart` singur NU recitește fișierul.
</details>

---

## Task 6 — Migrează un cron job la un systemd timer

Ai (sau simulezi) un cron job vechi: `0 2 * * * find /tmp -name "*.tmp" -mtime +1 -delete` (curăță fișiere `.tmp` mai vechi de o zi, zilnic la 02:00). Recreează-l ca pereche `.timer` + `.service`.

<details>
<summary>✅ Soluție</summary>

```bash
sudo tee /etc/systemd/system/tmp-cleanup.service > /dev/null <<'EOF'
[Unit]
Description=Curăță fișiere .tmp vechi din /tmp

[Service]
Type=oneshot
ExecStart=/usr/bin/find /tmp -name "*.tmp" -mtime +1 -delete
EOF

sudo tee /etc/systemd/system/tmp-cleanup.timer > /dev/null <<'EOF'
[Unit]
Description=Rulează tmp-cleanup.service zilnic la 02:00

[Timer]
OnCalendar=*-*-* 02:00:00
Persistent=true

[Install]
WantedBy=timers.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable --now tmp-cleanup.timer

systemctl list-timers | grep tmp-cleanup

# testează manual execuția serviciului, fără să aștepți ora 02:00:
sudo systemctl start tmp-cleanup.service
journalctl -u tmp-cleanup.service --since "5 minutes ago"
```
</details>

---

## Task 7 — Curățenie

Oprește și dezactivează ambele servicii create, șterge unit files-urile, și rulează `daemon-reload` final.

<details>
<summary>✅ Soluție</summary>

```bash
sudo systemctl disable --now heartbeat
sudo systemctl disable --now tmp-cleanup.timer

sudo rm /etc/systemd/system/heartbeat.service
sudo rm /etc/systemd/system/tmp-cleanup.service /etc/systemd/system/tmp-cleanup.timer

sudo systemctl daemon-reload
sudo systemctl reset-failed
```
</details>

---

## ✅ Checklist final

- [ ] Pot scrie un unit file `.service` complet, cu user dedicat și `Restart=`
- [ ] Știu să confirm recovery-ul automat după un crash simulat
- [ ] Înțeleg (din experiență, nu doar din teorie) de ce `daemon-reload` e obligatoriu
- [ ] Pot filtra `journalctl` după unit, timp și severitate
- [ ] Pot migra un cron job la o pereche `.timer`/`.service`, inclusiv testare manuală fără să aștept programarea
