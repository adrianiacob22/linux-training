# 🧪 Laborator — Modul 10: Administrare Avansată & DevOps Intro

[← Înapoi la Modul 10](README.md) · [Cuprins curs](../../README.md)

## Scenariu

Acesta e laboratorul final al cursului — combină aproape tot ce ai învățat până acum într-un flux DevOps complet: diagnostichezi o problemă de performanță metodic, containerizezi o aplicație, o provisionezi automat cu Ansible, versionezi configurația în Git, și pui monitorizare de bază peste tot.

## Cerințe prealabile
- O mașină/VM Linux cu Docker instalat (`docker --version`)
- Python 3 + `pip install ansible` (sau `apt install ansible`)
- Git

---

## Task 1 — Diagnostic metodic (USE Method)

Generează artificial o presiune pe CPU (`yes > /dev/null &` de câteva ori) și investighează folosind metoda USE: Utilization, Saturation, Errors. Documentează (într-un fișier text scurt) ce observi la fiecare pas.

<details>
<summary>✅ Soluție</summary>

```bash
for i in 1 2 3; do yes > /dev/null & done

mpstat -P ALL 1 3         # Utilization — %usr aproape de 100 pe core-urile ocupate
uptime                       # Saturation — load average crescut, peste numărul de core-uri
dmesg | tail -20               # Errors — de regulă gol în acest caz artificial

pkill -f "yes > /dev/null"
```

Concluzia de documentat: load average > nr. de core-uri = procese care așteaptă CPU (saturație), nu doar utilizare ridicată.
</details>

---

## Task 2 — Containerizează o mică aplicație

Creează o aplicație Flask/Python minimală care răspunde cu "OK" pe `/health`, scrie un `Dockerfile`, construiește imaginea și rulează containerul.

<details>
<summary>✅ Soluție</summary>

```bash
mkdir -p ~/lab-devops/app && cd ~/lab-devops/app

cat > app.py <<'EOF'
from http.server import HTTPServer, BaseHTTPRequestHandler

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        self.send_response(200)
        self.end_headers()
        self.wfile.write(b"OK")

HTTPServer(("0.0.0.0", 8000), Handler).serve_forever()
EOF

cat > Dockerfile <<'EOF'
FROM python:3.12-slim
WORKDIR /app
COPY app.py .
EXPOSE 8000
CMD ["python", "app.py"]
EOF

docker build -t lab-app:1.0 .
docker run -d --name lab-app -p 8000:8000 lab-app:1.0
curl localhost:8000     # ar trebui să răspundă "OK"
```
</details>

---

## Task 3 — Docker Compose

Extinde aplicația cu un al doilea "serviciu" (folosește `redis` ca exemplu simplu, fără să fie neapărat folosit de cod — scopul e exercițiul de orchestrare). Definește totul într-un `docker-compose.yml` și pornește cu un singur comand.

<details>
<summary>✅ Soluție</summary>

```bash
cd ~/lab-devops/app
docker rm -f lab-app     # curăță containerul pornit manual la Task 2

cat > docker-compose.yml <<'EOF'
services:
  app:
    build: .
    ports:
      - "8000:8000"
    depends_on:
      - cache
  cache:
    image: redis:7-alpine
EOF

docker compose up -d
docker compose ps
curl localhost:8000

docker compose logs app
docker compose down
```
</details>

---

## Task 4 — Provisioning cu Ansible

Scrie un inventory cu `localhost` (mod `ansible_connection=local`, ca să nu ai nevoie de un al doilea server) și un playbook care instalează un pachet (ex: `tree` sau `curl`) și confirmă instalarea.

<details>
<summary>💡 Hint</summary>

Pentru testare rapidă fără infrastructură reală, `ansible_connection=local` rulează task-urile direct pe mașina curentă, fără SSH.
</details>

<details>
<summary>✅ Soluție</summary>

```bash
cd ~/lab-devops
cat > inventory.ini <<'EOF'
[local]
localhost ansible_connection=local
EOF

ansible local -i inventory.ini -m ping

cat > provision.yml <<'EOF'
- hosts: local
  become: true
  tasks:
    - name: Instalează tree
      apt:
        name: tree
        state: present
        update_cache: true

    - name: Confirmă instalarea
      command: tree --version
      register: tree_version

    - name: Afișează versiunea
      debug:
        msg: "{{ tree_version.stdout }}"
EOF

ansible-playbook -i inventory.ini provision.yml --check     # dry-run întâi
ansible-playbook -i inventory.ini provision.yml               # execuție reală

ansible-playbook -i inventory.ini provision.yml                # rulează A DOUA OARĂ
# observă: "changed=0" — idempotența playbook-ului, nu reinstalează inutil
```
</details>

