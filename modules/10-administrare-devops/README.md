# 📦 Modul 10 — Administrare Avansată & DevOps Intro

> **Nivel:** Avansat | **Durată estimată:** 5h

[← Înapoi la cuprins](../../README.md)

## Obiective
- Administrarea la scară largă
- Introducere în tooling-ul modern DevOps

---

## 10.1 — Performanță & Tuning

```bash
# Kernel parameters relevante pentru performanță (persistă în /etc/sysctl.d/*.conf)
sysctl -w vm.swappiness=10                # cât de "dornic" e kernel-ul să folosească swap (0-100, mai mic = preferă RAM)
sysctl -w net.core.somaxconn=1024           # coada maximă de conexiuni TCP în așteptare (relevant pt. servere sub load mare)
sysctl -w vm.dirty_ratio=15                   # % din RAM care poate fi "dirty" (nescris pe disc) înainte de flush forțat

# I/O schedulers — algoritmul de ordonare a cererilor de disk
cat /sys/block/sda/queue/scheduler          # vezi scheduler-ul curent (opțiunea între [] e activă)
echo "mq-deadline" > /sys/block/sda/queue/scheduler    # bun pt. SSD-uri, latență predictibilă
# "none"/"noop" adesea preferat pe NVMe (device-ul își face singur optimizarea)

# Network tuning
sysctl -w net.ipv4.tcp_congestion_control=bbr    # algoritm modern de control congestie (dacă kernel-ul suportă)
ethtool eth0                                        # viteză/duplex interfață de rețea
ethtool -k eth0                                       # offloading features (checksums, TSO, etc.)
```

**Principiu general:** nu tuna "din burtă" — măsoară întâi (§10.2), schimbă UN parametru, remăsoară, documentează. Tuning fără măsurare e doar noroc.

---

## 10.2 — Troubleshooting metodic

**USE Method** (Brendan Gregg) — pentru fiecare resursă (CPU, memorie, disk, rețea), verifică:
- **U**tilization — cât de ocupată e resursa (%)
- **S**aturation — cât de multă muncă așteaptă în coadă
- **E**rrors — evenimente de eroare

```bash
# CPU
mpstat -P ALL 1        # utilization per core
uptime                    # load average = proxy pentru saturation (procese care așteaptă CPU)

# Memorie
free -h                       # utilization
vmstat 1                        # coloana "si"/"so" (swap in/out) = saturation semnal

# Disk
iostat -x 1                        # %util = utilization, avgqu-sz = saturation, mărimea cozii
dmesg | grep -i "i/o error"           # errors

# Rețea
ip -s link show eth0                    # erori/drop-uri per interfață
ss -s                                     # sumar conexiuni, retransmisii TCP = semnal de saturare
```

**RED Method** (pentru servicii/microservicii, nu resurse hardware) — per serviciu:
- **R**ate — câte request-uri/secundă
- **E**rrors — câte eșuează
- **D**uration — cât durează (latență, percentile p50/p95/p99)

**Flux de troubleshooting recomandat:**
1. Definește simptomul exact ("API-ul răspunde lent" ≠ "userii se plâng")
2. USE pe hardware — e o resursă saturată?
3. RED pe serviciu — unde apare latența/eroarea exact?
4. Izolează cu unelte specifice (`strace`, `tcpdump`, loguri aplicație)
5. O singură schimbare odată, remăsoară, documentează

```bash
strace -c -p 1234        # ce syscalls face un proces, și cât timp petrece în fiecare (sumar)
strace -f -e trace=network -p 1234     # doar syscalls de rețea, inclusiv thread-uri copil
```

---

## 10.3 — Containere — Docker basics

```bash
docker run -d --name web -p 8080:80 nginx        # pornește un container, detached, port mapping
docker ps                                             # containere active
docker ps -a                                            # toate (inclusiv oprite)
docker logs -f web                                        # loguri, live
docker exec -it web bash                                    # shell interactiv într-un container rulând
docker stop web && docker rm web                               # oprire + ștergere

docker images                       # imagini locale
docker pull ubuntu:22.04              # descarcă o imagine
docker build -t myapp:1.0 .             # construiește o imagine din Dockerfile-ul din directorul curent
```

**Dockerfile minimal:**
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["python", "app.py"]
```

**Docker Compose** — orchestrare multi-container, declarativă:
```yaml
# docker-compose.yml
services:
  web:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - db
  db:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: secret
    volumes:
      - dbdata:/var/lib/postgresql/data
volumes:
  dbdata:
```
```bash
docker compose up -d
docker compose logs -f
docker compose down
```

---

## 10.4 — Ansible basics

Ansible automatizează configurarea mai multor servere, fără agent instalat pe target (doar SSH + Python).

**Inventory** — ce servere administrezi:
```ini
# inventory.ini
[web]
web01.example.com
web02.example.com

