# Scenario 6: Min PR har fått feedback och jag behöver göra ändringar

## Situationen
Du har skapat en Pull Request (PR) och fått feedback från reviewers:
- "Kan du byta namn på denna variabel?"
- "Lägg till tester för edge case X"
- "Refaktorera denna metod"
- "Squasha dina commits innan merge"

Nu behöver du uppdatera din PR med ändringar baserat på feedbacken.

## Varför är detta viktigt?

- **Kodkvalitet** - Feedback förbättrar koden
- **Kunskapsdelning** - Du lär dig av reviewers
- **Teamstandard** - Håller kodbasen konsekvent
- **Git-hygien** - Ren commit-historik underlättar framtida arbete

## Grundläggande workflow för PR-ändringar

### Steg 1: Gör ändringarna lokalt

```bash
# Se till att du är på rätt branch
git checkout feature/user-login

# Gör dina ändringar i koden
# ... redigera filer baserat på feedback ...

# Kolla vad som ändrats
git status
git diff
```

**I IntelliJ:**
- Branch visas längst ner till höger
- Gör ändringar i editorn
- `Git → Show Diff` för att se ändringar

### Steg 2: Committa ändringarna

Du har två huvudalternativ här:

**Alternativ A: Ny commit (enklast)**
```bash
git add .
git commit -m "Address PR feedback: rename variable and add tests"
git push origin feature/user-login
```

**Alternativ B: Amenda senaste commit**
```bash
git add .
git commit --amend --no-edit
git push --force-with-lease origin feature/user-login
```

**I IntelliJ:**
- `Git → Commit` (eller `Ctrl+K`)
- Skriv meddelande
- (För amend: bocka i "Amend commit")
- Klicka "Commit and Push"

### Påverkan på PR

PR:en uppdateras automatiskt när du pushar! 
- GitHub/GitLab visar nya commits
- Reviewers får notis
- CI/CD körs igen

## Scenario A: Små ändringar - ny commit är OK

### Situationen
Reviewer vill att du:
- Fixar en typo
- Lägger till en kommentar
- Byter namn på en variabel

### Lösning - Skapa ny commit

```bash
# Gör ändringarna
# ...

# Committa med tydligt meddelande
git add .
git commit -m "Fix typo in error message"

# Pusha
git push origin feature/user-login
```

### Påverkan på historik
```
Före feedback:
feature-branch: A---B---C

Efter feedback:
feature-branch: A---B---C---D (D = feedback-fix)
```

**Fördelar:**
- Enkelt och säkert
- Tydligt vad som ändrats efter review
- Ingen force push behövs

**Nackdelar:**
- Fler commits i historiken
- Kan bli rörigt med många små fix-commits

## Scenario B: Många små ändringar - squash senare

### Situationen
Du får feedback i flera omgångar:
```
Commit A: Initial implementation
Commit B: Fix typo
Commit C: Add tests
Commit D: Rename variable
Commit E: Fix linting
```

Reviewern säger: "Looks good! Can you squash these commits before merge?"

### Lösning - Interactive rebase för att squasha

```bash
# Starta interactive rebase för de senaste 5 commits
git rebase -i HEAD~5
```

En editor öppnas:
```
pick abc123 Initial implementation
pick def456 Fix typo
pick ghi789 Add tests
pick jkl012 Rename variable
pick mno345 Fix linting
```

Ändra till (behåll första `pick`, resten `squash`):
```
pick abc123 Initial implementation
squash def456 Fix typo
squash ghi789 Add tests
squash jkl012 Rename variable
squash mno345 Fix linting
```

Spara och stäng. Nästa editor öppnas för commit-meddelandet:
```
# This is a combination of 5 commits.
# Edit the message as you like:

Add user login feature

- Implement login validation
- Add comprehensive tests
- Handle edge cases
```

Spara och stäng. Sedan:
```bash
git push --force-with-lease origin feature/user-login
```

**I IntelliJ:**
1. Öppna Git-loggen (`Alt+9`)
2. Högerklicka på commiten FÖRE dina ändringar
3. Välj `Interactively Rebase from Here...`
4. I dialogen: markera commits och klicka "Squash"
5. Redigera commit-meddelandet
6. Klicka "Start Rebasing"
7. Force push via Push-dialogen

### Påverkan på historik
```
Före squash:
feature-branch: A---B---C---D---E

Efter squash:
feature-branch: A' (A' innehåller allt från B, C, D, E)
```

## Scenario C: Behöver ändra mitt i historiken