---

## Task 5 — Versionare configurație (GitOps light)

Inițializează un repo Git pentru directorul `~/lab-devops`, fă un commit inițial, apoi adaugă un pre-commit hook care validează sintaxa `docker-compose.yml` înainte de a permite commit-ul.

<details>
<summary>✅ Soluție</summary>

```bash
cd ~/lab-devops
git init
git add .
git commit -m "Setup inițial: app containerizată + playbook Ansible"

cat > .git/hooks/pre-commit <<'EOF'
#!/usr/bin/env bash
if [ -f app/docker-compose.yml ]; then
    docker compose -f app/docker-compose.yml config >/dev/null || {
        echo "docker-compose.yml invalid — commit blocat"
        exit 1
    }
fi
EOF
chmod +x .git/hooks/pre-commit

# testează: strică temporar sintaxa YAML și încearcă un commit
echo "  invalid: [yaml" >> app/docker-compose.yml
git add app/docker-compose.yml
git commit -m "test strică sintaxa"     # ar trebui BLOCAT de hook

git checkout -- app/docker-compose.yml    # revert
```
</details>

---

## Task 6 — Monitorizare de bază

Pornește `node-exporter` într-un container și confirmă că expune metrici. Scrie (fără să rulezi neapărat un Prometheus complet) un `prometheus.yml` care l-ar targeta.

<details>
<summary>✅ Soluție</summary>

```bash
docker run -d --name node-exporter -p 9100:9100 prom/node-exporter

curl -s localhost:9100/metrics | grep node_memory_MemAvailable_bytes

cat > ~/lab-devops/prometheus.yml <<'EOF'
scrape_configs:
  - job_name: 'node'
    scrape_interval: 15s
    static_configs:
      - targets: ['localhost:9100']
EOF

# opțional, dacă vrei să vezi Prometheus complet rulând:
docker run -d --name prometheus -p 9090:9090 \
    -v ~/lab-devops/prometheus.yml:/etc/prometheus/prometheus.yml \
    prom/prometheus
# deschide http://localhost:9090 și verifică targetul "node" ca UP
```
</details>

---

## Task 7 — Pipeline CI/CD (schiță)

Scrie (fără să fie neapărat rulat, dacă nu ai un repo GitHub la îndemână) un workflow GitHub Actions care, la push pe `main`, rulează `docker compose config` (validare) și apoi ar declanșa playbook-ul Ansible.

<details>
<summary>✅ Soluție</summary>

```bash
mkdir -p ~/lab-devops/.github/workflows
cat > ~/lab-devops/.github/workflows/deploy.yml <<'EOF'
name: Deploy

on:
  push:
    branches: [main]

jobs:
  validate-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Validează docker-compose
        run: docker compose -f app/docker-compose.yml config

      - name: Instalează Ansible
        run: pip install ansible

      - name: Rulează playbook (dry-run)
        run: ansible-playbook -i inventory.ini provision.yml --check
EOF

git add .github
git commit -m "Adaugă pipeline CI/CD"
```
</details>

---

## Task 8 — Curățenie

Oprește și șterge toate containerele/imaginile create în acest laborator.

<details>
<summary>✅ Soluție</summary>

```bash
docker rm -f node-exporter prometheus 2>/dev/null
cd ~/lab-devops/app && docker compose down --rmi local
```
</details>

---

## ✅ Checklist final

- [ ] Pot aplica USE Method pentru a diagnostica o problemă de performanță, nu doar "mă uit la top"
- [ ] Pot construi și rula o aplicație containerizată de la zero (`Dockerfile` + `docker compose`)
- [ ] Pot scrie un playbook Ansible idempotent (rulat a doua oară, nu schimbă nimic inutil)
- [ ] Versionez configurația de infrastructură în Git, cu validare automată prin hooks
- [ ] Înțeleg structura de bază a unui pipeline CI/CD (validare → deploy)
- [ ] Pot expune și verifica metrici de sistem cu Node Exporter

---

## 🎓 Felicitări!

Ai parcurs toate cele 10 module ale cursului. Pentru a-ți consolida cunoștințele end-to-end, treci la unul din **Proiectele Finale (Capstone)** din [README-ul principal](../../README.md#-proiecte-finale-capstone) — acolo combini tot ce ai învățat într-un singur proiect complet, de la infrastructură până la monitorizare.
