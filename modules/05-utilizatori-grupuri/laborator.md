# 🧪 Laborator — Modul 5: Gestionarea Utilizatorilor & Grupurilor

[← Înapoi la Modul 5](README.md) · [Cuprins curs](../../README.md)

## Scenariu

O echipă nouă de 3 persoane (`alice` — team lead, `bob` — developer, `carol` — developer) se alătură proiectului. Trebuie să le provizionezi conturile, să le organizezi în grupuri corecte, să acorzi `bob`-ului drepturi sudo limitate la restart-ul unui singur serviciu, și să configurezi o politică de parole rezonabilă pentru toți. Pe lângă asta, mai ai un cont de serviciu (`deploy-bot`) care nu trebuie să permită login interactiv deloc.

## Cerințe prealabile
- O mașină/VM Linux cu drepturi root/sudo

---

## Task 1 — Grup de echipă și utilizatori

Creează grupul `proiect-y`, apoi cei trei utilizatori, toți cu acest grup ca grup **secundar** (grupul principal al fiecăruia rămâne cel implicit, per-user).

<details>
<summary>✅ Soluție</summary>

```bash
groupadd proiect-y

useradd -m -s /bin/bash -c "Alice - Team Lead" -G proiect-y alice
useradd -m -s /bin/bash -c "Bob - Developer" -G proiect-y bob
useradd -m -s /bin/bash -c "Carol - Developer" -G proiect-y carol

for u in alice bob carol; do
    passwd -l "$u"     # blocat temporar, îl vom debloca la Task 5 cu o parolă setată corect
    id "$u"
done
```
</details>

---

## Task 2 — Corectează o greșeală clasică

Cineva a rulat din greșeală `usermod -G docker alice` (fără `-a`). Simulează greșeala, observă efectul asupra grupurilor lui alice, apoi repară situația astfel încât alice să aibă **atât** `proiect-y` **cât și** `docker`.

<details>
<summary>💡 Hint</summary>

`groupadd docker` întâi, dacă nu există deja. Compară `groups alice` înainte și după fiecare comandă.
</details>

<details>
<summary>✅ Soluție</summary>

```bash
groupadd docker 2>/dev/null || true

groups alice                          # ex: alice : alice proiect-y

usermod -G docker alice                 # GREȘEALA — înlocuiește toate grupurile secundare
groups alice                              # ex: alice : alice docker   (proiect-y a dispărut!)

usermod -aG proiect-y,docker alice          # -a = append, corect: adaugă fără să șteargă
groups alice                                  # alice : alice proiect-y docker
```
</details>

---

## Task 3 — Cont de serviciu, fără login interactiv

Creează `deploy-bot`: fără parolă utilizabilă pentru login, cu shell `/usr/sbin/nologin`, dar cu home directory (pentru chei SSH de deploy). Confirmă că un `su - deploy-bot` eșuează sau nu oferă shell interactiv.

<details>
<summary>✅ Soluție</summary>

```bash
useradd -m -s /usr/sbin/nologin -c "Deploy automation account" deploy-bot
passwd -l deploy-bot          # blochează orice autentificare prin parolă

su - deploy-bot
# ar trebui să afișeze ceva de genul "This account is currently not available." și să revină imediat
```

Un cont de tip `nologin` poate fi în continuare folosit pentru autentificare **prin cheie SSH** cu `command=` forțată în `authorized_keys`, dacă e nevoie doar de operațiuni specifice (ex: rsync de deploy), fără shell interactiv complet.
</details>

---

## Task 4 — Sudo granular pentru bob

`bob` trebuie să poată face **doar** `systemctl restart nginx`, fără parolă, și nimic altceva prin sudo. Configurează asta corect, prin `/etc/sudoers.d/`, nu editând direct `/etc/sudoers`.

<details>
<summary>✅ Soluție</summary>

```bash
echo 'bob ALL=(ALL) NOPASSWD: /usr/bin/systemctl restart nginx' > /etc/sudoers.d/bob-nginx
visudo -cf /etc/sudoers.d/bob-nginx     # validează sintaxa ÎNAINTE de a lăsa fișierul activ
chmod 440 /etc/sudoers.d/bob-nginx

# Testare (ca bob)
su - bob -c "sudo -l"                                # confirmă ce comenzi are voie
su - bob -c "sudo systemctl restart nginx"             # ar trebui să meargă fără prompt de parolă
su - bob -c "sudo systemctl restart sshd"                 # ar trebui REFUZAT — nu e în lista permisă
```
</details>

---

## Task 5 — Politică de parole

Setează pentru toți cei trei useri din echipă (`alice`, `bob`, `carol`):
- parola expiră la 90 de zile
- avertizare cu 7 zile înainte de expirare
- minim 2 zile între două schimbări consecutive de parolă
- forțează schimbarea parolei la primul login

<details>
<summary>✅ Soluție</summary>

```bash
for u in alice bob carol; do
    echo "$u:ParolaTemporara$RANDOM" | chpasswd     # deblochează + setează o parolă temporară
    chage -M 90 -m 2 -W 7 -d 0 "$u"                    # -d 0 forțează schimbare la următorul login
    chage -l "$u"                                        # verificare
done
```

Opțional, pentru complexitate (dacă `libpam-pwquality` e instalat):
```bash
apt install -y libpam-pwquality 2>/dev/null || true
sed -i 's/^# minlen.*/minlen = 12/' /etc/security/pwquality.conf 2>/dev/null || \
    echo "minlen = 12" >> /etc/security/pwquality.conf
```
</details>

---

## Task 6 — Audit

Simulează câteva login-uri (sau folosește sesiunea curentă), apoi răspunde:
1. Cine e logat acum și de pe ce terminal?
2. Care a fost ultimul login al lui `carol` (probabil "never" dacă tocmai a fost creată)?
3. Câte reboot-uri a avut mașina recent?

<details>
<summary>✅ Soluție</summary>

```bash
who
w
lastlog | grep -E 'alice|bob|carol'
last reboot | head -5
```
</details>

---

## ✅ Checklist final

- [ ] Pot crea utilizatori cu grup principal + grupuri secundare corecte de la început
- [ ] Știu de ce `-a` la `usermod -aG` e obligatoriu și ce se strică dacă lipsește
- [ ] Pot crea un cont de serviciu fără login interactiv, dar utilizabil pentru automatizări
- [ ] Pot restrânge `sudo` la o singură comandă, pentru un singur utilizator, validat cu `visudo -cf`
- [ ] Pot configura o politică de expirare/complexitate a parolelor cu `chage`
- [ ] Știu ce unealtă de audit să folosesc pentru fiecare tip de întrebare (`who`, `w`, `last`, `lastlog`)
