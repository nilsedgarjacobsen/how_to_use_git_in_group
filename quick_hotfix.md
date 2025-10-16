# Scenario 8: Vi behöver snabbt fixa en kritisk bugg i produktion (hotfix)

## Situationen
En kritisk bugg har upptäckts i produktion:
- Applikationen kraschar för användare
- Betalningar fungerar inte
- Säkerhetshål som måste täppas till omedelbart
- Dataförlust pågår

**Du måste fixa buggen NU**, men pågående utvecklingsarbete får INTE inkluderas i fixen.

## Varför är hotfixes speciella?

**Normal feature:**
```
develop → feature-branch → PR → review → merge → deploy
Tar dagar/veckor
```

**Hotfix:**
```
main/production → hotfix-branch → fix → deploy
Tar minuter/timmar
```

**Utmaningar:**
- Måste baseras på production-koden (inte develop)
- Kan inte vänta på pågående features
- Måste mergas tillbaka till både main OCH develop
- Ingen tid för omfattande review
- Måste vara minimalistisk (bara bugfixen)

## Grundläggande hotfix-workflow

### Steg 1: Pausa ditt pågående arbete

```bash
# Spara ditt pågående arbete
git stash push -m "WIP: Feature work before hotfix"

# Eller committa det
git add .
git commit -m "WIP: Feature work (before hotfix)"
```

**I IntelliJ:**
`Git → Uncommit...` eller stash via commit-dialogen

### Steg 2: Skapa hotfix-branch från production

```bash
# Byt till main (eller master/production)
git checkout main

# Hämta absolut senaste från production
git pull origin main

# Skapa hotfix-branch
git checkout -b hotfix/fix-payment-crash

# Alternativt med namnkonvention som inkluderar datum/ticket
git checkout -b hotfix/2024-10-16-payment-crash
git checkout -b hotfix/JIRA-1234-null-pointer
```

**I IntelliJ:**
1. Byt till `main` (branch-menyn längst ner till höger)
2. Pull (`Ctrl+T`)
3. `New Branch...` → namnge `hotfix/fix-payment-crash`

### Steg 3: Gör den minimala fixen

```bash
# Redigera BARA de filer som behövs
# Ingen refaktorering, inga "nice to have"
# Bara vad som krävs för att stoppa buggen

# Exempel: Lägg till null-check
if (user != null && user.getPaymentMethod() != null) {
    processPayment(user);
}
```

**Principer för hotfix-kod:**
- ✅ Minsta möjliga ändring
- ✅ Defensiv kod (extra null-checks)
- ✅ Tydlig och enkel att granska
- ❌ Ingen omfattande omstrukturering
- ❌ Inga nya features
- ❌ Ingen "tech debt cleanup"

### Steg 4: Testa grundligt (snabbt)

```bash
# Kör relevanta tester
./gradlew test --tests PaymentServiceTest
npm test -- payment.test.js

# Manuell verifiering om möjligt
# Starta appen lokalt och reproducera buggen
```

**I IntelliJ:**
Högerklicka på test-klassen → `Run 'PaymentServiceTest'`

### Steg 5: Committa och pusha

```bash
git add .
git commit -m "Fix: Prevent null pointer in payment processing

- Add null checks for user and payment method
- Fixes crash reported in production"

# Pusha hotfix-branchen
git push origin hotfix/fix-payment-crash
```

**I IntelliJ:**
`Git → Commit` → Skriv tydligt meddelande → `Commit and Push`

### Steg 6: Skapa och merga PR (snabbt)

**På GitHub/GitLab:**
1. Skapa PR från `hotfix/fix-payment-crash` → `main`
2. Märk som kritisk/hotfix
3. Be om snabb review (om policy kräver det)
4. Merga så fort godkänd

**Alternativt (i nödsituation):**
```bash
# Direkt merge utan PR (bara i absolut nödsituation!)
git checkout main
git merge --no-ff hotfix/fix-payment-crash
git push origin main
```

### Steg 7: Deploy till production

```bash
# Beroende på CI/CD-setup
# Ofta triggas automatiskt vid push till main
# Eller manuell trigger av deployment pipeline
```

### Steg 8: Merga tillbaka till develop

**VIKTIGT:** Hotfixen måste också in i develop, annars försvinner den vid nästa release!

```bash
# Byt till develop
git checkout develop
git pull origin develop

# Merga hotfixen
git merge --no-ff hotfix/fix-payment-crash

# Lös eventuella konflikter (se scenario 2)

# Pusha
git push origin develop
```

**I IntelliJ:**
1. Byt till `develop`
2. Pull
3. `Git → Merge...` → välj `hotfix/fix-payment-crash`
4. Lös konflikter om de uppstår
5. Push

### Steg 9: Återgå till ditt pågående arbete

