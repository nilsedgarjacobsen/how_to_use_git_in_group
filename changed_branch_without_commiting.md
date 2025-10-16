# Scenario 5: Jag har uncommitted changes och behöver byta branch

## Situationen
Du är mitt i arbetet med ändringar som du inte committat än, och plötsligt behöver du:
- Byta till en annan branch (t.ex. för att granska en PR)
- Hämta uppdateringar från main
- Snabbt fixa en kritisk bugg
- Hjälpa en kollega med något i en annan branch

**Problemet:** Git låter dig inte alltid byta branch med uncommitted changes:
```bash
git checkout main
# error: Your local changes to the following files would be overwritten by checkout:
#   UserService.java
# Please commit your changes or stash them before you switch branches.
```

## Varför uppstår detta problem?

Git skyddar dina ändringar från att skrivas över. Om filen du ändrat ser olika ut i målbranchen, riskerar dina ändringar att förloras.

**När Git TILLÅTER branch-byte:**
- Filen du ändrat är identisk i båda branches
- Dina ändringar kan "flyttas över" utan konflikt

**När Git BLOCKERAR branch-byte:**
- Filen ser olika ut i målbranchen
- Risk för att dina ändringar skrivs över

## Tre huvudmetoder för att hantera detta

### Metod 1: Stash (spara tillfälligt)
**Användning:** När du vill komma tillbaka till samma ändrigar senare

### Metod 2: Commit (spara permanent)
**Användning:** När ändringarna är klara eller du vill spara dem som en checkpoint

### Metod 3: Discard (kasta bort)
**Användning:** När ändringarna var experiment eller inte behövs

## Metod 1: Stash - Spara ändringar tillfälligt

### Vad är stash?
Stash är som en temporär låda där du lägger dina ändringar medan du gör något annat. Tänk på det som "Ctrl+X" (klipp ut) för dina ändringar.

### Grundläggande stash-workflow

**Med Git-kommandon:**
```bash
# 1. Spara dina uncommitted changes
git stash

# eller med ett beskrivande meddelande
git stash push -m "Work in progress on login validation"

# 2. Byt branch och gör vad du behöver
git checkout main
# ... arbeta här ...

# 3. Återvänd till din ursprungliga branch
git checkout feature/user-login

# 4. Återställ dina ändringar
git stash pop
# eller om du vill behålla stashen (ta kopia istället)
git stash apply
```

**I IntelliJ IDEA:**
1. `Git → Uncommit...` (eller `Ctrl+Alt+Z`)
2. Eller: `Git → Commit` → längst ner till höger finns "Stash Changes"
3. Skriv ett meddelande som beskriver ändringarna
4. Klicka "Create Stash"
5. Byt branch (IntelliJ gör detta automatiskt om du försöker checka ut)
6. För att återställa: `Git → Unstash Changes...` → välj din stash → "Pop Stash"

### Skillnad mellan pop och apply

```bash
git stash pop     # Återställer OCH tar bort stashen
git stash apply   # Återställer men BEHÅLLER stashen
```

**Använd pop när:**
- Du är klar med ändringarna på den andra branchen
- Du vill ha "clean slate"

**Använd apply när:**
- Du vill applicera samma ändringar på flera branches
- Du vill behålla stashen som backup

### Hantera flera stashes

```bash
# Se alla sparade stashes
git stash list
# Output:
# stash@{0}: WIP on feature/login: abc123 Add validation
# stash@{1}: On main: def456 Fix urgent bug
# stash@{2}: WIP on feature/profile: ghi789 Update UI

# Återställ en specifik stash
git stash pop stash@{1}

# Se vad en stash innehåller
git stash show -p stash@{0}

# Ta bort en stash
git stash drop stash@{0}

# Ta bort alla stashes
git stash clear
```

**I IntelliJ:**
`Git → Unstash Changes...` visar alla stashes med beskrivningar

### Stash med både staged och unstaged changes

```bash
# Stasha allt (staged + unstaged + untracked)
git stash -u  # eller --include-untracked

# Stasha bara staged changes
git stash --staged

# Stasha allt utom staged
git stash --keep-index
```

### Påverkan på working directory

