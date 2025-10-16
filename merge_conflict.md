# Scenario 2: Jag får merge-konflikter när jag ska merga/rebasea

## Situationen
Du försöker merga main till din feature-branch (eller rebasea på main) och Git säger:
```
CONFLICT (content): Merge conflict in src/UserService.java
Automatic merge failed; fix conflicts and then commit the result.
```

## Vad är en merge-konflikt?
En konflikt uppstår när **samma rader** i samma fil har ändrats på olika sätt i båda branches. Git vet inte vilken version som är rätt och ber dig välja.

**Exempel:**
```
main-branch:        Ändrade rad 42 till "return user.getEmail();"
din feature-branch: Ändrade rad 42 till "return user.getPrimaryEmail();"
```
Git kan inte automatiskt avgöra vilken som är korrekt.

## Varför uppstår konflikter?

- **Parallellt arbete** - Två personer ändrar samma kod
- **Stora branches** - Ju längre branch, desto fler konflikter
- **Stora commits** - Svårt att se exakt vad som ändrades
- **Omstrukturering** - Någon flyttade kod du också ändrade
- **Dålig kommunikation** - Teamet koordinerar inte vem som jobbar var

## Förstå konfliktmarkeringar

När en konflikt uppstår, markerar Git filen så här:

```java
public String getUserEmail(User user) {
<<<<<<< HEAD (Current Change)
    return user.getEmail();
=======
    return user.getPrimaryEmail();
>>>>>>> feature/user-login (Incoming Change)
}
```

**Förklaring:**
- `<<<<<<< HEAD` - Din nuvarande version (vad som finns i din branch)
- `=======` - Skiljelinjen
- `>>>>>>> feature/user-login` - Den inkommande ändringen (från main)
- Allt däremellan är de motstridiga ändringarna

## Steg-för-steg: Lösa konflikter med Git-kommandon

### 1. Identifiera konfliktfiler
```bash
git status
```
Output visar:
```
Unmerged paths:
  (use "git add <file>..." to mark resolution)
        both modified:   src/UserService.java
        both modified:   src/LoginController.java
```

### 2. Öppna en konfliktfil i din editor
```bash
# Öppna filen och leta efter <<<<<<< markeringar
code src/UserService.java  # VS Code
idea src/UserService.java  # IntelliJ från kommandoraden
```

### 3. Bestäm vad som är rätt
Du har tre val:
- **Accept Current** - Behåll din version (ovanför `=======`)
- **Accept Incoming** - Använd main:s version (under `=======`)
- **Accept Both** - Kombinera båda (manuellt)
- **Manual Edit** - Skriv helt ny kod

### 4. Ta bort konfliktmarkeringarna
Redigera filen så den ser ut som du vill ha den, **utan** `<<<<<<<`, `=======`, eller `>>>>>>>`.

**Exempel - efter redigering:**
```java
public String getUserEmail(User user) {
    // Vi väljer den nya metoden från feature-branchen
    return user.getPrimaryEmail();
}
```

### 5. Markera filen som löst
```bash
git add src/UserService.java
```

### 6. Fortsätt merge/rebase

**Om du mergade:**
```bash
# När alla konflikter är lösta
git commit  # Git skapar automatiskt ett merge-commit meddelande
```

**Om du rebasade:**
```bash
# Efter varje löst konflikt
git add <fil>
git rebase --continue

# Upprepa för varje commit som har konflikter
```

### 7. Pusha (om du rebasade)
```bash
git push --force-with-lease origin feature/user-login
```

## Steg-för-steg: Lösa konflikter i IntelliJ IDEA

### 1. IntelliJ upptäcker konflikten
När en konflikt uppstår, visar IntelliJ en dialog: **"Files Merged with Conflicts"**

### 2. Öppna Merge-verktyget
- Klicka på **"Merge..."** i dialogen, ELLER
- Högerklicka på den röda filen i Project-vyn → **Git → Resolve Conflicts**

### 3. IntelliJ:s 3-panel merge-vy
IntelliJ visar tre paneler:

```
┌─────────────────┬─────────────────┬─────────────────┐
│   YOUR VERSION  │     RESULT      │  THEIR VERSION  │
│   (Current)     │   (Editerad)    │   (Incoming)    │
│                 │                 │                 │
│  Vänster panel  │  Mitten panel   │  Höger panel    │
└─────────────────┴─────────────────┴─────────────────┘
```

**Vänster panel:** Din version (från feature-branchen)
**Höger panel:** Deras version (från main)
**Mitten panel:** Resultatet (detta är vad som sparas)

### 4. Välj ändringar
Du kan:
- Klicka på **">>"** eller **"<<"** för att acceptera en sida
- Klicka på **"X"** för att ignorera en ändring
- Redigera direkt i mitten-panelen för manuella ändringar

**Användbara knappar:**
- **"Accept Left"** - Ta din version
- **"Accept Right"** - Ta main:s version
- **"Accept Both"** - Kombinera båda (i ordning)

### 5. Spara och fortsätt
- Klicka **"Apply"** när du är nöjd
- Upprepa för alla konfliktfiler
- IntelliJ markerar automatiskt filer som lösta

### 6. Slutför merge/rebase
- **För merge:** Klicka "Commit" (IntelliJ skapar merge-commit automatiskt)
- **För rebase:** IntelliJ fortsätter automatiskt till nästa konflikt

### 7. Pusha (vid rebase)
Klicka på gröna uppåtpilen → bocka i "Force Push"

## Vanliga konflikttyper och hur man löser dem

### Typ 1: Båda ändrade samma logik
```java
<<<<<<< HEAD
return user.getEmail().toLowerCase();
=======
return user.getPrimaryEmail();
>>>>>>> main
```
**Lösning:** Kombinera båda intentionerna:
```java
return user.getPrimaryEmail().toLowerCase();
```

