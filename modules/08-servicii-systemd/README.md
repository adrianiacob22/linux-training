# 📦 Modul 8 — Servicii & Systemd

> **Nivel:** Avansat | **Durată estimată:** 4h
> 🚧 *Conținut detaliat în lucru*

[← Înapoi la cuprins](../../README.md)

## Obiective
- Administrarea completă a serviciilor
- Crearea de unit-uri personalizate

## Conținut
1. **Systemd arhitectură** — units, targets, dependencies
2. **Gestionarea serviciilor** — `systemctl start/stop/restart/reload/enable/disable/status`
3. **Unit files** — `[Unit]`, `[Service]`, `[Install]` sections
4. **Tipuri de service** — `simple`, `forking`, `oneshot`, `notify`, `dbus`
5. **Timers** — înlocuitor modern pentru cron
6. **Journald** — `journalctl`, filtrare, persistență, dimensiune
7. **Targets** — runlevels moderne, schimbarea target-ului
8. **Socket activation & Path units**
9. **Crearea unui serviciu custom** — exemplu aplicație Python/Node.js

## Exerciții practice
- Creează un serviciu systemd pentru o aplicație web custom
- Înlocuiește un cron job cu un systemd timer
