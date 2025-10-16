# Scenario 9: Main är bruten efter en merge och blockerar teamet

## Situationen
Efter att en PR mergades till main fungerar ingenting:
- Build:en failar
- Tester går inte igenom
- Applikationen startar inte
- CI/CD-pipelinen är röd
- Ingen i teamet kan merga sina PR:s

**Akut situation:** Hela teamet är blockerade!

## Varför händer detta?

**Vanliga orsaker:**
- Tester passerade lokalt men inte på CI
- Merge-konflikt löstes felaktigt
- Feature fungerar isolerat men inte med senaste main
- Miljöskillnader (lokal vs CI environment)
- Beroende-konflikt
- Breaking change i en dependency

**Exempel:**
```
PR #123 mergad → main → CI körs → ❌ FAILED
  ↓
Alla andra PR:s kan inte mergas
  ↓
Teamet blockerat
```

## Prioritering: Åtgärda OMEDELBART

### Steg 0: Identifiera problemet (snabbt!)

```bash
# Kolla senaste commits i main
git log origin/main --oneline -5

# Kolla CI/CD logs
# (via GitHub Actions, GitLab CI, Jenkins, etc.)

# Försök bygga lokalt från main
git checkout main
git pull origin main
./gradlew build  # eller npm run build
```

**I IntelliJ:**
- `Git → Pull`
- `Build → Build Project` (eller `Ctrl+F9`)
- Kolla Build-fönstret för felmeddelanden

### Identifiera syndaren

```bash
# Vilket commit bröt main?
git log --oneline -10
# Leta efter senaste merge-commit

# Kolla CI-historiken
# Se vilket commit som var sista som passerade
```

## Lösning 1: Revert (snabbast och säkrast)

### När använda revert?
- ✅ När du hittat vilket commit som bröt main
- ✅ När fixen tar lång tid att hitta
- ✅ När teamet är blockerat
- ✅ När det är oklart vad som är fel

### Steg-för-steg

```bash
# 1. Checkout main
git checkout main
git pull origin main

# 2. Hitta commit-hashen som bröt
git log --oneline -5
# abc123 Merge pull request #456 from feature/broken-feature
# def456 Previous working commit
# ...

# 3. Revert merge-commiten
git revert -m 1 abc123

# -m 1 betyder "behåll main's version som förälder"
# Detta skapar en ny commit som ångrar ändringarna
```

**Editor öppnas för commit-meddelande:**
```
Revert "Merge pull request #456 from feature/broken-feature"

This reverts commit abc123.

Reason: Breaking build - CI tests failing
Will be re-applied after fix
```

```bash
# 4. Pusha omedelbart
git push origin main

# 5. Verifiera att main fungerar igen
# Vänta på CI/CD att köra
# ✅ Main är grön igen!
```

**I IntelliJ:**
1. Öppna Git-loggen (`Alt+9`)
2. Högerklicka på den brutna merge-commiten
3. Välj `Revert Commit`
4. IntelliJ skapar automatiskt revert-commit
5. Push (`Ctrl+Shift+K`)

### Påverkan på historik

```
Före revert:
main: A---B---C---M (M = bruten merge)

Efter revert:
main: A---B---C---M---R (R = revert av M, main fungerar igen!)
```

**Viktigt:**
- M finns kvar i historiken
- R ångrar ändringarna från M
- Main är åter i fungerande skick

### Efter revert: Fixa och återapplicera

```bash
# Personen som bröt main måste nu:

# 1. Återgå till sin feature-branch
git checkout feature/broken-feature

# 2. Fixa problemet
# ... redigera kod, fixa tester ...

# 3. Skapa ny PR
# Med titel: "Re-apply: Feature X (now fixed)"

# 4. Merge när CI är grön
```

## Lösning 2: Fast-forward fix (snabb patch)

### När använda fast-forward fix?
- ✅ När du omedelbart VET vad som är fel
- ✅ När fixen är trivial (en rad kod)
- ✅ När du kan fixa på <5 minuter

### Exempel: Glömd import

**Fel:**
```java
// PaymentService.java - saknar import
public class PaymentService {
    Logger logger = LoggerFactory.getLogger(PaymentService.class);
    // ❌ LoggerFactory not imported!
}
```