### Situationen
Reviewer hittar ett problem i commit B, men du har redan commits C och D efter det.

```
A---B (problem här!)---C---D
```

### Lösning - Interactive rebase med edit

```bash
# Starta interactive rebase
git rebase -i HEAD~4  # Gå tillbaka till före A
```

Editor öppnas:
```
pick aaa111 A: Add feature
pick bbb222 B: Add validation (problem här!)
pick ccc333 C: Add tests
pick ddd444 D: Update docs
```

Ändra `pick` till `edit` för commit B:
```
pick aaa111 A: Add feature
edit bbb222 B: Add validation
pick ccc333 C: Add tests
pick ddd444 D: Update docs
```

Spara och stäng. Git stannar vid commit B:
```bash
# Du är nu vid commit B
# Gör dina ändringar
# ...

# Lägg till ändringarna
git add .

# Amenda commiten
git commit --amend --no-edit

# Fortsätt rebasen
git rebase --continue

# Force push
git push --force-with-lease origin feature/user-login
```

**I IntelliJ:**
1. Interactive rebase som ovan
2. Välj "Edit" istället för "Squash"
3. Gör ändringar när rebasen pausar
4. `Git → Continue Rebasing`

### Påverkan på historik
```
Före:
A---B---C---D

Efter:
A---B'---C'---D' (B' är ändrad, C' och D' reapplicerade)
```

## Scenario D: Amenda senaste commit med feedback

### Situationen
Du fick feedback omedelbart efter din commit och vill inkludera ändringarna i samma commit.

### Lösning - Commit --amend

```bash
# Gör ändringar
# ...

# Lägg till ändringar
git add .

# Amenda senaste commit
git commit --amend

# En editor öppnas där du kan ändra meddelandet om du vill
# Eller använd --no-edit för att behålla meddelandet
git commit --amend --no-edit

# Force push
git push --force-with-lease origin feature/user-login
```

**I IntelliJ:**
1. Gör ändringar
2. `Git → Commit` (eller `Ctrl+K`)
3. Bocka i "Amend commit"
4. Klicka "Commit"
5. Push med force (IntelliJ varnar automatiskt)

### Påverkan på historik
```
Före amend:
feature-branch: A---B---C

Efter amend:
feature-branch: A---B---C' (C' inkluderar nya ändringar)
```

**OBS:** C och C' har olika commit-ID!

## Hantera flera review-rundor

### Best practice: Iterativa commits

**Under review-process:**
```bash
# Första reviewen
git commit -m "Address review: Add null checks"
git push

# Andra reviewen
git commit -m "Address review: Improve error messages"
git push

# Tredje reviewen
git commit -m "Address review: Add javadoc"
git push
```

**Innan merge:**
```bash
# Squasha alla review-fixes
git rebase -i HEAD~6
# Squasha alla "Address review" commits in i huvudcommiten
```

Detta ger:
- **Under review:** Tydlig historik över vad som ändrats
- **Efter merge:** Ren, linjär historik i main

## Force push - när och hur

### Varför behövs force push?

Force push behövs när du ändrar commits som redan finns på remote:
- Efter `--amend`
- Efter `rebase`
- Efter `squash`

### ALLTID använd --force-with-lease

```bash
# RÄTT sätt
git push --force-with-lease origin feature/user-login

# FEL sätt (farligt!)
git push -f origin feature/user-login
```

**Varför `--force-with-lease`?**
- Skyddar mot att skriva över andras commits
- Kollar att remote är som du tror
- Säkrare än `-f`

**I IntelliJ:**
IntelliJ använder automatiskt `--force-with-lease` när du bockar i "Force Push"

### När ska du INTE force pusha?

❌ På main eller develop
❌ På delade feature-branches (utan att kommunicera)
❌ På branches som andra bygger sitt arbete på
❌ Om du är osäker på vad som händer

## Kommunikation med reviewers

### Svara på kommentarer

**På GitHub:**
```
Reviewer: "Can you add error handling here?"
Du: "Done in commit abc123"
```

**På GitLab:**
```
Reviewer: "This variable name is confusing"
Du: "Renamed to `userAuthToken` in latest commit"
```

### Markera som löst

Många plattformar låter dig markera review-kommentarer som "Resolved" när du fixat dem.

### Be om re-review

Efter att ha pushat ändringar:
```
"@reviewer I've addressed all your comments. Ready for another look!"
```

## Vanliga fallgropar

### ❌ Force push utan --force-with-lease
```bash
git push -f  # Kan skriva över andras arbete!
```
**Lösning:** Använd alltid `--force-with-lease`

