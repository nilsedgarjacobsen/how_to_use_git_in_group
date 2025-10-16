# Scenario 3: Jag har committat något fel

## Situationen
Du har precis committat och inser att något är fel:
- Du inkluderade fel filer (t.ex. `.env` med lösenord)
- Du råkade commita känslig data (API-nycklar, personuppgifter)
- Commit-meddelandet är dåligt eller felaktigt
- Du glömde inkludera en viktig fil
- Du vill ändra något i själva koden

## VIKTIGT: Har du pushat eller inte?

Detta är den avgörande frågan eftersom:
- **Inte pushat** = Enkelt att fixa, påverkar bara dig
- **Redan pushat** = Svårare, kan påverka teamet

**Kolla om du pushat:**
```bash
git status
# Visar "Your branch is ahead of 'origin/feature-branch' by 1 commit"
# = Du har INTE pushat än
```

## Scenario A: Du har INTE pushat ännu (enkelt!)

### Problem 1: Dåligt commit-meddelande

**Situationen:**
```bash
git commit -m "fix"  # Oj, för vagt meddelande!
```

**Lösning - Ändra senaste commit:**
```bash
git commit --amend -m "Fix: Resolve null pointer exception in UserService.login()"
```

**I IntelliJ:**
1. `Git → Uncommit...` (eller `Ctrl+Alt+Z`)
2. Bocka i "Amend"
3. Skriv nytt meddelande
4. Klicka "Uncommit"

**Påverkan på historik:**
```
Före:  A---B---C (C har dåligt meddelande)
Efter: A---B---C' (C' har bra meddelande, men NYTT commit-ID)
```

### Problem 2: Glömde lägga till en fil

**Situationen:**
```bash
git commit -m "Add user validation"
# Oj, glömde ValidationHelper.java!
```

**Lösning - Lägg till filen i senaste commit:**
```bash
# Lägg till den glömda filen
git add src/ValidationHelper.java

# Amenda commiten (behåller samma meddelande)
git commit --amend --no-edit
```

**I IntelliJ:**
1. Markera filen i Project-vyn
2. `Git → Add` (eller `Ctrl+Alt+A`)
3. `Git → Commit` (eller `Ctrl+K`)
4. Bocka i "Amend commit"
5. Klicka "Commit"

**Påverkan på historik:**
```
Före:  A---B---C (C innehåller bara UserService.java)
Efter: A---B---C' (C' innehåller både UserService.java och ValidationHelper.java)
```

### Problem 3: Vill ändra själva koden

**Situationen:**
```bash
git commit -m "Add login feature"
# Oj, jag vill ändra något i koden!
```

**Lösning - Ändra kod och amenda:**
```bash
# Gör dina ändringar i filerna
# ...

# Lägg till ändringarna
git add .

# Amenda commiten
git commit --amend --no-edit
```

**I IntelliJ:**
- Gör dina ändringar
- `Git → Commit` → Bocka i "Amend commit"

### Problem 4: Inkluderade fel fil (t.ex. .env)

**Situationen:**
```bash
git commit -m "Add config"
# Oj, jag commitade .env med lösenord!
```

**Lösning - Ta bort filen från commiten:**
```bash
# Ta bort från commit (men BEHÅLL lokalt)
git reset HEAD~1

# Eller ta bort filen från både commit OCH lokal disk
git reset --hard HEAD~1
# (VARNING: Detta tar bort alla dina ändringar!)

# Bättre: Ta bort bara den filen från staging
git restore --staged .env

# Lägg till i .gitignore så det inte händer igen
echo ".env" >> .gitignore
git add .gitignore
git commit -m "Add .env to gitignore"

# Commita resten igen
git add .
git commit -m "Add config (without .env)"
```

**I IntelliJ:**
1. `Git → Uncommit...` (eller `Ctrl+Alt+Z`)
2. Välj "Reset" och "Soft" (behåller ändringar)
3. Klicka "Uncommit"
4. Ta bort .env från Staging Area
5. Lägg till .env i .gitignore
6. Commita på nytt

**Påverkan på historik:**
```
Före:  A---B---C (C innehåller .env)
Efter: A---B (C är borta, ändringar är tillbaka i working directory)
```

## Scenario B: Du har redan pushat (svårare!)

### Problem 1: Ändra commit-meddelande (pushat)

**VARNING:** Detta kräver force push och kan störa teamet!

```bash
# Ändra meddelandet
git commit --amend -m "Bättre meddelande"

# Force push
git push --force-with-lease origin feature/user-login
```

**När du INTE ska göra detta:**
- Om andra jobbar på samma branch
- På main eller develop (nästan aldrig!)
- Om commiten redan är mergad till main

**Kommunicera med teamet:**
```
"Heads up! Jag kommer force-pusha feature/user-login för att fixa ett 
commit-meddelande. Om ni har lokal version, kör: git pull --rebase"
```

### Problem 2: Känslig data redan pushad (KRITISKT!)

**Situationen:** Du pushade API-nycklar, lösenord, eller personuppgifter.

**OMEDELBAR ÅTGÄRD:**

1. **Rotera hemligheter FÖRST**
   - Byt lösenord/API-nycklar INNAN du fixar Git
   - Känslig data är redan exponerad i historiken!

2. **Ta bort från senaste commit (om senaste):**
```bash
# Uncommit
git reset HEAD~1

# Ta bort känslig fil
git restore --staged .env

# Lägg till i .gitignore
echo ".env" >> .gitignore

# Commita på nytt
git add .
git commit -m "Add config (removed sensitive data)"

# Force push
git push --force-with-lease origin feature/user-login
```

3. **Om det är längre bak i historiken:**

Git-historik är permanent. Du måste använda `git filter-branch` eller `git filter-repo` (avancerat).