```
Före stash:
Working Directory: UserService.java (ändrad), LoginController.java (ändrad)
Staging Area: (tom)

Efter stash:
Working Directory: (ren, som senaste commit)
Stash stack: [dina ändringar sparade här]

Efter stash pop:
Working Directory: UserService.java (ändrad), LoginController.java (ändrad)
Stash stack: (tom)
```

## Metod 2: Commit - Spara permanent

### När ska du committa istället för stasha?

**Committa när:**
- Du har nått en naturlig checkpoint
- Ändringarna är logiskt kompletta (även om funktionen inte är klar)
- Du vill ha en säkerhetskopia i historiken
- Du ska vara borta länge (stash kan lätt glömmas bort)

### WIP commits (Work In Progress)

```bash
# Committa som "work in progress"
git add .
git commit -m "WIP: Login validation (not complete)"

# Byt branch
git checkout main
# ... arbeta ...

# Återvänd
git checkout feature/user-login

# Fortsätt jobba och amenda senare
# ... gör ändringar ...
git add .
git commit --amend -m "Add login validation"
```

**Fördel:** Säkrare än stash - försvinner aldrig "av misstag"

**Nackdel:** Rörig historik om du glömmer städa upp WIP-commits

**I IntelliJ:**
- Committa normalt med "WIP: " i början av meddelandet
- När du är klar: `Git → Uncommit...` → "Amend" → städa upp meddelandet

### Cleanup av WIP commits före merge

Använd interactive rebase för att squasha WIP-commits:

```bash
git rebase -i HEAD~3
```
Ändra `pick` till `squash` för WIP-commits:
```
pick abc123 Add login feature
squash def456 WIP: Fix validation
squash ghi789 WIP: Add tests
```

## Metod 3: Discard - Kasta bort ändringar

### När ska du kasta ändringar?

**Discard när:**
- Ändringarna var bara experiment
- Du råkade ändra fel fil
- Du vill börja om från början
- Ändringarna är obsolete efter att ha läst kod i annan branch

### Kasta alla uncommitted changes

**Med Git-kommandon:**
```bash
# TA BORT alla uncommitted changes (VARNING: kan ej ångras!)
git restore .

# Eller gamla sättet
git checkout -- .

# Inklusive untracked filer
git clean -fd
```

**I IntelliJ:**
1. Högerklicka på projektet/filen i Project-vyn
2. `Git → Revert...` eller `Rollback...`
3. Bekräfta (OBS: Detta kan inte ångras!)

### Kasta ändringar i specifika filer

```bash
# Återställ en specifik fil
git restore src/UserService.java

# Ta bort från staging area (men behåll ändringarna i filen)
git restore --staged src/UserService.java
```

**I IntelliJ:**
Högerklicka på filen → `Git → Revert...`

### Säker discard med stash först

Om du är osäker, gör en stash först (som backup):

```bash
# Säkerhetskopiera först
git stash

# Testa att allt fungerar
# ... om något gick fel ...

# Återställ från stash
git stash pop
```

## Specialfall: Konflikter vid stash pop

### Situationen
```bash
git stash pop
# CONFLICT (content): Merge conflict in UserService.java
```

Detta händer när filen ändrats både i stashen och i din nuvarande branch.

### Lösning

**Steg 1: Lös konflikten som vanligt**
```bash
# Öppna filen och lös konfliktmarkeringar (<<<<<<<, =======, >>>>>>>)
# Se scenario 2 för detaljer
```

**Steg 2: Markera som löst**
```bash
git add UserService.java
```

**Steg 3: Ta bort stashen manuellt**
```bash
git stash drop
```

**I IntelliJ:**
IntelliJ öppnar automatiskt merge-verktyget vid konflikt

## Vanliga fallgropar

### ❌ Glömmer att pop:a stash
```bash
git stash  # Senare: "Vart tog mina ändringar vägen?"
```
**Lösning:** Använd `git stash list` regelbundet. Eller committa istället!

### ❌ Stashar på fel branch
```bash
git stash          # På feature-branch
git checkout main
git stash pop      # Applicerar på main istället!
```
**Lösning:** Kolla alltid branch innan `stash pop`

### ❌ Kör git clean utan att förstå
```bash
git clean -fd  # TAR BORT alla untracked filer!
```
**Lösning:** Kör `git clean -n` först (dry-run) för att se vad som tas bort