### ❌ Glömmer pusha efter ändringar
```bash
git commit -m "Fix bug"
# Glömmer git push
# Reviewern ser inte ändringarna!
```
**Lösning:** Kolla alltid PR:en efter push för att verifiera

### ❌ Squashar för tidigt
```bash
git rebase -i HEAD~3  # Under pågående review
# Reviewern kan inte se vad som ändrats mellan reviews
```
**Lösning:** Squasha först när PR:en är godkänd och redo för merge

### ❌ Ändrar fel commit vid rebase
Interactive rebase kan vara förvirrande. Läs noga!

**Lösning:** Använd IntelliJ:s UI, eller gör backup först:
```bash
git branch backup-branch  # Skapa backup innan rebase
```

## Verifiering - hur vet du att det gick bra?

```bash
# Kolla att ändringar finns lokalt
git log --oneline -3
git show HEAD

# Kolla att du pushat
git status
# Ska visa: "Your branch is up to date with 'origin/feature-branch'"

# Kolla PR på GitHub/GitLab
# - Senaste commit syns
# - CI/CD har körts
# - Reviewers är notifierade
```

**I IntelliJ:**
- Kolla Git-loggen
- Status längst ner: "↑0 ↓0" = synkad med remote
- Öppna PR:en i webbläsaren

## Best practices för PR-ändringar

✅ **Små, iterativa ändringar** - Lägg till commits under review
✅ **Tydliga commit-meddelanden** - "Address review: fix null check"
✅ **Squasha före merge** - Ren historik i main
✅ **Svara på kommentarer** - Kommunicera vad du fixat
✅ **Testa efter ändringar** - Kör alltid tester före push
✅ **Force push med försiktighet** - Använd `--force-with-lease`
✅ **Be om re-review** - Säg till när du är klar

## Squash strategies - olika team-preferenser

### Strategi 1: Squash vid merge (GitHub/GitLab)
```bash
# Arbeta med många små commits under review
# Plattformen squashar automatiskt vid merge
```
**Fördel:** Enklast, ingen manuell squash
**Nackdel:** Mindre kontroll över slutligt meddelande

### Strategi 2: Squash manuellt före merge
```bash
# Squasha själv innan reviewer godkänner
git rebase -i HEAD~5
```
**Fördel:** Full kontroll, renare PR
**Nackdel:** Mer arbete, kräver force push

### Strategi 3: Aldrig squasha
```bash
# Behåll alla commits som de är
```
**Fördel:** Fullständig historik
**Nackdel:** Rörig git-historik i main

**Rekommendation:** Fråga ditt team vilken strategi ni använder!

## Scenario-exempel: Komplett review-cykel

```bash
# 1. Skapa PR
git push origin feature/user-login

# 2. Första reviewen - små ändringar
# Feedback: "Add null check"
# ... gör ändringar ...
git add .
git commit -m "Address review: add null check"
git push origin feature/user-login

# 3. Andra reviewen - fler ändringar
# Feedback: "Add tests"
# ... gör ändringar ...
git add .
git commit -m "Address review: add tests for edge cases"
git push origin feature/user-login

# 4. Godkänd! Squasha före merge
git rebase -i HEAD~3
# Squasha de två "Address review" commits in i huvudcommiten
git push --force-with-lease origin feature/user-login

# 5. Merga PR
# Klicka "Merge" på GitHub/GitLab
```

## Ångra om något går fel

### Ångra force push
```bash
# Hitta tidigare tillstånd
git reflog
# Leta efter före force push

# Återställ
git reset --hard HEAD@{n}

# Force push tillbaka (om det var fel)
git push --force-with-lease origin feature/user-login
```

### Ångra squash
```bash
# Om du inte pushat än
git reset --hard ORIG_HEAD

# Om du redan pushat - använd backup
git reset --hard backup-branch
git push --force-with-lease origin feature/user-login
```

**I IntelliJ:**
Använd reflog via Git-loggen eller reset till tidigare commit

## Rekommendation

**Under review:**
- Använd separata commits för varje ändring
- Tydliga meddelanden som refererar till feedback
- Pusha efter varje större ändring

**Före merge:**
- Squasha review-commits om teamet föredrar det
- Skriv ett bra slutligt commit-meddelande
- Verifiera att alla tester går igenom

**Kommunikation:**
- Svara på alla review-kommentarer
- Markera lösta issues
- Be om re-review när klar

Med dessa verktyg och workflows kan du effektivt hantera PR-feedback och hålla en ren git-historik!