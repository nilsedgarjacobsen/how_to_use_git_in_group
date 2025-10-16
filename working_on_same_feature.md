# Scenario 7: Två personer behöver jobba på samma feature samtidigt

## Situationen
Du och en kollega behöver samarbeta på samma feature-branch:
- Ni delar en feature som är för stor för en person
- Du behöver fortsätta på någon annans påbörjade arbete
- Pair programming där ni växlar vem som committar
- En måste lämna över sitt arbete till en annan

**Utmaningen:** Koordinera ändringar utan att skriva över varandras arbete eller skapa konstiga konflikter.

## Varför är detta svårare än vanligt Git-arbete?

**Normal workflow:**
```
main ← Du arbetar isolerat i din branch
```

**Delad branch:**
```
feature-branch ← Du arbetar ← Kollega arbetar
                  ↓              ↓
              Konflikter, överskrivning, förvirring
```

**Risker:**
- Skriva över varandras commits
- Force push som förstör andras arbete
- Merge-konflikter som är svåra att lösa
- Förlorade ändringar

## Grundläggande workflow för delad branch

### Regel #1: Kommunicera konstant

```
"Jag committar nu och pushar UserService.java"
"Jag håller på med LoginController, vänta 10 min med push"
"Jag har pushat, du kan pulla nu"
```

### Regel #2: Pull ofta

```bash
# VARJE gång före du börjar jobba
git pull origin feature/user-login

# VARJE gång före du pushar
git pull origin feature/user-login
git push origin feature/user-login
```

### Regel #3: Aldrig force push på delade branches

```bash
# ALDRIG göra detta på delad branch!
git push --force origin feature/user-login

# Om du MÅSTE (sällsynt), kommunicera först!
```

## Best practice: Pull --rebase

### Problemet med vanlig pull (merge)

```bash
git pull origin feature/user-login
```

Detta skapar merge-commits när båda har nya commits:
```
Före:  A---B---C (dina commits)
       A---B---D (kollegans commits)

Efter: A---B---C---M (M = merge-commit)
            \     /
             D----

Resultat: Rörig historik med många merge-commits
```

### Lösningen: Pull med rebase

```bash
git pull --rebase origin feature/user-login
```

Detta "flyttar" dina commits ovanpå kollegans:
```
Före:  A---B---C (dina commits)
       A---B---D (kollegans commits)

Efter: A---B---D---C' (C' applicerad ovanpå D)

Resultat: Ren, linjär historik
```

### Konfigurera pull --rebase som standard

```bash
# För denna branch
git config pull.rebase true

# För alla branches
git config --global pull.rebase true

# Eller för endast aktuellt repo
git config pull.rebase true
```

**I IntelliJ:**
1. `Settings/Preferences` → `Version Control` → `Git`
2. Under "Update method": välj "Rebase"
3. Nu använder `Update Project` (Ctrl+T) automatiskt rebase

## Workflow steg-för-steg

### Scenario: Du och kollega jobbar på samma branch

**Din morgon-rutin:**
```bash
# 1. Byt till feature-branchen
git checkout feature/user-login

# 2. Hämta senaste från remote
git pull --rebase origin feature/user-login

# 3. Börja jobba
# ...
```

**När du är klar med en del:**
```bash
# 1. Committa dina ändringar
git add .
git commit -m "Add login validation"

# 2. Hämta kollegans senaste (innan du pushar!)
git pull --rebase origin feature/user-login

# 3. Lös eventuella konflikter (se nedan)

# 4. Pusha
git push origin feature/user-login

# 5. Kommunicera: "Jag har pushat login validation"
```

**I IntelliJ:**
1. `Git → Commit` och committa
2. `Git → Pull` (eller `Ctrl+T`) - välj Rebase
3. Lös konflikter om de uppstår
4. `Git → Push` (eller `Ctrl+Shift+K`)

## Hantera konflikter vid pull --rebase

### När konflikter uppstår

```bash
git pull --rebase origin feature/user-login
# CONFLICT (content): Merge conflict in UserService.java
```

### Lösning steg-för-steg

```bash
# 1. Hitta konfliktfiler
git status
# Visar: both modified: UserService.java

# 2. Öppna och lös konflikten (se scenario 2)
# Redigera filen, ta bort <<<<<<<, =======, >>>>>>>

# 3. Markera som löst
git add UserService.java

# 4. Fortsätt rebasen
git rebase --continue

# 5. Om fler konflikter: upprepa steg 2-4
# 6. När alla konflikter lösta: pusha
git push origin feature/user-login
```