### Typ 2: En lade till, en tog bort
```java
<<<<<<< HEAD
// Din branch tog bort metoden helt
=======
public void oldMethod() {
    // main har kvar gammal metod
}
>>>>>>> main
```
**Lösning:** Undersök varför den togs bort. Om den verkligen är gammal, ta bort den. Om main behöver den, behåll den och flagga med teamet.

### Typ 3: Import-konflikter
```java
<<<<<<< HEAD
import com.example.UserService;
import com.example.EmailService;
=======
import com.example.EmailValidator;
import com.example.UserService;
>>>>>>> main
```
**Lösning:** Behåll alla imports och låt IntelliJ sortera dem (`Ctrl+Alt+O`):
```java
import com.example.EmailService;
import com.example.EmailValidator;
import com.example.UserService;
```

### Typ 4: Whitespace/formattering
```java
<<<<<<< HEAD
public void save(User user){
=======
public void save(User user) {
>>>>>>> main
```
**Lösning:** Följ projektets kodstil (ofta med space innan `{`). Använd IDE:ns auto-format efter att löst konflikten.

## Så påverkas git-historiken

### Vid merge med konflikter
```
main:           A---B---C
                 \       \
feature-branch:   D---E---M (M = merge commit med konfliktlösningar)
```
Merge-commiten (M) innehåller dina manuella konfliktlösningar.

### Vid rebase med konflikter
```
Före:  main:    A---B---C
                 \
       feature:   D---E

Efter: main:    A---B---C
                         \
       feature:           D'---E' (D' och E' innehåller konfliktlösningar)
```
Varje commit (D', E') reappliceras med dina konfliktlösningar integrerade.

## Tips för att undvika konflikter

✅ **Uppdatera ofta från main** - Små konflikter är lättare än stora
✅ **Små, fokuserade commits** - Lättare att se vad som ändrades
✅ **Kommunicera med teamet** - "Jag refaktorerar UserService idag"
✅ **Korta feature-branches** - Färre ändringar = färre konflikter
✅ **Atomic commits** - En sak per commit
✅ **Kör tester efter** - Säkerställ att konfliktlösningen fungerar

## Vanliga fallgropar

### ❌ Glömmer ta bort konfliktmarkeringar
```java
// Filen innehåller fortfarande <<<<<<<
git add file.java  // Committar bruten kod!
```
**Lösning:** Sök efter `<<<<<<<` i projektet innan commit.

**I IntelliJ:** `Edit → Find → Find in Files` → sök efter `<<<<<<<`

### ❌ Väljer alltid "Accept Yours"
Du missar viktiga ändringar från main och kan skriva över andras arbete.

**Lösning:** Läs och förstå båda ändringarna. Fråga kollegan vid osäkerhet.

### ❌ Löser konflikten utan att förstå koden
Resultatet kan vara logiskt felaktigt även om Git är nöjd.

**Lösning:** Om du inte förstår, fråga personen som skrev koden i main.

### ❌ Glömmer köra tester
Konfliktlösningen kan vara syntaktiskt korrekt men ändå bryta logik.

**Lösning:** Kör alltid `npm test` / `./gradlew test` efter konfliktlösning.

## Verifiering - hur vet du att det gick bra?

```bash
# Inga fler konflikter
git status  # Ska inte visa "Unmerged paths"

# Projektet bygger
./gradlew build  # eller ditt build-kommando

# Testerna går igenom
npm test

# Kolla vad som faktiskt ändrades
git diff HEAD~1  # (efter merge commit)
```

**I IntelliJ:**
- Inga röda filer i Project-vyn
- Bygget lyckas (`Ctrl+F9`)
- Testerna går igenom (`Ctrl+Shift+F10`)

## När du är osäker på vilken version som är rätt

### 1. Kolla Git-loggen
```bash
# Se vem som ändrade raden och varför
git log -p src/UserService.java
```

**I IntelliJ:** Högerklicka på raden → **Git → Show History for Selection**

### 2. Kolla original commits
```bash
# Se hela commiten som introducerade ändringen
git show <commit-hash>
```

### 3. Fråga upphovsmannen
```bash
# Se vem som skrev koden
git blame src/UserService.java
```

**I IntelliJ:** Högerklicka på raden → **Git → Annotate with Git Blame**

### 4. Testa båda versionerna
Skapa en branch där du testar den andra lösningen:
```bash
# Spara ditt nuvarande arbete
git stash

# Testa den andra versionen
git checkout --theirs src/UserService.java
./gradlew test

# Återgå till din lösning
git checkout --ours src/UserService.java
git stash pop
```

## Ångra om något går fel

### Avbryt merge/rebase helt
```bash
# Under pågående merge
git merge --abort

# Under pågående rebase
git rebase --abort
```

**I IntelliJ:** `Git → Abort Merge/Rebase` från menyn

### Ångra redan genomförd merge
```bash
# Om du INTE pushat än
git reset --hard HEAD~1

# Om du redan pushat (skapa ny commit som ångrar)
git revert -m 1 HEAD
```

**VIKTIGT:** Vid revert av merge, `-m 1` säger åt Git vilken förälder som är "main".

## Rekommendation för nybörjare

1. **Ta det lugnt** - Konflikter är normala, inte fel
2. **Läs båda versionerna** - Förstå vad varje sida försöker göra
3. **Testa efter varje konflikt** - Bygg och kör tester innan du fortsätter
4. **Be om hjälp** - Om du inte förstår koden, fråga upphovsmannen
5. **Öva i IntelliJ** - Merge-verktyget är mycket bättre än manuell redigering

Med övning blir det enklare att snabbt identifiera och lösa konflikter!