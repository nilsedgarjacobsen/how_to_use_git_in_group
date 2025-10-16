# Scenario 10: Jag behöver hitta när och var en bugg introducerades i historiken

## Situationen
En bugg har upptäckts, men du vet inte när den uppstod:
- "Det fungerade förra veckan, men inte nu"
- "Denna feature har alltid varit buggig, men varför?"
- "Vem ändrade denna logik senast?"
- "När slutade det här att fungera?"

**Mål:** Hitta exakt vilket commit som introducerade buggen, så du kan förstå varför och hur man fixar den.

## Varför är detta viktigt?

**Fördelar med att hitta ursprunget:**
- ✅ Förstå buggens kontext och anledning
- ✅ Identifiera relaterad kod som också kan vara buggig
- ✅ Lära sig av misstaget
- ✅ Kontakta rätt person för input
- ✅ Se om buggen är del av ett större problem

**Exempel:**
```
Bug upptäckt: Användare kan inte logga in
När introducerades den? Commit abc123, 2 veckor sedan
Varför? Ändring i password-hashing algoritm
Vem? Alice (hon kan förklara logiken)
Fix: Återställ till gammal algoritm eller fixa den nya
```

## Verktyg för att hitta buggens ursprung

### 1. Git Log - Söka i historiken
### 2. Git Blame - Se vem som skrev varje rad
### 3. Git Bisect - Binärsökning för att hitta buggigt commit
### 4. Git Show - Undersöka specifika commits
### 5. Git Diff - Jämföra versioner

## Metod 1: Git Log - Sök i commit-historiken

### Scenario: Du vet ungefär när buggen uppstod

```bash
# Se commits från senaste veckan
git log --since="1 week ago" --oneline

# Se commits från specifik författare
git log --author="Alice" --oneline

# Sök i commit-meddelanden
git log --grep="password" --oneline

# Kombinera flera filter
git log --since="2 weeks ago" --author="Alice" --grep="login" --oneline
```

**I IntelliJ:**
1. Öppna Git-loggen (`Alt+9`)
2. Använd sökfältet längst upp
3. Filtrera på branch, författare, datum

### Hitta commits som ändrade specifik fil

```bash
# Alla commits som ändrade UserService.java
git log --oneline -- src/UserService.java

# Med diff för att se ändringarna
git log -p -- src/UserService.java

# Med statistik
git log --stat -- src/UserService.java
```

**I IntelliJ:**
Högerklicka på filen → `Git → Show History`

### Hitta commits som ändrade specifik funktion

```bash
# Sök efter commits som nämner "validatePassword"
git log -S "validatePassword" --oneline

# Med diff
git log -S "validatePassword" -p
```

**Flagga `-S`:** Söker efter commits där en sträng lades till eller togs bort

**Exempel output:**
```
abc123 Refactor password validation
def456 Add password complexity check
ghi789 Initial login implementation
```

### Påverkan: Förstå commit-historiken

```
A---B---C---D---E (main)
         \
          F---G (feature-branch, mergad vid D)

Alla dessa commits kan innehålla buggen!
```

## Metod 2: Git Blame - Se vem som skrev varje rad

### Scenario: Du vet VILKEN RAD som är buggig

```bash
# Visa vem som skrev varje rad i filen
git blame src/UserService.java

# Output:
# abc123 (Alice  2024-10-01 10:23:45 +0200  42) if (password.length() > 0) {
# abc123 (Alice  2024-10-01 10:23:45 +0200  43)     return true;
# def456 (Bob    2024-10-15 14:30:22 +0200  44) }
```

**Format:**
```
<commit-hash> (<author> <date> <line-num>) <code>
```

### Med mer detaljer

```bash
# Visa också commit-meddelande
git blame -l src/UserService.java

# Visa email istället för namn
git blame -e src/UserService.java

# Blame specifika rader (42-50)
git blame -L 42,50 src/UserService.java
```

**I IntelliJ:**
Högerklicka på rad-numret i editorn → `Annotate with Git Blame`

**IntelliJ visar:**
- Commit-hash (klicka för att se full commit)
- Författare
- Datum
- Commit-meddelande (hover)

### Djupdykning med blame