**VIKTIGT vid rebase-konflikter:**
- "Theirs" = kollegans version (redan på remote)
- "Yours" = din version (lokala commits)

**I IntelliJ:**
IntelliJ öppnar automatiskt merge-verktyget vid konflikter. Efter lösning: `Git → Continue Rebasing`

### Om rebasen blir för komplicerad

```bash
# Avbryt rebasen
git rebase --abort

# Försök merge istället (enklare men rörigare historik)
git pull origin feature/user-login

# Lös eventuella konflikter
git add .
git commit

# Pusha
git push origin feature/user-login
```

## Arbetsdelning: Olika filer

### Best case: Ni jobbar på olika filer

```bash
Du:      UserService.java, UserRepository.java
Kollega: LoginController.java, SecurityConfig.java
```

**Resultat:** Nästan inga konflikter!

```bash
git pull --rebase origin feature/user-login
# Updating abc123..def456
# Fast-forward (inga konflikter!)
```

### Kommunicera vem som jobbar var

**I chatten:**
```
Du: "Jag tar UserService och allt i /repository"
Kollega: "Ok, jag tar LoginController och /config"
```

**Med kod-kommentarer:**
```java
// TODO @yourname: Implement validation logic
public void validateUser(User user) {
    // Work in progress
}
```

## Arbetsdelning: Samma fil - olika metoder

### Mer utmanande men hanterbart

**Strategier:**
1. **Dela upp vertikalt (olika metoder)**
   ```
   Du:      loginUser(), validateCredentials()
   Kollega: logoutUser(), refreshSession()
   ```

2. **Dela upp horisontellt (olika lager)**
   ```
   Du:      Affärslogik
   Kollega: Testfall
   ```

3. **Tidsmässig koordinering**
   ```
   Du:      Jobbar 09-12
   Kollega: Jobbar 13-17
   Pull/push vid överlämning
   ```

## Branching-strategi för större samarbeten

### Alternativ: Sub-branches

Istället för att båda jobba direkt på `feature/user-login`:

```bash
# Skapa personliga sub-branches
git checkout feature/user-login
git checkout -b feature/user-login-alice
git checkout feature/user-login
git checkout -b feature/user-login-bob

# Alice jobbar i sin branch
cd alice-workspace
git checkout feature/user-login-alice
# ... arbete ...
git push origin feature/user-login-alice

# Bob jobbar i sin branch
cd bob-workspace
git checkout feature/user-login-bob
# ... arbete ...
git push origin feature/user-login-bob

# Slå ihop regelbundet till huvudbranchen
git checkout feature/user-login
git merge feature/user-login-alice
git merge feature/user-login-bob
git push origin feature/user-login
```

### Workflow-visualisering

```
feature/user-login (huvudbranch)
    ├── feature/user-login-alice (Alice's arbete)
    └── feature/user-login-bob   (Bob's arbete)

Regelbunden merge tillbaka till feature/user-login
```

**Fördelar:**
- Mindre risk för konflikter
- Tydligare vem som gör vad
- Lättare att granska varandras kod

**Nackdelar:**
- Mer administrativt arbete
- Kräver regelbundna merges

## Pair programming workflow

### Scenario: Växlar vid samma dator

```bash
# Alice committar
git add .
git commit -m "Add user validation (pair: Alice, Bob)"
git push origin feature/user-login

# Bob tar över
git pull --rebase origin feature/user-login
# ... fortsätter jobba ...
git add .
git commit -m "Implement password hashing (pair: Bob, Alice)"
git push origin feature/user-login
```

**Tips för pair programming:**
- Inkludera båda namn i commit-meddelanden
- Commit ofta (varje gång ni byter)
- Pusha efter varje commit

### Remote pair programming

**Med olika datorer:**
```bash
# Alice (driver)
git add .
git commit -m "WIP: Half-way through validation logic"
git push origin feature/user-login

# Bob (navigator → driver)
git pull --rebase origin feature/user-login
# Fortsätter där Alice slutade
git add .
git commit -m "Complete validation logic"
git push origin feature/user-login
```

## Vanliga fallgropar

### ❌ Force push på delad branch

```bash
# Alice gör rebase och force push
git push --force origin feature/user-login

# Bob's lokala branch är nu ur synk!
git pull  # Error: divergent branches
```

