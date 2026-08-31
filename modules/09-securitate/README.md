# 📦 Modul 9 — Securitate pe Linux

> **Nivel:** Avansat | **Durată estimată:** 5h
> 🚧 *Conținut detaliat în lucru*

[← Înapoi la cuprins](../../README.md)

## Obiective
- Hardening sistem Linux
- Detecția și răspunsul la incidente

## Conținut
1. **Principiul privilegiului minim**
2. **SSH Hardening** — dezactivare root login, schimbare port, fail2ban, 2FA
3. **Firewall avansat** — iptables/nftables, politici stricte, logging
4. **SELinux / AppArmor** — moduri, politici, troubleshooting
5. **Auditare sistem** — `auditd`, reguli audit, analiza log-urilor
6. **Scanare vulnerabilități** — `lynis`, `rkhunter`, `chkrootkit`, `nmap`
7. **Criptare** — LUKS (disk encryption), GPG (fișiere), SSL/TLS
8. **Umask, SUID/SGID audit**
9. **Kernel hardening** — `sysctl`, parametri de securitate

## Exerciții practice
- Hardening complet al unui server Ubuntu fresh install
- Configurare SELinux pentru o aplicație web