**Fix:**
```bash
# 1. Skapa snabb fix-branch
git checkout main
git pull origin main
git checkout -b fix/add-missing-import

# 2. Lägg till den glömda importen
# ... redigera filen ...

# 3. Committa och pusha
git add src/PaymentService.java
git commit -m "Fix: Add missing LoggerFactory import"
git push origin fix/add-missing-import

# 4. Skapa PR och merga OMEDELBART
# (eller direkt merge om tillåtet)
git checkout main
git merge fix/add-missing-import
git push origin main
```

**Tidslinje:**
```
Problem upptäckt: 10:00
Fix implementerad: 10:03
Mergad och CI grön: 10:05
```

## Lösning 3: Force push (SISTA UTVÄGN - farligt!)

### ⚠️ VARNING: Använd ENDAST i extrema fall

**När:**
- ❌ Praktiskt taget ALDRIG på main
- ✅ Endast om INGEN har pullt den brutna main än
- ✅ Endast efter samråd med hela teamet

```bash
# Steg 1: Identifiera senaste fungerande commit
git log --oneline -10
# def456 Last working commit
# abc123 Broken merge (detta vill vi ta bort)

# Steg 2: Reset lokalt
git reset --hard def456

# Steg 3: KOMMUNICERA MED TEAMET
# "⚠️ Force pushing main i 2 min, stasha/committa ert arbete!"

# Steg 4: Force push (FARLIGT!)
git push --force-with-lease origin main

# Steg 5: Meddela teamet
# "Force push done, ni behöver köra: git fetch && git reset --hard origin/main"
```

**Konsekvenser:**
- Alla som pullt den brutna main måste reset:a
- Commit-historiken är omskriven
- Risk för förlorat arbete

**Rekommendation:** Använd revert istället!

## Kommunikation under krisen

### Omedelbart meddelande till teamet

```
🚨 BLOCKER: Main is broken

Commit: abc123 "Merge PR #456"
Issue: Build failing, tests not passing
Action: I'm reverting now
ETA: 5 minutes

DO NOT merge anything until I give all-clear!
```

### Under åtgärd

```
Working on it...
Revert in progress
CI running...
```

### Efter lösning

```
✅ Main is fixed!

Reverted commit: abc123
Main is green again: [CI link]
You can continue merging PRs

@author-of-broken-commit Please fix and re-apply
```

## Scenario: Komplext problem - felsökning

### När det inte är uppenbart vad som gått fel

**Steg 1: Återskapa lokalt**
```bash
git checkout main
git pull origin main

# Försök bygga
./gradlew clean build

# Kör tester
./gradlew test --tests com.example.*

# Läs felmeddelanden noga
```

**Steg 2: Jämför med fungerande version**
```bash
# Hitta senaste fungerande commit
git log --oneline -10

# Checkout den versionen
git checkout def456

# Bygger den?
./gradlew build
# ✅ Ja, den fungerar

# Checkout bruten version igen
git checkout main

# Vad är skillnaden?
git diff def456..main
```

**Steg 3: Binary search med git bisect**
```bash
# Starta bisect
git bisect start

# Markera nuvarande (bruten) version
git bisect bad

# Markera fungerande version
git bisect good def456

# Git checkar ut mitten-commit
# Testa: ./gradlew build

# Om det fungerar:
git bisect good
# Om det inte fungerar:
git bisect bad

# Upprepa tills Git hittar första brutna commit
# Git visar: "abc123 is the first bad commit"

# Avsluta
git bisect reset
```

**I IntelliJ:**
Använd kommandoraden för bisect - inget bra UI-stöd

## Preventiva åtgärder

### 1. Skydda main-branchen

**På GitHub:**
- Settings → Branches → Branch protection rules
- ✅ Require status checks to pass before merging
- ✅ Require branches to be up to date
- ✅ Require review from code owners

**På GitLab:**
- Settings → Repository → Protected branches
- ✅ Allowed to merge: Maintainers only
- ✅ Require merge request approval

### 2. CI/CD som gatekeeper

```yaml
# GitHub Actions exempel
on:
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run tests
        run: ./gradlew test
      - name: Build
        run: ./gradlew build
```

**Blockera merge om CI failar!**

### 3. Pre-merge checklist

- ✅ Alla tester passerar lokalt
- ✅ Build fungerar lokalt
- ✅ Branch är uppdaterad med main
- ✅ Kod-granskning genomförd
- ✅ CI är grön

### 4. Kör tester mot senaste main

```bash
# Innan du mergar din PR
git checkout feature/my-feature
git fetch origin
git merge origin/main
./gradlew test
# Om detta passerar, mergar du
```

## Vanliga fallgropar