```bash
# Problem: Rad 42 är buggig enligt blame
# abc123 (Alice  2024-10-01) if (password.length() > 0) {

# Se full commit
git show abc123

# Se vad som ändrades i commit abc123
git diff abc123^ abc123
```

### Följ rad genom refaktoreringar

```bash
# Följ en rad även när filen bytt namn eller flyttats
git blame -C -C -C src/UserService.java
```

**Flaggor:**
- `-C`: Följ rader mellan filer i samma commit
- `-C -C`: Följ rader när filer skapades
- `-C -C -C`: Följ rader även mellan commits

## Metod 3: Git Bisect - Binärsökning efter buggigt commit

### När använda bisect?

**Perfekt för:**
- ✅ Du vet att buggen INTE fanns i en gammal version
- ✅ Du vet att buggen FINNS i nuvarande version
- ✅ Du kan enkelt testa om buggen finns
- ✅ Många commits mellan fungerande och bruten version

### Hur bisect fungerar

```
Fungerande version (good): commit A
Bruten version (bad): commit Z

Commits mellan: A-B-C-D-E-F-G-H-I-J-Z (10 commits)

Naiv sökning: Testa alla 10 commits = 10 tester
Binärsökning: Testa ~4 commits = 4 tester

Bisect gör binärsökning automatiskt!
```

### Steg-för-steg bisect

**Steg 1: Starta bisect**
```bash
git bisect start
```

**Steg 2: Markera nuvarande (bruten) version**
```bash
# Du är på main, buggen finns här
git bisect bad
```

**Steg 3: Hitta och markera fungerande version**
```bash
# Kolla log för att hitta gammal version
git log --oneline

# Säg till exempel att abc123 fungerade
git bisect good abc123
```

**Git checkar automatiskt ut ett commit i mitten:**
```
Bisecting: 5 revisions left to test after this (roughly 3 steps)
[def456] Some commit message
```

**Steg 4: Testa om buggen finns**
```bash
# Bygg och testa
./gradlew test
# eller kör applikationen manuellt

# Om buggen FINNS:
git bisect bad

# Om buggen INTE finns:
git bisect good
```

**Steg 5: Upprepa**

Git checkar ut nästa commit i mitten. Upprepa tills Git hittar första buggiga commit:

```
abc123def456789... is the first bad commit
commit abc123def456789...
Author: Alice <alice@example.com>
Date:   Mon Oct 1 10:23:45 2024

    Refactor password validation
```

**Steg 6: Avsluta bisect**
```bash
# När du är klar
git bisect reset

# Du är tillbaka på main
```

### Automatisera bisect med script

**Scenario:** Du har ett automatiskt test som kan avgöra om buggen finns

```bash
# Skapa test-script (test-bug.sh)
#!/bin/bash
./gradlew test --tests PasswordValidationTest
exit $?  # 0 = pass (good), non-zero = fail (bad)
```

```bash
# Kör bisect automatiskt
git bisect start
git bisect bad HEAD
git bisect good abc123
git bisect run ./test-bug.sh

# Git kör scriptet för varje commit automatiskt
# och hittar första buggiga commit!
```

**I IntelliJ:**
Ingen direkt UI för bisect - använd terminal i IntelliJ (`Alt+F12`)

### Visualisering av bisect-processen

```
Initial state (10 commits mellan good och bad):
good: A---B---C---D---E---F---G---H---I---J: bad

Steg 1: Testa E (mitten)
A---B---C---D---E---F---G---H---I---J
            ↑ test här
Result: bad → buggen finns i A-E

Steg 2: Testa C
A---B---C---D---E
      ↑ test här
Result: good → buggen finns i C-E

Steg 3: Testa D
    C---D---E
        ↑ test här
Result: bad → D är första buggiga!

Total: 3 tester istället för 10!
```

## Metod 4: Git Show - Undersöka specifika commits

### När du hittat ett misstänkt commit

```bash
# Visa full information om commit
git show abc123

# Visa bara filnamn som ändrades
git show --name-only abc123

# Visa statistik
git show --stat abc123

# Visa ändringarna i specifik fil
git show abc123:src/UserService.java
```

**I IntelliJ:**
Dubbelklicka på commit i Git-loggen för att se detaljer

### Jämför commit med föregående

