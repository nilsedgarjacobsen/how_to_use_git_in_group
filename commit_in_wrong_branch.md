# Scenario 4: Jag har committat på fel branch

## Situationen
Du har precis committat och inser att du är på fel branch:
- Du skulle jobba i `feature/user-login` men är på `main`
- Du committade på `develop` istället för din feature-branch
- Du har flera commits på fel branch
- Du upptäcker felet efter att ha pushat

## Varför händer detta?

- **Glömde brancha** - Började jobba direkt utan att kolla nuvarande branch
- **Automatiskt checkout** - IDE:n eller Git checkade ut fel branch
- **Förvirring** - Många branches öppna, tappade koll på vilken som är aktiv
- **Efter pull** - Pull kan ibland checka ut en annan branch

## VIKTIGT: Kolla alltid vilken branch du är på!

```bash
# Se nuvarande branch
git branch
# eller
git status
```

**I IntelliJ:** Kolla längst ner till höger - visar nuvarande branch.

**Tips:** Konfigurera din terminal att visa branch-namn i prompten!

## Scenario A: Ett commit på fel branch (inte pushat)

### Situationen
```bash
git status
# On branch main  # <-- Ajdå!

git log --oneline -1
# abc123 Add login feature  # Detta skulle varit på feature/user-login
```

### Lösning - Flytta commiten till rätt branch

**Steg 1: Skapa (eller byt till) rätt branch**
```bash
# Om branchen inte finns - skapa den MED din commit
git branch feature/user-login

# Nu har båda branches samma commits:
# main:                A---B---C (C = ditt nya commit)
# feature/user-login:  A---B---C (samma C)
```

**Steg 2: Ta bort commiten från fel branch**
```bash
# Byt till den felaktiga branchen (om du inte redan är där)
git checkout main

# Ta bort senaste commit från main
git reset --hard HEAD~1

# Nu ser det ut så här:
# main:                A---B (C är borta härifrån)
# feature/user-login:  A---B---C (C finns här!)
```

**Steg 3: Byt till rätt branch och fortsätt jobba**
```bash
git checkout feature/user-login
# Nu är allt på rätt ställe!
```

### Med IntelliJ IDEA

**Steg 1: Skapa ny branch från nuvarande position**
1. Längst ner till höger: Klicka på branch-namnet (t.ex. "main")
2. Välj `New Branch...`
3. Döp den till `feature/user-login`
4. Bocka i "Checkout branch" 
5. Klicka "Create"

**Steg 2: Återställ main**
1. Byt tillbaka till main: Klicka på branch-namnet → välj `main` → `Checkout`
2. Öppna Git-loggen (`Alt+9`)
3. Högerklicka på commiten FÖRE ditt felaktiga commit
4. Välj `Reset Current Branch to Here...`
5. Välj "Hard" (tar bort commiten helt)
6. Klicka "Reset"

**Steg 3: Byt tillbaka till rätt branch**
1. Klicka på branch-namnet → välj `feature/user-login` → `Checkout`

### Påverkan på historik
```
Start:  main (HEAD):  A---B---C (fel!)

Efter:  main:                A---B
        feature/user-login:  A---B---C (rätt!)
```

## Scenario B: Flera commits på fel branch (inte pushat)

### Situationen
```bash
git log --oneline -3
# abc123 Add validation
# def456 Add helper class  
# ghi789 Add login feature
# ^ Alla tre ska till feature/user-login, inte main!
```

### Lösning - Flytta flera commits

**Steg 1: Skapa branch från nuvarande position**
```bash
# Skapar branch som inkluderar alla dina commits
git branch feature/user-login
```

**Steg 2: Ta bort commits från fel branch**
```bash
# Återställ main till innan dina commits (3 commits tillbaka)
git reset --hard HEAD~3
```

**Steg 3: Byt till rätt branch**
```bash
git checkout feature/user-login
```

### Alternativ: Cherry-pick (om du bara vill ha vissa commits)

**Situationen:** Du vill ha commit `def456` och `abc123`, men INTE `ghi789`.