### ❌ Väntar för länge med revert

```
10:00 - Main bruten
10:30 - Fortfarande felsöker
11:00 - Teamet frustrerat
```

**Lösning:** Revert omedelbart, felsök sedan

### ❌ Försöker fixa med fler commits

```
Broken commit → Quick fix → Another fix → Still broken...
```

**Lösning:** Revert först, sedan en proper fix i ny PR

### ❌ Glömmer kommunicera

Teamet fortsätter merga och skapar fler problem.

**Lösning:** Meddela OMEDELBART i teamchatt

### ❌ Force push utan samordning

Skapar kaos för alla andra.

**Lösning:** Använd revert istället

## Verifiering - hur vet du att main är fixad?

```bash
# Kolla CI/CD status
# Ska vara grön ✅

# Bygga lokalt från main
git checkout main
git pull origin main
./gradlew clean build
# ✅ Ska lyckas

# Kör alla tester
./gradlew test
# ✅ Alla ska passa

# Kolla att appen startar
./gradlew bootRun  # eller npm start
# ✅ Ska starta utan errors
```

**I IntelliJ:**
- Build Project → inga errors
- Run All Tests → alla gröna
- Main-branchen i Git-loggen visar grön CI-status

## Vem ansvarar för att fixa?

### Scenario 1: Tydligt vem som bröt
```
PR #456 av Alice → bröt main
→ Alice ansvarar för att fixa
```

### Scenario 2: Otydligt, merge-konflikt
```
PR #456 (Alice) + PR #457 (Bob) → konflikt
→ Senaste personen att merga tar ansvar
→ Eller: båda samarbetar om att fixa
```

### Scenario 3: Akut och ingen tillgänglig
```
Alice är offline/semester
→ Teamet reverterar
→ Alice fixar när hon är tillbaka
```

**Kultur över policy:**
- Ingen skuldbeläggning
- Teamet hjälps åt
- Lär av misstag

## Post-mortem: Efter att main är fixad

### Viktiga frågor

1. **Vad gick fel?**
   - Vilken kod/test bröt?
   - Varför upptäcktes det inte före merge?

2. **Varför hände det?**
   - Saknades CI-check?
   - Otillräcklig testning?
   - Miljöskillnader?

3. **Hur förhindrar vi det?**
   - Nya CI-checks?
   - Bättre tester?
   - Strängare review-policy?

### Dokumentera lärdomar

```markdown
## Post-Mortem: Main broken 2024-10-16

**Incident:** Build failing after PR #456 merged

**Root Cause:** Missing integration test for payment flow

**Fix:** Reverted commit abc123, proper fix in PR #458

**Prevention:**
- Add integration test for payment flows
- Require integration tests to pass before merge
- Update PR template to remind about integration tests

**Timeline:**
10:00 - Main broken (detected by CI)
10:05 - Team notified
10:08 - Revert initiated
10:10 - Main restored
10:45 - Proper fix merged
```

## Generella tips

✅ **Revert först, fråga sen** - Unblock teamet omedelbart
✅ **Kommunicera tydligt** - Håll alla informerade
✅ **Lär av misstag** - Post-mortem utan skuldbeläggning
✅ **Förbättra CI/CD** - Fånga problem före merge
✅ **Testautomatisering** - Minska risk för manuella fel
✅ **Branch protection** - Tekniska safeguards
✅ **Snabb responstid** - Ha en "on-call" rotation

## Ångra om revert var fel

### Om du revertade fel commit

```bash
# Revert av revert = återställer originalet
git revert <revert-commit-hash>
git push origin main
```

### Om main fortfarande är bruten efter revert

```bash
# Du revertade fel commit, main fortfarande bruten
# Hitta rätt commit och revert
git log --oneline -10
git revert -m 1 <actual-broken-commit>
git push origin main
```

## Rekommendation

**När main är bruten:**
1. **0-2 min:** Identifiera problem och meddela teamet
2. **2-5 min:** Beslut - Revert eller quick fix?
3. **5-10 min:** Åtgärda (revert är nästan alltid rätt val)
4. **10-15 min:** Verifiera att main är grön
5. **Efter:** Post-mortem och förbättringar

**Mantra:** "Revert first, fix later"

**Prioritetsordning:**
1. Återställ main till fungerande skick (revert)
2. Unblock teamet
3. Fixa problemet ordentligt (ny PR)
4. Lär av misstaget (post-mortem)

Med denna approach minimerar du downtime och håller teamet produktivt!