```bash
# Se vad som ändrades i commit abc123
git diff abc123^ abc123

# Jämför två specifika commits
git diff abc123 def456

# Jämför med current state
git diff abc123 HEAD
```

## Metod 5: Sökning med Git Log + Diff

### Hitta när en specifik kod-rad försvann

```bash
# Hitta när "validatePassword" togs bort
git log -S "validatePassword" --oneline

# Med full diff
git log -S "validatePassword" -p
```

### Hitta när en regex matchade

```bash
# Hitta ändringar som matchar regex
git log -G "password.*validate" --oneline -p
```

**Skillnad mellan -S och -G:**
- `-S "text"`: Exakt textsökning (lades till eller togs bort)
- `-G "regex"`: Regex-sökning (även ändringar)

## Praktiskt exempel: Komplett bugg-jakt

### Scenario: "Login fungerade förra veckan, men inte nu"

**Steg 1: Hitta fungerande version**
```bash
# Kolla commits från förra veckan
git log --since="1 week ago" --until="7 days ago" --oneline

# Hitta senaste från förra veckan
git log --until="7 days ago" --oneline -1
# Output: abc123 Last working commit
```

**Steg 2: Starta bisect**
```bash
git bisect start
git bisect bad HEAD  # Nuvarande version (bruten)
git bisect good abc123  # Förra veckans version (fungerade)
```

**Steg 3: Testa varje version**
```bash
# Git checkar ut ett commit
# Testa manuellt eller automatiskt
./gradlew bootRun
# ... testa login ...

# Om bruten:
git bisect bad

# Om fungerar:
git bisect good

# Upprepa
```

**Steg 4: Git hittar buggen**
```
def456 is the first bad commit
Author: Bob <bob@example.com>
Date: Thu Oct 10 14:23:11 2024

    Refactor authentication flow
```

**Steg 5: Undersök commiten**
```bash
git bisect reset  # Återgå till main

# Se full commit
git show def456

# Se vem som kan förklara
# (Bob skrev det, fråga honom!)
```

**Steg 6: Förstå ändringen**
```bash
# Vad ändrades?
git diff def456^ def456

# Vilka filer?
git show --stat def456

# Hitta relaterade commits
git log --author="Bob" --since="2 weeks ago" --oneline
```

## Vanliga fallgropar

### ❌ Bisect utan tydlig good/bad

```bash
git bisect good <commit-that-might-have-bug>
# Osäker om commiten verkligen var good
# → Bisect hittar fel commit!
```

**Lösning:** Testa noga att good-versionen verkligen fungerar

### ❌ Glömmer bisect reset

```bash
# Efter bisect är klar, glömmer reset
# Du sitter kvar på ett gammalt commit
# Gör ändringar här → Förvirring!
```

**Lösning:** Kör alltid `git bisect reset` när klar

### ❌ Ändrar kod under bisect

```bash
# Under bisect: gör ändringar i filer
# → Bisect-resultatet blir oanvändbart
```

**Lösning:** Gör inga ändringar under bisect

### ❌ Blame visar refaktorerare, inte ursprunglig författare

```bash
git blame UserService.java
# Visar Alice, men hon bara formaterade koden
# Egentliga författaren var Bob
```

**Lösning:** Använd `git blame -w` (ignorera whitespace) eller följ historiken längre

## Verktyg och tips

### IntelliJ Git History superkrafter

**Annotate:**
- Högerklicka på rad → `Annotate with Git Blame`
- Klicka på commit-hash för att se full diff
- Högerklicka på annoterad rad → `Show History for Selection`

**Show History:**
- Högerklicka på fil/metod → `Git → Show History`
- Filtrera på författare, datum, branch
- Se alla ändringar över tid

**Compare with Branch:**
- Högerklicka på fil → `Git → Compare with Branch...`
- Jämför med main, develop, eller annan branch

### Git Extensions och GUI-verktyg

**GitKraken:**
- Visuell git-historik
- Enkel blame-view
- Sökning och filtrering

**SourceTree:**
- File History med visuell graf
- Blame annotations
- Sökfunktioner

**VS Code:**
- GitLens extension
- Inline blame
- File history

### Shell-alias för snabbare arbete