```bash
# Skapa och byt till rätt branch (från rätt startpunkt)
git checkout -b feature/user-login HEAD~3

# Cherry-pick de commits du vill ha
git cherry-pick def456
git cherry-pick abc123

# Ta bort alla tre från main
git checkout main
git reset --hard HEAD~3
```

**I IntelliJ:**
1. Öppna Git-loggen
2. Högerklicka på commiten du vill kopiera
3. Välj `Cherry-Pick`
4. Upprepa för alla commits du vill ha
5. Återställ main som tidigare

### Påverkan på historik
```
Start:  main:  A---B---C---D---E

Efter:  main:                A---B
        feature/user-login:  A---B---C---D---E
```

## Scenario C: Committat på fel branch OCH pushat

### Situationen
```bash
git log --oneline -1
# abc123 Add login feature

git push origin main  # Oj, pushade till main!
```

**Detta är allvarligare** eftersom andra kan ha pullt dina commits.

### Lösning - Om ingen hunnit pulla

**Steg 1: Skapa rätt branch**
```bash
git branch feature/user-login
git push origin feature/user-login
```

**Steg 2: Ta bort från main (lokalt)**
```bash
git reset --hard HEAD~1
```

**Steg 3: Force push till main**
```bash
# VARNING: Gör detta bara om INGEN annan har pullt!
git push --force-with-lease origin main
```

**Steg 4: Byt till rätt branch**
```bash
git checkout feature/user-login
```

### Lösning - Om andra redan har pullt (säkrare)

Använd `revert` istället för att radera:

```bash
# Skapa rätt branch först
git checkout -b feature/user-login HEAD~1
git push origin feature/user-login

# Byt tillbaka till main
git checkout main

# Skapa en revert-commit
git revert HEAD
git push origin main
```

Detta skapar en ny commit som ångrar din ändring, istället för att radera historik.

**Påverkan på historik:**
```
main:  A---B---C---C' (C' ångrar C)
feature/user-login:  A---B---C (originalet)
```

**Kommunicera med teamet:**
```
"Sorry, jag pushade till main av misstag. Jag har reverterat det nu.
Den riktiga featuren finns i feature/user-login."
```

## Scenario D: Jobbade direkt på main utan att brancha

### Situationen
```bash
git status
# On branch main
# Changes not staged for commit:
#   modified: UserService.java
#   modified: LoginController.java
```
Du har ändrat filer men inte committat än - och inser att du är på main!

### Lösning - Flytta uncommitted changes

**Metod 1: Stash och skapa branch**
```bash
# Spara ändringar tillfälligt
git stash

# Skapa och byt till rätt branch
git checkout -b feature/user-login

# Återställ dina ändringar
git stash pop
```

**Metod 2: Skapa branch direkt (enklare!)**
```bash
# Skapar branch OCH tar med uncommitted changes
git checkout -b feature/user-login

# Dina ändringar följer med automatiskt!
```

**I IntelliJ:**
1. Längst ner till höger: Klicka på "main"
2. Välj `New Branch...`
3. Döp den till `feature/user-login`
4. Bocka i "Checkout branch"
5. Klicka "Create"
6. Dina uncommitted changes följer automatiskt med!

### Påverkan
```
main:                A---B (oförändrad)
feature/user-login:  A---B (dina uncommitted changes finns här nu)
```

## Specialfall: Flytta commits mellan branches

### Situationen
Du har två feature-branches och råkade commita på fel en:
```
feature/login:     A---B---C (C hör hemma i feature/profile)
feature/profile:   A---B
```

### Lösning - Cherry-pick till rätt branch
```bash
# Byt till rätt branch
git checkout feature/profile

# Kopiera commiten
git cherry-pick C

# Ta bort från fel branch
git checkout feature/login
git reset --hard HEAD~1
```

**I IntelliJ:**
1. Öppna Git-loggen
2. Byt till `feature/profile`
3. Högerklicka på commit C → `Cherry-Pick`
4. Byt till `feature/login`
5. Reset till HEAD~1 (se scenario 3)