```bash
# Byt tillbaka till din feature-branch
git checkout feature/user-profile

# Återställ ditt stashade arbete
git stash pop

# Eller om du committade WIP
git reset HEAD~1  # Om du vill uncommita

# Uppdatera med hotfixen (valfritt men rekommenderat)
git merge develop
```

### Steg 10: Städa upp

```bash
# Ta bort hotfix-branchen lokalt
git branch -d hotfix/fix-payment-crash

# Ta bort från remote
git push origin --delete hotfix/fix-payment-crash
```

**I IntelliJ:**
Högerklicka på branchen i Git-loggen → `Delete Branch`

## Visualisering av hotfix-flödet

### Komplett workflow-diagram

```
FÖRE HOTFIX:
main:    A---B---C (production)
          \
develop:   D---E---F (pågående utveckling)
            \
feature:     G---H (ditt arbete)

HOTFIX SKAPAS:
main:    A---B---C
          \       \
develop:   D---E---F
            \
feature:     G---H (stashat)
              \
hotfix:        X (ny branch från C)

EFTER FIX OCH MERGE:
main:    A---B---C---X' (hotfix mergad, deployed)
          \       \   \
develop:   D---E---F---X'' (hotfix mergad här också)
            \
feature:     G---H (fortsätter som vanligt)
```

### Påverkan på historik

**I main:**
```
C (före) → X (hotfix) → merge commit → deployed
```

**I develop:**
```
F (senaste develop) → X (hotfix mergad från main)
```

## Git Flow hotfix-konvention

Om ditt team använder Git Flow:

```bash
# Använd git-flow verktyg (om installerat)
git flow hotfix start fix-payment-crash

# Gör fixen
# ...

# Avsluta (mergar automatiskt till både main och develop)
git flow hotfix finish fix-payment-crash
```

**Vad git-flow gör automatiskt:**
1. Skapar branch från main
2. Mergar till main
3. Taggar releasen
4. Mergar till develop
5. Tar bort hotfix-branchen

## Scenario: Konflikter vid merge till develop

### Situationen
```bash
git checkout develop
git merge hotfix/fix-payment-crash
# CONFLICT (content): Merge conflict in PaymentService.java
```

**Varför händer detta?**
Develop har ändrat samma kod som hotfixen.

### Lösning

```bash
# 1. Öppna filen och se konflikten
# 2. Lös konflikten (se scenario 2)
# Oftast vill du ha BÅDA ändringarna:
# - Hotfixen (bugfix)
# - Develops nya features

# 3. Markera som löst
git add PaymentService.java

# 4. Slutför merge
git commit

# 5. Pusha
git push origin develop
```

**Best practice vid hotfix-konflikt:**
- Bevara hotfixen (bugfixen måste finnas kvar!)
- Integrera develops ändringar runt bugfixen
- Testa noga efter merge
- Fråga teamet om osäker

## Olika hotfix-strategier beroende på branching-modell

### Strategi 1: Git Flow (main + develop)
```
main → hotfix → merge till main OCH develop
```

### Strategi 2: GitHub Flow (endast main)
```
main → hotfix → merge till main → develop synkar från main
```

### Strategi 3: Trunk-based (main = production)
```
main → hotfix → merge direkt till main
```

### Strategi 4: Release branches
```
release/v1.2 → hotfix → merge till release OCH main OCH develop
```

**Rekommendation:** Följ den branching-strategi ditt team redan använder!

## Kommunikation under hotfix

### Meddela teamet

**Innan:**
```
@team HOTFIX: Kritisk bugg i produktion (payment crash)
Jag fixar nu, tar ca 30 min
Håll på med era branches, förklaring kommer
```

**Under:**
```
@team Hotfix är pushad till main
Kommer snart merga till develop också
```

**Efter:**
```
@team Hotfix deployed och mergad till develop
Ni kan pulla develop för att få fixen
Om ni får konflikter, ping mig!
```

### Dokumentera i JIRA/issue tracker

- Skapa/uppdatera issue med "HOTFIX" label
- Länka till commit/PR
- Dokumentera vad som fixades
- Dokumentera root cause om känd

## Vanliga fallgropar

### ❌ Skapar hotfix från develop

```bash
git checkout develop  # FEL!
git checkout -b hotfix/fix-bug
```

**Problem:** Inkluderar opågående, otestad utveckling i hotfixen

**Lösning:** Skapa ALLTID från main/production

### ❌ Inkluderar extra "förbättringar"

```java
// HOTFIX - FEL!
public void processPayment(User user) {
    if (user != null) {  // Bugfix
        // Men samtidigt refaktorerar du...
        validateUser(user);
        logPaymentAttempt(user);
        sendNotification(user);
    }
}
```

**Lösning:** BARA bugfixen, inget annat