```bash
# Lägg till i ~/.gitconfig eller ~/.bashrc

# Kort log
alias gl="git log --oneline --graph --decorate"

# Sök i commits
alias gsearch="git log --all --grep"

# Blame med ignore whitespace
alias gblame="git blame -w"

# Hitta commits som ändrade en fil
alias gfile="git log --follow --oneline --"
```

## Verifiering - hur vet du att du hittat rätt commit?

```bash
# Kolla commiten noga
git show <commit-hash>

# Verifiera att buggen introducerades här
# Checkout commit före
git checkout <commit-hash>^
./gradlew test  # Fungerar?

# Checkout commiten själv
git checkout <commit-hash>
./gradlew test  # Bruten?

# Om ja, du har hittat rätt!

# Återgå till main
git checkout main
```

## Efter att du hittat buggens ursprung

### 1. Förstå varför
```bash
# Läs commit-meddelandet
git show <commit-hash>

# Se hela PR:en om det finns
# (länk i commit-meddelande eller via GitHub/GitLab)

# Kontakta författaren om oklart
```

### 2. Hitta relaterade problem
```bash
# Andra commits av samma författare runt samma tid
git log --author="Alice" --since="<date>" --until="<date>"

# Andra filer ändrade i samma commit
git show --name-only <commit-hash>
```

### 3. Skapa fix
```bash
# Skapa branch
git checkout -b fix/password-validation-bug

# Gör fix (ofta ändra tillbaka eller förbättra logiken)
# ...

# Committa med referens
git commit -m "Fix password validation bug

Introduced in commit abc123
Root cause: length check was backwards
Fix: Correct validation logic"
```

### 4. Testa att fixen fungerar
```bash
# Kör tester
./gradlew test

# Manuell verifiering
./gradlew bootRun
```

## Generella tips

✅ **Dokumentera fynd** - Skriv ner vilket commit, varför, och hur du fixar
✅ **Kommunicera** - Dela med teamet vad du hittat
✅ **Lär av misstag** - Diskutera i retrospektiv
✅ **Förbättra tester** - Lägg till test som hade fångat buggen
✅ **Commit-meddelanden** - Skriv tydliga meddelanden som hjälper framtida debugging
✅ **Små commits** - Lättare att hitta buggens exakta ursprung

## Förebyggande åtgärder

### Skriv bättre commit-meddelanden

**Dåligt:**
```
git commit -m "fix bug"
```

**Bra:**
```
git commit -m "Fix: Correct password validation length check

Password validation was checking length > 0 instead of >= 8.
This allowed weak passwords.

Fixes #123"
```

### Commit ofta och atomärt

```bash
# Dåligt: En jättecommit
git commit -m "Refactor entire authentication system"
# (svårt att hitta vad som bröt)

# Bra: Flera små commits
git commit -m "Extract password validation to separate method"
git commit -m "Add password complexity check"
git commit -m "Update password hashing algorithm"
# (lätt att bisect och hitta exakt vad som bröt)
```

### Tagga viktiga versioner

```bash
# Tagga releases
git tag -a v1.0.0 -m "Release 1.0.0"
git push origin v1.0.0

# Gör det lätt att hitta "det fungerade i v1.0.0"
git bisect good v1.0.0
```

## Sammanfattning: Vilken metod när?

| Situation | Metod | Verktyg |
|-----------|-------|---------|
| Vet vilken fil | Git Log | `git log -- file.java` |
| Vet vilken rad | Git Blame | IntelliJ Annotate |
| Vet att det fungerade förut | Git Bisect | `git bisect` |
| Söker specifik kod | Git Log -S | `git log -S "text"` |
| Undersöker misstänkt commit | Git Show | `git show <hash>` |
| Jämföra versioner | Git Diff | `git diff A B` |

## Rekommendation

**För snabb sökning:** Börja med Git Blame och Git Log
**För systematisk sökning:** Använd Git Bisect
**För djupdykning:** Kombinera alla metoder

**Workflow:**
1. Reproducera buggen
2. Hitta ungefärlig tidsperiod (git log)
3. Använd bisect för att hitta exakt commit
4. Undersök commit med show/diff
5. Förstå varför (blame, kontakta författare)
6. Fixa och testa

Med dessa verktyg kan du alltid spåra buggar tillbaka till sitt ursprung!