```bash
# Detta är komplext - kontakta senior utvecklare eller DevOps!
# Alternativ: radera hela branchen och börja om
```

**I IntelliJ:** Det finns inget enkelt UI för detta. Använd kommandoraden.

**Preventivt:**
- Använd alltid `.gitignore` för känsliga filer
- Använd environment variables
- Lägg aldrig in riktiga credentials i kod

### Problem 3: Vill ta bort flera commits (pushat)

**Situationen:**
```bash
git log --oneline
# abc123 Fix typo
# def456 Add feature
# ghi789 WIP checkpoint  # <- Vill ta bort denna
# jkl012 Previous work
```

**Lösning - Interactive rebase:**
```bash
# Gå tillbaka 3 commits
git rebase -i HEAD~3
```

En editor öppnas:
```
pick ghi789 WIP checkpoint
pick def456 Add feature
pick abc123 Fix typo
```

Ändra till (för att ta bort WIP):
```
drop ghi789 WIP checkpoint
pick def456 Add feature
pick abc123 Fix typo
```

Spara och stäng. Sedan:
```bash
git push --force-with-lease origin feature/user-login
```

**I IntelliJ:**
1. Öppna Git-loggen (`Alt+9`)
2. Högerklicka på commiten FÖRE den du vill ändra
3. Välj `Interactively Rebase from Here...`
4. I dialogen: markera raden och klicka "Drop" eller "Edit"
5. Klicka "Start Rebasing"
6. Force push

**Påverkan på historik:**
```
Före:  A---B---C---D---E
Efter: A---B---D'---E' (C borttagen, D och E har nya ID:n)
```

## Alternativ: Revert istället för force push

Om branchen är delad eller du vill undvika force push:

```bash
# Skapar en NY commit som ångrar den gamla
git revert HEAD

# Pusha normalt (ingen force behövs)
git push origin feature/user-login
```

**Fördelar:**
- Säkert - förstör ingen historik
- Funkar på delade branches
- Transparent - alla ser vad som hände

**Nackdelar:**
- Rörig historik - både fel och fix syns
- Fungerar inte för känslig data (den är fortfarande i historiken)

**Påverkan på historik:**
```
A---B---C---C' (C' ångrar C, men C finns kvar i historiken)
```

## Vanliga fallgropar

### ❌ Force push utan --force-with-lease
```bash
git push -f  # Kan skriva över andras commits!
```
**Lösning:** Använd ALLTID `--force-with-lease`.

### ❌ Amenda efter push utan att kommunicera
Teamet får konflikter när de pullar.

**Lösning:** Kommunicera alltid före force push.

### ❌ Tror att "git reset" tar bort från remote
`git reset` påverkar bara lokal historik. Remote är oförändrad tills du pushar.

### ❌ Använder --hard utan att vara säker
```bash
git reset --hard HEAD~1  # ALLA ändringar försvinner!
```
**Lösning:** Använd `--soft` om du vill behålla ändringar.

## Olika reset-lägen förklarat

```bash
git reset --soft HEAD~1   # Uncommittar, behåller i staging
git reset --mixed HEAD~1  # Uncommittar, tar bort från staging (default)
git reset --hard HEAD~1   # Uncommittar, TAR BORT ALLA ÄNDRINGAR
```

**Visualisering:**
```
Working Directory → Staging Area → Commit

--soft:  Flyttar från Commit → Staging Area
--mixed: Flyttar från Commit → Working Directory
--hard:  RADERAR ALLT
```

## Verifiering - hur vet du att det gick bra?

```bash
# Kolla senaste commits
git log --oneline -5

# Kolla vad som finns i senaste commit
git show HEAD

# Kolla att känslig fil inte finns
git log --all --full-history -- .env
# (Ska inte visa något om filen verkligen är borta)

# Kolla status
git status
```

**I IntelliJ:**
- Kolla Git-loggen för att se commits
- `Git → Show History` för att se filhistorik

## Generella tips för att undvika fel commits

✅ **Använd .gitignore** - Lägg till känsliga filer INNAN du börjar jobba
✅ **Granska före commit** - `git diff --staged` för att se vad som committas
✅ **Små, fokuserade commits** - Lättare att hitta och fixa fel
✅ **Använd commit hooks** - Automatiska checkar (t.ex. pre-commit hook som blockerar .env)
✅ **Commit ofta lokalt** - Pusha först när du är säker
✅ **Skriv tydliga meddelanden** - Ta dig tid att skriva bra meddelanden från början

## När ska du INTE amenda/force push?

- ❌ På main eller develop branch
- ❌ På commits som redan är mergeade till main
- ❌ På delade feature-branches (utan att kommunicera)
- ❌ På publika branches som andra förväntas använda
- ❌ På fredag kväll (ingen tid att fixa om något går fel)

## Ångra om något går fel

### Hittar ditt gamla commit
```bash
# Git sparar en log över alla HEAD-ändringar
git reflog

# Hitta ditt gamla commit i listan, t.ex:
# abc123 HEAD@{1}: commit: Original commit

# Återställ till det
git reset --hard abc123
```

**I IntelliJ:**
- `Git → Uncommit...` steg för steg
- Eller hitta commiten i Git-loggen → högerklicka → `Reset Current Branch to Here`

## Rekommendation

**Inte pushat än?**
→ Använd `--amend` fritt och utan bekymmer

**Redan pushat?**
→ Tänk noga! Överväg `revert` istället för force push, speciellt på delade branches

**Känslig data?**
→ ROTERA SECRETS FÖRST, sedan fixa Git, kommunicera med teamet

Med dessa verktyg kan du fixa nästan alla commit-misstag - lugn är nyckeln!