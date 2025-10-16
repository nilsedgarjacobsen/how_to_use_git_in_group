# Scenario 1: Uppdatera gammal feature-branch med nya ändringar från main

## Situationen
Du har jobbat i din feature-branch `feature/user-login` i några dagar. Under tiden har flera teammedlemmar mergat sina branches till `main`. Din branch är nu "efter" och du behöver få in de senaste ändringarna.

## Varför behöver du göra detta?
- **Undvika stora merge-konflikter senare** - Ju längre du väntar, desto fler konflikter kan uppstå
- **Testa mot senaste koden** - Din feature kan ha blivit inkompatibel med nya ändringar
- **Lättare code review** - Reviewers ser att din kod fungerar med senaste main
- **Teamets förväntan** - Många team kräver att PR:en är uppdaterad innan merge

## Två huvudmetoder: Merge vs Rebase

### Metod 1: Merge (säkrare, enklare)
Skapar en merge-commit som kombinerar main och din branch.

**Fördelar:**
- Säkrare - omöjligt att förstöra historik
- Bevarar exakt när du integrerade main
- Funkar alltid, även på delade branches

**Nackdelar:**
- Skapar extra merge-commits (kan bli rörigt)
- Historiken blir inte linjär

### Metod 2: Rebase (renare historik)
"Flyttar" dina commits ovanpå senaste main.

**Fördelar:**
- Ren, linjär historik
- Ser ut som om du just börjat från senaste main
- Lättare att följa i git-loggen

**Nackdelar:**
- Kräver force push om du redan pushat
- Kan orsaka problem om andra jobbar på samma branch
- Något svårare att förstå

## Steg-för-steg: Merge-metoden

### Med Git-kommandon

```bash
# 1. Se till att du inte har uncommitted changes
git status

# 2. Byt till din feature-branch (om du inte redan är där)
git checkout feature/user-login

# 3. Hämta senaste från remote
git fetch origin

# 4. Merga in main
git merge origin/main

# 5. Lös eventuella konflikter (se scenario 2)
# Redigera filer, sen:
git add .
git commit

# 6. Pusha de uppdaterade ändringarna
git push origin feature/user-login
```

### Med IntelliJ IDEA

1. **Öppna Git-fönstret:** `View → Tool Windows → Git` (eller `Alt+9`)
2. **Uppdatera från remote:** Klicka på den blå nedåtpilen "Update Project" i verktygsfältet (eller `Ctrl+T`)
3. **Välj merge-metod:** 
   - I dialogrutan som dyker upp, välj "Merge" under "Update method"
   - Säkerställ att "origin/main" är vald som källa
4. **Klicka "OK"**
5. **Vid konflikter:** IntelliJ öppnar en merge-dialog automatiskt (se scenario 2)
6. **Pusha:** Klicka på den gröna uppåtpilen "Push" i verktygsfältet (eller `Ctrl+Shift+K`)

## Steg-för-steg: Rebase-metoden

### Med Git-kommandon

```bash
# 1. Se till att du inte har uncommitted changes
git status

# 2. Byt till din feature-branch
git checkout feature/user-login

# 3. Hämta senaste från remote
git fetch origin

# 4. Rebasea på main
git rebase origin/main

# 5. Lös eventuella konflikter (en åt gången)
# Redigera filer, sen:
git add .
git rebase --continue
# Upprepa tills alla konflikter är lösta

# 6. Force push (eftersom historiken har ändrats)
git push --force-with-lease origin feature/user-login
```

**VIKTIGT:** Använd `--force-with-lease` istället för `-f`. Det skyddar dig från att skriva över ändringar som någon annan pushat.

### Med IntelliJ IDEA

1. **Öppna Git-fönstret:** `View → Tool Windows → Git`
2. **Högerklicka på din branch** i branch-listan (längst ner till höger)
3. **Välj:** `Checkout` (för att byta till den)
4. **Högerklicka på `origin/main`** i Git-loggen
5. **Välj:** `Rebase Current onto Selected`
6. **Vid konflikter:** IntelliJ hjälper dig lösa dem en i taget
7. **Force push:** 
   - Klicka på den gröna uppåtpilen "Push"
   - IntelliJ varnar att du behöver force push
   - Bocka i "Force push" (IntelliJ använder `--force-with-lease` automatiskt)

## Så påverkas git-historiken

### Före uppdatering
```
main:           A---B---C---D
                 \
feature-branch:   E---F---G
```

### Efter merge
```
main:           A---B---C---D
                 \           \
feature-branch:   E---F---G---M  (M = merge commit)
```
Din branch har nu alla commits från main (B, C, D) PLUS en ny merge-commit (M).

### Efter rebase
```
main:           A---B---C---D
                             \
feature-branch:               E'---F'---G'
```
Dina commits (E, F, G) har "flyttats" och fått nya ID:n (E', F', G'). De baseras nu på D istället för A.

## När uppdateringen är klar

### Vad händer när du senare mergar till main?

**Efter merge-uppdatering:**
```
main: A---B---C---D---M---H  (H = din PR merge)
       \           \ /
        E---F---G---M
```
Alla dina commits (E, F, G) och merge-commiten (M) syns i main.

**Efter rebase-uppdatering:**
```
main: A---B---C---D---E'---F'---G'
```
Dina commits ligger prydligt i rad efter D. Mycket renare historik!

## Vanliga fallgropar

### ❌ Glömmer att fetcha först
```bash
git merge origin/main  # Mergar gammal version av main!
```
**Lösning:** Kör alltid `git fetch` först.

### ❌ Rebasar en delad branch
Om din kollega också jobbar på `feature/user-login` och du rebasar, förstör du för hen.

**Lösning:** Använd merge på delade branches, rebase bara på dina egna.

### ❌ Force push utan --force-with-lease
```bash
git push -f  # Kan skriva över andras arbete!
```
**Lösning:** Använd alltid `--force-with-lease`.

## Verifiering - hur vet du att det gick bra?

```bash
# Kolla att du har de senaste commits från main
git log --oneline --graph -10

# Kolla att dina ändringar finns kvar
git diff origin/main

# Kör testerna!
npm test  # eller ditt testkommando
```

**I IntelliJ:** 
- Kolla Git-loggen (`Alt+9`, fliken "Log")
- Verifiera att main:s senaste commits finns
- Kör testerna (`Ctrl+Shift+F10`)

## Generella tips

✅ **Uppdatera ofta** - Helst dagligen om main är aktiv
✅ **Små commits** - Lättare att rebasea och lösa konflikter
✅ **Korta branches** - Ju kortare tid din branch lever, desto färre konflikter
✅ **Kör tester efter uppdatering** - Säkerställ att integration fungerar
✅ **Kommunicera vid force push** - Meddela teamet om ni delar branch

## När ska du INTE göra detta?

- **Precis innan en demo** - Risk att något går sönder
- **På fredagskvällen** - Ingen tid att fixa om något går fel
- **När du har uncommitted changes** - Stash eller commita först (se scenario 5)

## Ångra om något går fel

### Ångra merge
```bash
git merge --abort  # Under pågående merge
git reset --hard HEAD~1  # Efter mergad men inte pusha
```

### Ångra rebase
```bash
git rebase --abort  # Under pågående rebase
git reset --hard ORIG_HEAD  # Precis efter rebase
```

**I IntelliJ:** `Git → Abort Merge/Rebase` från menyn

## Rekommendation

För de flesta team: **Börja med merge-metoden**. Den är säkrare och lättare att förstå. När du känner dig bekväm kan du experimentera med rebase på dina egna, isolerade branches.

Vissa team har policys - fråga ditt team vilken metod ni föredrar!