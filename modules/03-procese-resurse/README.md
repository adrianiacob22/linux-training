# 📦 Modul 3 — Gestionarea Proceselor & Resurselor

> **Nivel:** Mediu | **Durată estimată:** 4h
> 🚧 *Conținut detaliat în lucru*

[← Înapoi la cuprins](../../README.md)

## Obiective
- Monitorizarea și controlul proceselor
- Optimizarea utilizării resurselor

## Conținut
1. **Procesele în Linux** — PID, PPID, stări ale proceselor, fork/exec
2. **Vizualizare procese** — `ps`, `pstree`, `top`, `htop`, `atop`
3. **Semnale** — `kill`, `killall`, `pkill`, `SIGTERM`, `SIGKILL`, `SIGHUP`
4. **Prioritatea proceselor** — `nice`, `renice`, valori nice (-20 → 19)
5. **Jobs & Background** — `&`, `fg`, `bg`, `jobs`, `nohup`, `disown`
6. **Monitorizare resurse** — `free`, `vmstat`, `iostat`, `sar`, `lsof`
7. **Limite de resurse** — `ulimit`, `/etc/security/limits.conf`
8. **cgroups** — introducere, limitare CPU/memorie
9. **Planificarea task-urilor** — `cron`, `crontab`, `at`, `anacron`

## Exerciții practice
- Monitorizează un server sub load cu `htop` + `iostat`
- Configurează un job cron complex cu logging