[db]
db01.example.com ansible_user=admin
```

**Ad-hoc commands** — o singură acțiune, fără playbook:
```bash
ansible web -i inventory.ini -m ping                          # verifică conectivitate + Python disponibil
ansible web -i inventory.ini -a "uptime"                        # rulează o comandă pe toate host-urile din grup
ansible all -i inventory.ini -m apt -a "name=nginx state=present" --become    # instalare pachet, ca root
```

**Playbook** — set de task-uri declarative, idempotente:
```yaml
# deploy-web.yml
- hosts: web
  become: true
  tasks:
    - name: Instalează Nginx
      apt:
        name: nginx
        state: present
        update_cache: true

    - name: Copiază configurația site-ului
      copy:
        src: files/site.conf
        dest: /etc/nginx/sites-available/site.conf

    - name: Activează site-ul
      file:
        src: /etc/nginx/sites-available/site.conf
        dest: /etc/nginx/sites-enabled/site.conf
        state: link
      notify: restart nginx

  handlers:
    - name: restart nginx
      service:
        name: nginx
        state: restarted
```
```bash
ansible-playbook -i inventory.ini deploy-web.yml
ansible-playbook -i inventory.ini deploy-web.yml --check     # dry-run, nu modifică nimic
```

**Roluri** — structură reutilizabilă pentru playbook-uri complexe (`roles/nginx/tasks/main.yml`, `roles/nginx/templates/`, etc.) — folosite prin `roles:` în playbook, în loc de `tasks:` inline.

---

## 10.5 — Git pentru sysadmin

Git nu e doar pentru cod — și configurația de infrastructură beneficiază de version control (**GitOps**: starea dorită a infrastructurii trăiește într-un repo Git, aplicată automat).

```bash
git init /etc/nginx                    # versionează configurația de sistem direct (practică comună pt. /etc)
git log --oneline -- sites-available/site.conf     # istoric al schimbărilor unui fișier de config specific
git diff HEAD~1 -- sysctl.conf                        # ce s-a schimbat exact, între două versiuni
```

**Git hooks** — automatizări la evenimente Git, ex: validare sintaxă la commit:
```bash
# .git/hooks/pre-commit (executabil)
#!/usr/bin/env bash
nginx -t || { echo "Config invalidă, commit blocat"; exit 1; }
```

**Git flow** simplificat pentru infrastructură: branch `main` = starea LIVE, branch-uri de feature pentru schimbări, PR/review înainte de merge, deploy automat la merge (vezi §10.6 CI/CD).

---

## 10.6 — CI/CD basics

**GitHub Actions** — pipeline definit în `.github/workflows/deploy.yml`:
```yaml
name: Deploy
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Rulează teste
        run: ./run-tests.sh
      - name: Deploy pe server
        run: |
          ansible-playbook -i inventory.ini deploy-web.yml
        env:
          ANSIBLE_HOST_KEY_CHECKING: "False"
```

**GitLab CI** — echivalent, în `.gitlab-ci.yml`:
```yaml
stages:
  - test
  - deploy

test:
  stage: test
  script:
    - ./run-tests.sh

deploy:
  stage: deploy
  script:
    - ansible-playbook -i inventory.ini deploy-web.yml
  only:
    - main
```

**Principiu:** fiecare push pe `main` declanșează automat teste, apoi (dacă trec) deploy — elimină pașii manuali predispuși la erori.

---

## 10.7 — Monitoring Stack

**Prometheus** — colectează metrici prin "pull" (scraping periodic de endpoint-uri `/metrics`):
```yaml
# prometheus.yml
scrape_configs:
  - job_name: 'node'
    static_configs:
      - targets: ['web01:9100', 'web02:9100']
```

**Node Exporter** — expune metrici de sistem (CPU, memorie, disk) în formatul pe care Prometheus îl poate citi:
```bash
docker run -d --name node-exporter -p 9100:9100 prom/node-exporter
curl localhost:9100/metrics | head -20     # format text simplu, "metric_name value"
```

**Grafana** — vizualizare, dashboard-uri, peste datele din Prometheus.

**Alerting** (Alertmanager) — reguli bazate pe metrici Prometheus:
```yaml
# alert.rules.yml
groups:
  - name: disk
    rules:
      - alert: DiskSpaceLow
        expr: node_filesystem_avail_bytes{mountpoint="/"} / node_filesystem_size_bytes{mountpoint="/"} < 0.1
        for: 5m
        annotations:
          summary: "Spațiu disk sub 10% pe {{ $labels.instance }}"
```

---

## 10.8 — Loguri centralizate

Pe măsură ce ai mai multe servere, `journalctl`/`grep` pe fiecare mașină individual nu mai scalează — trebuie centralizare.

- **ELK Stack** — Elasticsearch (stocare + căutare) + Logstash (parsare/ingest) + Kibana (vizualizare). Puternic, dar consumă resurse semnificative.
- **Loki + Grafana** — mai ușor, indexează doar metadate (nu conținutul complet, ca Elasticsearch), integrare nativă cu Grafana pentru corelare loguri+metrici în același dashboard.

```yaml
# promtail-config.yml (agent care trimite loguri către Loki)
scrape_configs:
  - job_name: syslog
    static_configs:
      - targets: [localhost]
        labels:
          job: varlogs
          __path__: /var/log/*.log
```

**Principiu:** logurile trebuie **agregate central**, cu retenție și căutare rapidă — un incident pe 10 servere e imposibil de investigat eficient sărind manual de pe o mașină pe alta.

---

## 🧪 Laborator practic

[👉 Laborator Modul 10](laborator.md) — deploy o aplicație containerizată cu Docker Compose, apoi provisionează-o automat cu un playbook Ansible complet.

## Exerciții practice
- Deploy aplicație containerizată cu Docker Compose
- Scrie un playbook Ansible care provisionează un server complet