**Lösning om det händer (Bob's perspektiv):**
```bash
# Spara lokalt arbete
git stash

# Hämta den force-pushade versionen
git fetch origin
git reset --hard origin/feature/user-login

# Återställ lokalt arbete
git stash pop
```

**Prevention:** Kommunicera ALLTID före force push!

### ❌ Glömmer pulla innan push

```bash
# Du pushar utan att pulla först
git push origin feature/user-login
# error: failed to push some refs
```

**Lösning:**
```bash
git pull --rebase origin feature/user-login
git push origin feature/user-login
```

### ❌ Jobbar på samma kod samtidigt

```bash
# Ni ändrar båda samma rader samtidigt
# → Garanterad konflikt vid pull!
```

**Lösning:** Kommunicera aktivt vad ni jobbar på

### ❌ Stora commits ovanpå varandra

```bash
# Alice committar 500 rader
# Bob committar 500 rader
# → Svåra konflikter
```

**Lösning:** Små, frekventa commits

## Verifiering - hur vet ni att ni är synkade?

```bash
# Kolla status
git status
# Ska visa: "Your branch is up to date with 'origin/feature-user-login'"

# Kolla senaste commits
git log --oneline --graph -5

# Kolla remote
git fetch
git log --oneline origin/feature-user-login -5
# Ska matcha din lokala log
```

**I IntelliJ:**
- Status längst ner: "↑0 ↓0" = synkad
- Git-loggen visar både lokal och remote branch på samma commit

## Kommunikations-protokoll

### Innan du börjar jobba
```
"Jag börjar jobba på UserService nu"
"Kommer du röra den? Annars tar jag den"
```

### När du committar
```
"Committed and pushed validation logic"
"Du kan pulla nu om du vill"
```

### Innan force push (sällsynt!)
```
"Jag behöver rebasea för att fixa historiken"
"Kommer force pusha om 5 min, stasha ditt arbete!"
"Force push klar, ni kan köra: 
 git fetch origin && git reset --hard origin/feature-branch"
```

### När konflikt uppstår
```
"Jag fick konflikt i UserService, jobbar du där?"
"Ja, vänta 2 min så committar jag"
```

## Generella tips för framgångsrikt samarbete

✅ **Kommunicera konstant** - Över-kommunikation är bättre än under-kommunikation
✅ **Små commits** - Lättare att integrera varandras arbete
✅ **Pull före push** - Alltid hämta senaste innan du pushar
✅ **Använd pull --rebase** - Håller historiken ren
✅ **Olika filer om möjligt** - Minskar konflikter drastiskt
✅ **Commit ofta** - Ju kortare tid mellan commits, desto färre konflikter
✅ **Testa efter pull** - Verifiera att integreringen fungerar
✅ **Aldrig force push** - (eller kommunicera först)

## Alternativa arbetssätt

### Alternativ 1: Feature-branching med sub-branches
```
feature/user-login (ej direkt arbete)
    ├── feature/user-login-backend (Person A)
    └── feature/user-login-frontend (Person B)
```

### Alternativ 2: Task-baserade branches
```
feature/user-login-validation (Person A)
feature/user-login-ui (Person B)
→ Båda mergas till develop
```

### Alternativ 3: Mob programming
```
Alla jobbar tillsammans vid en dator/skärm
En person "driver", resten navigerar
Växlar driver var 10-15 min
```

**Rekommendation:** Diskutera med teamet vilken strategi som passar bäst för just er feature!

## När ska ni INTE dela branch?

❌ Om featuren kan delas i oberoende delar
❌ Om ni jobbar i helt olika tidszoner
❌ Om teamet saknar erfarenhet av Git
❌ För stora, långlivade features

**Istället:** Dela upp i flera separata feature-branches som mergas oberoende

## Ångra om något går fel

### Om force push skedde av misstag

**Offrets perspektiv:**
```bash
# Kolla vad som finns på remote
git fetch origin
git log origin/feature-user-login

# Om ditt arbete är nyare/bättre
git push --force-with-lease origin feature-user-login
# (kommunicera först!)

# Om remote är nyare/bättre
git reset --hard origin/feature-user-login
```

### Om du är förvirrad och osäker

```bash
# Skapa backup
git branch backup-$(date +%Y%m%d-%H%M%S)

# Be om hjälp från kollega eller senior utvecklare
```

## Rekommendation

**För mindre features (1-3 dagar):**
→ Dela branch direkt, använd pull --rebase, kommunicera tätt

**För större features (1+ vecka):**
→ Använd sub-branches, merga regelbundet till huvudbranchen

**För mycket stora features:**
→ Dela upp i flera separata feature-branches

**Viktigast:** Kommunikation! Git-verktyg hjälper, men tydlig kommunikation är nyckeln till framgångsrikt samarbete.

Med dessa strategier kan ni effektivt samarbeta utan att trampa varandra på tårna!