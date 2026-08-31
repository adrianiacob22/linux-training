# 🐧 Curs Linux 

> **Audiență țintă:** Utilizatori cu cunoștințe de bază Linux (navigare în terminal, comenzi simple) care doresc să avanseze spre administrare de sistem și automatizare.
> **Durată estimată:** ~40–50 ore de studiu + practică

---

## 📚 Structura Cursului

| Modul | Titlu | Nivel | Ore est. | Status |
|-------|-------|-------|----------|--------|
| 1 | [Recapitulare & Fundamente Solide](modules/01-recapitulare-fundamente/README.md) | Mediu | 3h | ✅ |
| 2 | [Sistemul de Fișiere & Permisiuni Avansate](modules/02-sistem-fisiere-permisiuni/README.md) | Mediu | 4h | 🚧 |
| 3 | [Gestionarea Proceselor & Resurselor](modules/03-procese-resurse/README.md) | Mediu | 4h | 🚧 |
| 4 | [Rețelistică pe Linux](modules/04-retelistica/README.md) | Mediu–Avansat | 5h | 🚧 |
| 5 | [Gestionarea Utilizatorilor & Grupurilor](modules/05-utilizatori-grupuri/README.md) | Mediu | 3h | 🚧 |
| 6 | [Managementul Pachetelor & Compilare din Sursă](modules/06-pachete-compilare/README.md) | Mediu | 3h | 🚧 |
| 7 | [**Bash Scripting**](modules/07-bash-scripting/README.md) | Mediu–Avansat | 10h | ✅ |
| 8 | [Servicii & Systemd](modules/08-servicii-systemd/README.md) | Avansat | 4h | 🚧 |
| 9 | [Securitate pe Linux](modules/09-securitate/README.md) | Avansat | 5h | 🚧 |
| 10 | [Administrare Avansată & DevOps Intro](modules/10-administrare-devops/README.md) | Avansat | 5h | 🚧 |

✅ = conținut detaliat disponibil · 🚧 = doar plan (obiective + conținut + exerciții), urmează dezvoltarea

---

## 🛠️ Resurse Recomandate

### Cărți
- **"The Linux Command Line"** — William E. Shotts Jr. *(gratuit online)*
- **"Linux Bible"** — Christopher Negus
- **"How Linux Works"** — Brian Ward
- **"The Practice of System and Network Administration"** — Limoncelli et al.
- **"Linux Shell Scripting Cookbook"** — Clif Flynt & Sarath Lakshman

### Documentație Online
- [man7.org](https://man7.org/linux/man-pages/) — Manual pages Linux
- [tldp.org](https://tldp.org) — Linux Documentation Project
- [explainshell.com](https://explainshell.com) — Explicații comenzi
- [shellcheck.net](https://www.shellcheck.net) — Linter Bash online
- [devhints.io/bash](https://devhints.io/bash) — Bash cheat sheet

### Platforme de Practică
- **OverTheWire: Bandit** — provocări Linux (wargames)
- **TryHackMe** — module Linux
- **HackTheBox** — mașini Linux
- **KodeKloud** — cursuri interactive Linux/DevOps
- **Linux Foundation Training** — cursuri oficiale

### Tooling Recomandat
| Tool | Scop |
|------|------|
| `shellcheck` | Linter pentru scripturi Bash |
| `shfmt` | Formatare automată scripturi Bash |
| `bat` | `cat` cu syntax highlighting |
| `ripgrep (rg)` | `grep` modern și rapid |
| `fd` | `find` modern |
| `tmux` | Terminal multiplexer |
| `htop` / `btop` | Monitor sistem interactiv |

---

## 📋 Proiecte Finale (Capstone)

### Opțiunea A — Server de Producție
Configurează de la zero un server LEMP (Linux, Nginx, MySQL, PHP) complet securizat:
- SSL/TLS cu Let's Encrypt
- Firewall strict
- Monitoring cu alerting
- Backup automatizat
- Toate configurate prin Bash scripts + Ansible

### Opțiunea B — Pipeline CI/CD
Construiește un pipeline complet:
- Server Git (Gitea sau GitLab CE)
- CI/CD cu GitHub Actions sau GitLab CI
- Deploy automat cu zero-downtime
- Monitoring și logging

### Opțiunea C — Infrastructure as Code
- Provision complet al unui mediu multi-server cu Ansible
- Configurare Prometheus + Grafana + Alertmanager
- Scripturi de DR (Disaster Recovery)

---

## 📊 Evaluare & Certificare

| Tip | Pondere |
|-----|---------|
| Quiz-uri per modul | 20% |
| Exerciții practice | 40% |
| Proiect final (Capstone) | 40% |

**Prag de promovare:** 70% din total

---

*Curs creat: Mai 2026 | Versiune: 1.0*