### ❌ Multiple stashes utan beskrivningar
```bash
git stash  # "WIP on feature/login"
git stash  # "WIP on feature/login"
# Vilken var vilken?
```
**Lösning:** Använd alltid `git stash push -m "beskrivning"`

## Scenario-exempel: Akut buggfix

### Situationen
Du jobbar i `feature/user-login` med uncommitted changes, och får veta att det finns en kritisk bugg i produktion som du måste fixa NU.

### Lösning steg-för-steg

```bash
# 1. Spara ditt pågående arbete
git stash push -m "Login validation work in progress"

# 2. Byt till main och skapa hotfix-branch
git checkout main
git pull origin main
git checkout -b hotfix/critical-bug

# 3. Fixa buggen
# ... redigera filer ...
git add .
git commit -m "Fix critical null pointer in payment flow"

# 4. Pusha hotfixen
git push origin hotfix/critical-bug
# Skapa PR och merga

# 5. Återvänd till ditt arbete
git checkout feature/user-login
git stash pop

# 6. Uppdatera din branch med bugfixen (om relevant)
git fetch origin
git merge origin/main
```

**I IntelliJ:**
1. Stash via Uncommit dialog
2. Byt branch via branch-menyn längst ner till höger
3. Fixa bugg, committa, pusha
4. Byt tillbaka till feature-branch
5. Unstash via Git → Unstash Changes

## Verifiering - hur vet du att det gick bra?

```bash
# Kolla working directory status
git status
# Ska visa dina återställda ändringar

# Kolla att stashen är borta (om du poppade)
git stash list
# Ska inte visa stashen längre (om du använde pop)

# Kolla att du är på rätt branch
git branch
# Ska visa * vid rätt branch

# Verifiera att koden kompilerar
./gradlew build  # eller ditt build-kommando
```

**I IntelliJ:**
- Git-fönstret visar uncommitted changes
- Projektet bygger utan fel
- Du är på rätt branch (längst ner till höger)

## Alternativ metod: Temporary commit

Om du inte gillar stash, kan du göra en temporary commit:

```bash
# Skapa temporär commit
git add .
git commit -m "TEMP - work in progress"

# Byt branch och gör ditt
git checkout main
# ...

# Återvänd
git checkout feature/user-login

# Ångra commiten (behåll ändringarna)
git reset HEAD~1
```

**Fördel:** Enklare att förstå för nybörjare
**Nackdel:** Risk att glömma "uncommita", skapar rörig historik

## Generella tips

✅ **Stasha med beskrivningar** - Alltid `git stash push -m "..."`
✅ **Committa ofta** - Minskar behovet av stash
✅ **Kolla stash-listan regelbundet** - `git stash list`
✅ **Rensa gamla stashes** - Behåll inte stashes i flera dagar
✅ **Använd IntelliJ:s UI** - Enklare än kommandon för stash
✅ **Testa efter pop/apply** - Kör build och tester

## När ska du använda vilken metod?

**Använd Stash när:**
- Snabb switch för att granska PR
- Akut bugfix som avbryter ditt arbete
- Vill testa något i en annan branch temporärt
- Ändringar är ofullständiga/experimentella

**Använd Commit när:**
- Ändringar representerar en logisk enhet (även om ofullständig)
- Du ska vara borta länge
- Du vill ha checkpoint i historiken
- Ändringarna är värdefulla

**Använd Discard när:**
- Ändringar var bara tester
- Du vill börja om
- Ändringarna är obsolete

## Ångra om något går fel

### Om du stashade av misstag
```bash
# Återställ senaste stash
git stash pop

# Eller hitta i listan
git stash list
git stash apply stash@{n}
```

### Om du discardade av misstag
Tyvärr finns ingen enkel ångra-knapp. Men:
- IntelliJ:s Local History kan rädda dig: `Right-click → Local History → Show History`
- IDE:n sparar ofta ändringar även efter discard

### Om stash pop skapade konflikt
```bash
# Avbryt helt
git reset --hard HEAD
git checkout .

# Stashen finns fortfarande kvar i listan
git stash list
```

## Rekommendation

För de flesta situationer: **Använd stash**. Det är säkert, reversibelt, och precis det Git-verktyg som är designat för detta scenario.

För längre arbetspauser: **Committa** istället, även om det är WIP. Stashes glöms lätt bort.

Med dessa verktyg kan du smidigt växla mellan branches utan att förlora ditt arbete!