### Påverkan på historik
```
feature/login:     A---B
feature/profile:   A---B---C' (C' är kopia av C med nytt ID)
```

## Vanliga fallgropar

### ❌ Glömmer kolla branch före commit
```bash
# Vana gör att man bara committar utan att tänka
git commit -m "Add feature"  # På fel branch!
```
**Lösning:** Gör det till vana att alltid köra `git status` först.

### ❌ Force push till main
Detta kan förstöra andras arbete om de redan pullt.

**Lösning:** Använd `revert` istället på delade branches.

### ❌ Tror att stash är samma som commit
Stash är temporär lagring - glöm inte `stash pop`!

**Lösning:** Kom ihåg att återställa stashen, eller använd `checkout -b` direkt.

### ❌ Cherry-pick utan att ta bort originalet
Nu finns commiten på BÅDA branches.

**Lösning:** Kom ihåg att reset:a från den felaktiga branchen.

## Verifiering - hur vet du att det gick bra?

```bash
# Kolla att du är på rätt branch
git branch
# Ska visa * vid feature/user-login

# Kolla att commits finns på rätt ställe
git log --oneline --all --graph

# Kolla att main är ren
git checkout main
git log --oneline -3
# Ska INTE visa dina nya commits

# Kolla remote
git fetch
git branch -a
# Verifiera vilka branches som finns på remote
```

**I IntelliJ:**
- Kolla branch-namn längst ner till höger
- Öppna Git-loggen och titta på "All branches"
- Verifiera att commits är på rätt branch

## Preventiva åtgärder

✅ **Visa branch i terminal-prompt**
```bash
# Lägg till i ~/.bashrc eller ~/.zshrc
parse_git_branch() {
  git branch 2> /dev/null | sed -e '/^[^*]/d' -e 's/* \(.*\)/(\1)/'
}
PS1="\u@\h \W \$(parse_git_branch) $ "
```

✅ **Git hooks som varnar**
Skapa `.git/hooks/pre-commit`:
```bash
#!/bin/bash
branch=$(git symbolic-ref HEAD | sed -e 's,.*/\(.*\),\1,')
if [ "$branch" = "main" ]; then
  echo "You are on main! Are you sure? (y/n)"
  read response
  if [ "$response" != "y" ]; then
    exit 1
  fi
fi
```

✅ **Branch protection i GitHub/GitLab**
- Kräv pull requests för main
- Blockera direct push till main

✅ **Kolla branch före varje commit**
```bash
# Gör detta till en vana
git status  # Kolla branch
git add .
git commit -m "..."
```

## Generella tips

✅ **Små, frekventa commits** - Lättare att flytta enskilda commits
✅ **Tydliga branch-namn** - Lättare att upptäcka fel branch
✅ **Brancha tidigt** - Skapa branch innan du börjar jobba
✅ **IDE-indikator** - Håll koll på branch-namn i IDE:n
✅ **Git alias** - Skapa alias som visar branch tydligt

## När ska du INTE flytta commits?

- ❌ Om commiten redan är mergad till main
- ❌ Om andra bygger sitt arbete på din commit
- ❌ På publika/production branches utan att kommunicera
- ❌ Om du är osäker - fråga först!

## Ångra om något går fel

```bash
# Hitta tidigare tillstånd
git reflog

# Återställ till före dina ändringar
git reset --hard HEAD@{n}  # där n är rätt position från reflog

# Eller återställ en specifik branch
git checkout main
git reset --hard origin/main  # Återställ från remote
```

**I IntelliJ:**
`Git → Uncommit...` stegvis, eller använd Git-loggen för att reset:a

## Rekommendation

**Innan commit:**
1. Kolla branch: `git status` eller titta i IDE
2. Skapa rätt branch om du inte redan har gjort det
3. Committa med ro

**Efter upptäckt av fel:**
1. Paniksläck inte - det går alltid att fixa
2. Har du inte pushat? → Enkelt med branch + reset
3. Har du pushat? → Kommunicera och överväg revert

Med dessa verktyg kan du alltid flytta commits till rätt plats!