### ❌ Glömmer merga till develop

```bash
git checkout main
git merge hotfix/fix-bug
git push
# Glömmer develop!
# → Buggen återkommer vid nästa release
```

**Lösning:** Alltid merga till både main OCH develop

### ❌ Force push till main

```bash
git push --force origin main  # ALDRIG!
```

**Problem:** Kan förstöra production-historik

**Lösning:** Använd aldrig force push på main

## Verifiering - hur vet du att det gick bra?

### Före deployment
```bash
# Hotfixen finns i main
git log origin/main --oneline -3
# Ska visa din hotfix-commit

# Tester passerar
./gradlew test
npm test

# Build lyckas
./gradlew build
```

### Efter deployment
```bash
# Verify i production
# - Logga in på prod environment
# - Reproducera buggen (ska vara fixad)
# - Kolla logs för errors

# Hotfixen finns i develop
git log origin/develop --oneline -5
# Ska visa din hotfix-commit
```

### Checklist
- ✅ Buggen är fixad i production
- ✅ Hotfixen är mergad till main
- ✅ Hotfixen är mergad till develop
- ✅ Hotfix-branchen är deletad
- ✅ Teamet är meddelat
- ✅ Issue tracker är uppdaterad
- ✅ Tester passerar i både main och develop

## Scenario-exempel: Komplett hotfix från start till slut

```bash
# 1. ALARM! Bugg i produktion
# Meddelande från ops: "Payment system down!"

# 2. Stasha pågående arbete
git stash push -m "WIP before hotfix"

# 3. Skapa hotfix från main
git checkout main
git pull origin main
git checkout -b hotfix/payment-null-check

# 4. Identifiera och fixa buggen (10 min)
# ... lägg till null-check i PaymentService.java ...

# 5. Testa
./gradlew test --tests PaymentServiceTest
# ✅ Alla tester passerar

# 6. Committa och pusha
git add src/PaymentService.java
git commit -m "Fix: Add null check in payment processing"
git push origin hotfix/payment-null-check

# 7. Skapa och merga PR till main (5 min)
# Via GitHub UI, quick review från lead

# 8. Auto-deploy triggas → Production fixad! (5 min)

# 9. Merga till develop
git checkout develop
git pull origin develop
git merge --no-ff hotfix/payment-null-check
git push origin develop

# 10. Återgå till arbete
git checkout feature/user-profile
git stash pop

# 11. Uppdatera med hotfixen
git merge develop

# 12. Städa
git branch -d hotfix/payment-null-check
git push origin --delete hotfix/payment-null-check

# Total tid: ~30 minuter
```

## När ska du INTE göra en hotfix?

❌ Buggen är irriterande men inte kritisk
❌ Det finns en workaround
❌ Buggen bara påverkar development environment
❌ Fixen kräver stora ändringar

**Istället:**
- Skapa en prioriterad feature-branch
- Fixa via normal PR-process med proper review
- Testa mer omfattande

## Preventiva åtgärder

✅ **Bra tester** - Fångar buggar före production
✅ **Staging environment** - Testar före production-deploy
✅ **Feature flags** - Kan stänga av features utan deploy
✅ **Monitoring** - Upptäcker problem tidigt
✅ **Rollback-plan** - Kan snabbt återgå till tidigare version

## Generella tips

✅ **Håll dig lugn** - Panik skapar fler buggar
✅ **Minimalistisk fix** - Bara vad som behövs
✅ **Testa grundligt** - Men snabbt
✅ **Kommunicera** - Håll teamet uppdaterad
✅ **Dokumentera** - Skriv ner vad som hände
✅ **Post-mortem** - Diskutera hur buggen uppstod (senare)
✅ **Merga till develop** - Glöm inte detta steg!

## Ångra om något går fel

### Om hotfixen orsakar nya problem

```bash
# Revert hotfixen i main (om möjligt)
git checkout main
git revert <hotfix-commit-hash>
git push origin main

# Revert i develop
git checkout develop
git revert <hotfix-commit-hash>
git push origin develop

# Rollback deployment (om ert system stödjer det)
```

### Om du mergat fel

```bash
# Hitta merge-commiten
git log --oneline --graph

# Revert merge
git revert -m 1 <merge-commit-hash>
git push origin develop
```

## Rekommendation

**Före hotfix:**
- Verifiera att det VERKLIGEN är kritiskt
- Reproducera buggen lokalt
- Planera fixen (5 min brainstorming)

**Under hotfix:**
- Minimal ändring
- Testa grundligt
- Kommunicera med teamet

**Efter hotfix:**
- Merga till alla relevanta branches
- Verifiera i production
- Post-mortem (varför uppstod buggen?)

Med denna process kan du hantera produktionskritiska buggar snabbt och säkert!