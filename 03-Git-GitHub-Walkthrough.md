# Git & GitHub - Kompletter Walkthrough

## Inhaltsverzeichnis
1. [Einführung](#einführung)
2. [Installation und Setup](#installation-und-setup)
3. [Git Grundlagen](#git-grundlagen)
4. [Branches und Merging](#branches-und-merging)
5. [Remote Repositories](#remote-repositories)
6. [GitHub Basics](#github-basics)
7. [Collaboration Workflows](#collaboration-workflows)
8. [Pull Requests](#pull-requests)
9. [Konflikte lösen](#konflikte-lösen)
10. [Git Best Practices](#git-best-practices)
11. [Fortgeschrittene Git-Befehle](#fortgeschrittene-git-befehle)
12. [GitHub Features](#github-features)
13. [Praktische Szenarien](#praktische-szenarien)

---

## Einführung

### Was ist Git?

**Git** ist ein verteiltes Versionskontrollsystem, das von Linus Torvalds entwickelt wurde. Es ermöglicht:

- **Versionsverwaltung**: Änderungen nachverfolgen und rückgängig machen
- **Collaboration**: Mehrere Entwickler arbeiten gleichzeitig am selben Projekt
- **Branching**: Parallele Entwicklung verschiedener Features
- **History**: Vollständige Historie aller Änderungen
- **Backup**: Dezentrale Sicherung des Codes

### Was ist GitHub?

**GitHub** ist eine Hosting-Plattform für Git-Repositories mit zusätzlichen Features:

- **Remote Hosting**: Cloud-basierte Repository-Verwaltung
- **Collaboration**: Pull Requests, Issues, Discussions
- **CI/CD**: GitHub Actions für Automatisierung
- **Documentation**: Wiki, README, GitHub Pages
- **Social Coding**: Profile, Follower, Stars

### Git vs GitHub

| Aspekt | Git | GitHub |
|--------|-----|--------|
| Was ist es? | Versionskontrollsystem | Hosting-Plattform |
| Lokal/Remote | Lokal | Remote (Cloud) |
| Kosten | Kostenlos | Kostenlos + Paid Plans |
| Installation | Muss installiert werden | Web-basiert |
| Alternativen | - | GitLab, Bitbucket |

---

## Installation und Setup

### Git installieren

#### Windows
```powershell
# Mit Chocolatey
choco install git

# Oder: Herunterladen von https://git-scm.com/download/win
```

#### macOS
```bash
# Mit Homebrew
brew install git

# Oder: Xcode Command Line Tools
xcode-select --install
```

#### Linux (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install git
```

### Installation überprüfen

```bash
git --version
# Output: git version 2.42.0
```

### Git konfigurieren

#### Globale Konfiguration (für alle Projekte)

```bash
# Name setzen
git config --global user.name "Max Mustermann"

# E-Mail setzen
git config --global user.email "max@example.com"

# Standard-Editor (z.B. VS Code)
git config --global core.editor "code --wait"

# Standard-Branch-Name (main statt master)
git config --global init.defaultBranch main

# Farbige Ausgabe aktivieren
git config --global color.ui auto
```

#### Konfiguration anzeigen

```bash
# Alle Einstellungen anzeigen
git config --list

# Spezifische Einstellung anzeigen
git config user.name
git config user.email
```

#### Lokale Konfiguration (nur für aktuelles Projekt)

```bash
# Im Projektverzeichnis
git config user.name "Anderer Name"
git config user.email "andere@example.com"
```

### SSH-Key für GitHub einrichten

#### 1. SSH-Key generieren

```bash
# Ed25519-Key (empfohlen)
ssh-keygen -t ed25519 -C "deine@email.com"

# Oder RSA (falls Ed25519 nicht unterstützt wird)
ssh-keygen -t rsa -b 4096 -C "deine@email.com"

# Speicherort: ~/.ssh/id_ed25519 (Enter für Standard)
# Passphrase: Optional, aber empfohlen
```

#### 2. SSH-Agent starten

```bash
# SSH-Agent starten
eval "$(ssh-agent -s)"

# Key zum Agent hinzufügen
ssh-add ~/.ssh/id_ed25519
```

#### 3. Public Key kopieren

```bash
# Windows (PowerShell)
Get-Content ~/.ssh/id_ed25519.pub | Set-Clipboard

# macOS
pbcopy < ~/.ssh/id_ed25519.pub

# Linux
cat ~/.ssh/id_ed25519.pub | xclip -selection clipboard
```

#### 4. Key zu GitHub hinzufügen

1. Gehe zu GitHub → Settings → SSH and GPG keys
2. Klicke auf "New SSH key"
3. Gib einen Titel ein (z.B. "Mein Laptop")
4. Füge den Public Key ein
5. Klicke auf "Add SSH key"

#### 5. Verbindung testen

```bash
ssh -T git@github.com

# Erfolgreiche Ausgabe:
# Hi username! You've successfully authenticated...
```

---

## Git Grundlagen

### Repository initialisieren

```bash
# Neues Repository erstellen
mkdir mein-projekt
cd mein-projekt
git init

# Ergebnis: Erstellt .git/-Verzeichnis
```

### Status überprüfen

```bash
# Aktuellen Status anzeigen
git status

# Kurzform
git status -s
```

### Dateien zum Staging hinzufügen

```bash
# Einzelne Datei hinzufügen
git add datei.txt

# Mehrere Dateien
git add datei1.txt datei2.txt

# Alle Dateien im aktuellen Verzeichnis
git add .

# Alle Dateien im gesamten Repository
git add -A

# Interaktives Hinzufügen
git add -p
```

### Commit erstellen

```bash
# Commit mit Message
git commit -m "Initiales Commit"

# Commit mit mehrzeiliger Message
git commit -m "Kurzbeschreibung" -m "Ausführliche Beschreibung der Änderungen"

# Alle geänderten Dateien committen (überspringt git add)
git commit -am "Update alle Dateien"

# Editor für ausführliche Message öffnen
git commit
```

### Commit-Message Best Practices

```bash
# Gut ✅
git commit -m "Add user authentication feature"
git commit -m "Fix login button not responding on mobile"
git commit -m "Update README with installation instructions"

# Schlecht ❌
git commit -m "fix"
git commit -m "changes"
git commit -m "asdfasdf"
```

**Conventional Commits Format:**

```bash
# Format: <type>(<scope>): <subject>

git commit -m "feat(auth): add user login functionality"
git commit -m "fix(api): resolve CORS issue on production"
git commit -m "docs(readme): add installation guide"
git commit -m "style(button): improve hover animation"
git commit -m "refactor(utils): simplify date formatting function"
git commit -m "test(auth): add unit tests for login"
git commit -m "chore(deps): update dependencies"
```

### Historie anzeigen

```bash
# Alle Commits anzeigen
git log

# Kompakte Ansicht
git log --oneline

# Mit Graph
git log --graph --oneline --all

# Letzte N Commits
git log -n 5

# Commits eines bestimmten Autors
git log --author="Max"

# Commits mit Änderungen
git log -p

# Commits in bestimmtem Zeitraum
git log --since="2024-01-01" --until="2024-12-31"

# Statistiken
git log --stat

# Schöne Formatierung
git log --pretty=format:"%h - %an, %ar : %s"
```

### Änderungen anzeigen

```bash
# Unterschiede zwischen Working Directory und Staging
git diff

# Unterschiede im Staging Area
git diff --staged

# Unterschiede zwischen Commits
git diff commit1 commit2

# Unterschiede für bestimmte Datei
git diff datei.txt

# Statistik der Änderungen
git diff --stat
```

### Dateien ignorieren (.gitignore)

```bash
# .gitignore erstellen
touch .gitignore
```

**Beispiel .gitignore:**

```gitignore
# Dependencies
node_modules/
bower_components/

# Build-Outputs
dist/
build/
*.min.js
*.min.css

# Logs
logs/
*.log
npm-debug.log*

# Environment-Dateien
.env
.env.local
.env.production

# IDE-Dateien
.vscode/
.idea/
*.swp
*.swo
*~

# OS-Dateien
.DS_Store
Thumbs.db

# Test-Coverage
coverage/
.nyc_output/

# Temporäre Dateien
tmp/
temp/
*.tmp

# Spezifische Dateien
config/secrets.json
```

```bash
# .gitignore zum Repository hinzufügen
git add .gitignore
git commit -m "Add .gitignore"
```

### Dateien aus dem Repository entfernen

```bash
# Datei löschen und aus Git entfernen
git rm datei.txt

# Datei nur aus Git entfernen (behalten im Dateisystem)
git rm --cached datei.txt

# Ordner rekursiv entfernen
git rm -r ordner/

# Commit erforderlich
git commit -m "Remove unnecessary files"
```

### Dateien umbenennen

```bash
# Datei umbenennen
git mv alte-datei.txt neue-datei.txt

# Entspricht:
# mv alte-datei.txt neue-datei.txt
# git rm alte-datei.txt
# git add neue-datei.txt

git commit -m "Rename file"
```

### Änderungen rückgängig machen

```bash
# Änderungen im Working Directory verwerfen
git checkout -- datei.txt

# Moderne Alternative (Git 2.23+)
git restore datei.txt

# Alle Änderungen verwerfen
git restore .

# Datei aus Staging entfernen (aber Änderungen behalten)
git reset HEAD datei.txt

# Moderne Alternative
git restore --staged datei.txt

# Letzten Commit rückgängig machen (Änderungen behalten)
git reset --soft HEAD~1

# Letzten Commit rückgängig machen (Änderungen verwerfen)
git reset --hard HEAD~1

# ACHTUNG: --hard löscht alle Änderungen unwiderruflich!
```

---

## Branches und Merging

### Was sind Branches?

Branches ermöglichen parallele Entwicklung ohne den Hauptcode zu beeinflussen.

```
main:      A---B---C---D
                \
feature:         E---F---G
```

### Branch erstellen und wechseln

```bash
# Alle Branches anzeigen
git branch

# Alle Branches inkl. Remote
git branch -a

# Neuen Branch erstellen
git branch feature-login

# Zu Branch wechseln
git checkout feature-login

# Branch erstellen UND wechseln (in einem Befehl)
git checkout -b feature-login

# Moderne Alternative (Git 2.23+)
git switch feature-login
git switch -c feature-login  # erstellen und wechseln
```

### Branch-Workflow

```bash
# 1. Neuen Feature-Branch erstellen
git checkout -b feature-user-profile

# 2. Änderungen machen
echo "User Profile Code" > profile.js
git add profile.js
git commit -m "Add user profile page"

# 3. Weitere Commits
echo "More changes" >> profile.js
git commit -am "Improve profile layout"

# 4. Zurück zu main wechseln
git checkout main

# 5. Feature-Branch mergen
git merge feature-user-profile

# 6. Branch löschen (optional)
git branch -d feature-user-profile
```

### Branch löschen

```bash
# Branch löschen (nur wenn gemerged)
git branch -d branch-name

# Branch löschen (erzwingen, auch wenn nicht gemerged)
git branch -D branch-name

# Remote Branch löschen
git push origin --delete branch-name
```

### Merging

#### Fast-Forward Merge

Wenn keine neuen Commits auf dem Ziel-Branch sind:

```bash
# main: A---B---C
# feature:        D---E

git checkout main
git merge feature

# Resultat:
# main: A---B---C---D---E
```

```bash
git merge feature
# Fast-forward merge
```

#### 3-Way Merge

Wenn beide Branches neue Commits haben:

```bash
# main:    A---B---C---F
#               \
# feature:       D---E

git checkout main
git merge feature

# Resultat:
# main:    A---B---C---F---G (Merge-Commit)
#               \         /
# feature:       D---E---
```

```bash
git merge feature
# Merge commit erstellt
```

#### Merge mit No-Fast-Forward

Erzwingt einen Merge-Commit, auch bei Fast-Forward-Möglichkeit:

```bash
git merge --no-ff feature-branch

# Immer einen Merge-Commit erstellen (nützlich für Historie)
```

### Rebase (Alternative zu Merge)

Rebase verschiebt Commits auf einen anderen Base-Commit:

```bash
# main:    A---B---C---D
#               \
# feature:       E---F

git checkout feature
git rebase main

# Resultat:
# main:    A---B---C---D
#                       \
# feature:               E'---F' (neue Commits)
```

```bash
# Feature-Branch auf main rebasen
git checkout feature-branch
git rebase main

# Konflikte lösen (falls vorhanden)
# ... Dateien bearbeiten ...
git add .
git rebase --continue

# Rebase abbrechen
git rebase --abort
```

#### Wann Merge vs Rebase?

**Merge verwenden:**
- ✅ Feature-Branches zu main mergen
- ✅ Historie soll sichtbar bleiben
- ✅ Öffentliche/geteilte Branches

**Rebase verwenden:**
- ✅ Lokalen Feature-Branch aufräumen
- ✅ Lineare Historie gewünscht
- ✅ Vor dem Push zum Remote
- ❌ NIEMALS auf öffentlichen Branches!

### Branch-Strategien

#### Git Flow

```
main (production)
  └── develop (development)
      ├── feature/login
      ├── feature/profile
      └── release/v1.0
      └── hotfix/critical-bug
```

#### GitHub Flow (Einfacher)

```
main (production)
  ├── feature/login
  ├── feature/profile
  └── bugfix/navigation
```

#### Trunk-Based Development

```
main (alle arbeiten hier)
  └── kurzlebige Feature-Branches (max 1-2 Tage)
```

---

## Remote Repositories

### Remote Repository hinzufügen

```bash
# Remote hinzufügen
git remote add origin https://github.com/username/repo.git

# Oder mit SSH
git remote add origin git@github.com:username/repo.git

# Remotes anzeigen
git remote -v

# Remote Details
git remote show origin
```

### Push (Hochladen)

```bash
# Zum Remote pushen
git push origin main

# Beim ersten Push: Tracking-Branch setzen
git push -u origin main
# Danach reicht: git push

# Alle Branches pushen
git push --all

# Tags pushen
git push --tags

# Branch zum Remote pushen
git push origin feature-branch

# Force Push (VORSICHT!)
git push --force
# Sicherer: force with lease
git push --force-with-lease
```

### Pull (Herunterladen)

```bash
# Änderungen vom Remote holen und mergen
git pull origin main

# Mit Rebase statt Merge
git pull --rebase origin main

# Wenn Tracking-Branch gesetzt ist
git pull
```

### Fetch vs Pull

```bash
# Fetch: Nur herunterladen, nicht mergen
git fetch origin

# Zeigt was verfügbar ist
git fetch --all

# Dann manuell mergen
git merge origin/main

# Pull = Fetch + Merge
git pull origin main
```

### Clone (Repository kopieren)

```bash
# Repository klonen (HTTPS)
git clone https://github.com/username/repo.git

# Repository klonen (SSH)
git clone git@github.com:username/repo.git

# In spezifischen Ordner klonen
git clone https://github.com/username/repo.git mein-ordner

# Nur bestimmten Branch klonen
git clone -b develop https://github.com/username/repo.git

# Shallow Clone (nur letzte N Commits)
git clone --depth 1 https://github.com/username/repo.git
```

---

## GitHub Basics

### Repository auf GitHub erstellen

#### Via GitHub Website:

1. Gehe zu https://github.com
2. Klicke auf "+" → "New repository"
3. Fülle die Details aus:
   - Repository Name
   - Description (optional)
   - Public/Private
   - Initialize with README (optional)
   - .gitignore (optional)
   - License (optional)
4. Klicke "Create repository"

#### Bestehendes lokales Repo zu GitHub pushen:

```bash
# 1. Auf GitHub ein leeres Repository erstellen (ohne README)

# 2. Remote hinzufügen
git remote add origin git@github.com:username/repo.git

# 3. Branch umbenennen (falls nötig)
git branch -M main

# 4. Pushen
git push -u origin main
```

### README.md erstellen

```markdown
# Projekt-Name

Kurze Beschreibung des Projekts.

## Features

- Feature 1
- Feature 2
- Feature 3

## Installation

\`\`\`bash
npm install
\`\`\`

## Verwendung

\`\`\`bash
npm start
\`\`\`

## Technologien

- Next.js
- TypeScript
- Tailwind CSS

## Lizenz

MIT
```

```bash
# README erstellen und committen
echo "# Mein Projekt" > README.md
git add README.md
git commit -m "Add README"
git push
```

### GitHub Issues

Issues sind für Bug-Reports, Feature-Requests und Diskussionen.

```bash
# Issues können nur via GitHub Web-Interface erstellt werden
```

**Gutes Issue-Template:**

```markdown
## Beschreibung
Klare Beschreibung des Problems oder Features

## Schritte zum Reproduzieren
1. Gehe zu '...'
2. Klicke auf '...'
3. Scrolle nach unten zu '...'
4. Siehe Fehler

## Erwartetes Verhalten
Was sollte passieren?

## Aktuelles Verhalten
Was passiert tatsächlich?

## Screenshots
Falls zutreffend

## Umgebung
- OS: [z.B. Windows 11]
- Browser: [z.B. Chrome 120]
- Version: [z.B. 1.2.3]
```

### GitHub Projects

Kanban-Board für Projektmanagement:

1. Repository → Projects → New project
2. Wähle Template (Kanban, Table, etc.)
3. Füge Issues hinzu
4. Verschiebe zwischen Spalten (Todo, In Progress, Done)

---

## Collaboration Workflows

### Fork-Workflow

Für Open-Source-Projekte:

```bash
# 1. Repository auf GitHub forken (via Web-Interface)

# 2. Deinen Fork klonen
git clone git@github.com:dein-username/original-repo.git
cd original-repo

# 3. Upstream Remote hinzufügen
git remote add upstream git@github.com:original-owner/original-repo.git

# 4. Neuen Branch für deine Änderungen
git checkout -b feature-improvement

# 5. Änderungen machen und committen
git add .
git commit -m "Add improvement"

# 6. Zu deinem Fork pushen
git push origin feature-improvement

# 7. Pull Request auf GitHub erstellen

# 8. Fork aktualisieren mit Upstream
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

### Shared Repository Workflow

Für Team-Projekte mit direktem Zugriff:

```bash
# 1. Repository klonen
git clone git@github.com:team/project.git
cd project

# 2. Neuen Feature-Branch erstellen
git checkout -b feature/user-auth

# 3. Änderungen machen
git add .
git commit -m "feat(auth): implement user authentication"

# 4. Vor dem Push: Main aktualisieren
git checkout main
git pull origin main

# 5. Feature-Branch rebasen
git checkout feature/user-auth
git rebase main

# 6. Pushen
git push origin feature/user-auth

# 7. Pull Request erstellen
```

### Feature-Branch Workflow

```bash
# Täglich:

# 1. Main aktualisieren
git checkout main
git pull origin main

# 2. Feature-Branch aktualisieren
git checkout feature-branch
git rebase main  # oder: git merge main

# 3. Arbeiten
# ... Code ändern ...
git add .
git commit -m "Progress on feature"

# 4. Pushen (überschreibt vorherige Pushes nach Rebase)
git push --force-with-lease origin feature-branch

# 5. Wenn fertig: Pull Request erstellen
```

---

## Pull Requests

### Pull Request erstellen

1. **Pushe deinen Branch:**
   ```bash
   git push origin feature-branch
   ```

2. **Auf GitHub:**
   - Gehe zum Repository
   - Klicke "Compare & pull request" (erscheint automatisch)
   - Oder: Pull requests → New pull request

3. **PR Details ausfüllen:**
   ```markdown
   ## Was macht dieser PR?
   Implementiert User-Authentication mit JWT
   
   ## Änderungen
   - [ ] Login-Seite erstellt
   - [ ] JWT-Token-Handling
   - [ ] Protected Routes
   
   ## Tests
   - [ ] Unit Tests hinzugefügt
   - [ ] Manuelle Tests durchgeführt
   
   ## Screenshots
   [Hier Screenshots einfügen]
   
   ## Related Issues
   Closes #123
   ```

4. **Reviewer zuweisen**

5. **Labels hinzufügen** (feature, bugfix, etc.)

### Pull Request Review

```bash
# PR lokal testen
git fetch origin
git checkout -b review-branch origin/feature-branch

# Testen und ändern
# ... Code testen ...

# Wenn Änderungen nötig
# ... Änderungen machen ...
git add .
git commit -m "Review feedback: improve error handling"
git push origin review-branch
```

**Review-Kommentare auf GitHub:**
- Einzelne Zeilen kommentieren
- Änderungen vorschlagen
- Approve / Request Changes / Comment

### PR-Merge-Strategien

#### 1. Merge Commit (Standard)
```bash
# Erhält alle Commits und erstellt Merge-Commit
# Resultat: Feature-Branch-Historie sichtbar
```

#### 2. Squash and Merge
```bash
# Alle Commits zu einem zusammenfassen
# Resultat: Saubere lineare Historie
```

#### 3. Rebase and Merge
```bash
# Commits ohne Merge-Commit übernehmen
# Resultat: Lineare Historie
```

### Draft Pull Requests

```bash
# Für PRs die noch nicht fertig sind

# 1. Normalen PR erstellen
# 2. Als "Draft" markieren
# 3. Wenn fertig: "Ready for review"
```

### PR-Templates

**.github/pull_request_template.md:**

```markdown
## Beschreibung
<!-- Beschreibe deine Änderungen -->

## Art der Änderung
- [ ] Bug Fix
- [ ] Neues Feature
- [ ] Breaking Change
- [ ] Dokumentation

## Wie wurde das getestet?
<!-- Beschreibe deine Tests -->

## Checklist
- [ ] Code folgt den Projekt-Richtlinien
- [ ] Selbst-Review durchgeführt
- [ ] Kommentare hinzugefügt
- [ ] Dokumentation aktualisiert
- [ ] Keine neuen Warnungen
- [ ] Tests hinzugefügt
- [ ] Alle Tests bestanden
```

---

## Konflikte lösen

### Was sind Merge-Konflikte?

Konflikte entstehen, wenn dieselbe Stelle in einer Datei unterschiedlich geändert wurde.

```bash
# Konflikt-Situation:
# main:    A---B---C (ändert Zeile 5)
#               \
# feature:       D (ändert auch Zeile 5)

git checkout main
git merge feature
# CONFLICT (content): Merge conflict in file.txt
```

### Konflikt-Markierungen

```javascript
// file.js
function hello() {
<<<<<<< HEAD
  console.log("Hello from main");
=======
  console.log("Hello from feature");
>>>>>>> feature
}
```

- `<<<<<<< HEAD`: Version im aktuellen Branch
- `=======`: Trenner
- `>>>>>>> feature`: Version im zu mergenden Branch

### Konflikte manuell lösen

```bash
# 1. Status prüfen
git status
# both modified: file.js

# 2. Datei öffnen und bearbeiten
# Entscheide welche Version du behalten willst

# Option A: Main-Version behalten
function hello() {
  console.log("Hello from main");
}

# Option B: Feature-Version behalten
function hello() {
  console.log("Hello from feature");
}

# Option C: Beide kombinieren
function hello() {
  console.log("Hello from main and feature");
}

# 3. Konflikt-Markierungen entfernen
# 4. Datei speichern

# 5. Als gelöst markieren
git add file.js

# 6. Merge fortsetzen
git commit -m "Merge feature branch, resolve conflicts"
```

### Konflikte mit VS Code lösen

VS Code zeigt Konflikte mit Buttons an:

```
Accept Current Change | Accept Incoming Change | Accept Both Changes | Compare Changes
```

```bash
# 1. Klicke auf gewünschte Option
# 2. Speichere Datei
# 3. git add file.js
# 4. git commit
```

### Merge abbrechen

```bash
# Wenn Konflikte zu komplex sind
git merge --abort

# Zurück zum Zustand vor dem Merge
```

### Konflikte beim Rebase

```bash
git rebase main
# CONFLICT...

# 1. Konflikte lösen
# ... Dateien bearbeiten ...

# 2. Als gelöst markieren
git add .

# 3. Rebase fortsetzen
git rebase --continue

# Oder abbrechen
git rebase --abort
```

### Konflikte beim Pull

```bash
git pull origin main
# CONFLICT...

# Gleicher Prozess wie beim Merge:
# 1. Konflikte lösen
# 2. git add
# 3. git commit
```

### Konflikte vermeiden

```bash
# 1. Häufig vom main pullen
git checkout main
git pull origin main
git checkout feature
git merge main  # oder: git rebase main

# 2. Kleine, fokussierte Commits

# 3. Kommunikation im Team

# 4. Nicht dieselben Dateien gleichzeitig bearbeiten

# 5. Pull Requests schnell reviewen und mergen
```

---

## Git Best Practices

### Commit-Richtlinien

```bash
# ✅ GUTE Commits

# Klein und fokussiert
git commit -m "feat(auth): add login form validation"

# Beschreibend
git commit -m "fix(api): resolve timeout issue in user endpoint"

# Präsens
git commit -m "add password reset functionality"

# ❌ SCHLECHTE Commits

# Zu groß
git commit -m "fix everything"

# Unklar
git commit -m "changes"

# Vergangenheit
git commit -m "added feature"

# Unprofessionell
git commit -m "wtf is this"
```

### Branch-Namenskonventionen

```bash
# Format: <type>/<kurze-beschreibung>

# Features
git checkout -b feature/user-authentication
git checkout -b feature/payment-integration

# Bugfixes
git checkout -b bugfix/login-error
git checkout -b fix/memory-leak

# Hotfixes
git checkout -b hotfix/security-vulnerability

# Experimente
git checkout -b experiment/new-ui

# Releases
git checkout -b release/v1.2.0
```

### .gitignore Best Practices

```gitignore
# Spezifisch vor Allgemein

# Dependencies
node_modules/

# Build
dist/
build/

# Environment
.env
.env.local

# IDEs
.vscode/
.idea/

# OS
.DS_Store
Thumbs.db

# Logs
*.log

# Test Coverage
coverage/
```

### Wann committen?

```bash
# ✅ Gute Zeitpunkte:
# - Feature komplett implementiert
# - Bug gefixt
# - Tests geschrieben
# - Refactoring abgeschlossen
# - Logischer Arbeitsschritt beendet

# ❌ Schlechte Zeitpunkte:
# - Code funktioniert nicht
# - Mitten in der Implementierung
# - Ende des Arbeitstags (nur um zu speichern)
```

### Git-Workflows

#### Feature Development

```bash
# 1. Main aktualisieren
git checkout main
git pull origin main

# 2. Feature-Branch erstellen
git checkout -b feature/new-dashboard

# 3. Arbeiten und committen
git add .
git commit -m "feat(dashboard): add initial layout"

# 4. Mehr Änderungen
git commit -m "feat(dashboard): add widgets"
git commit -m "feat(dashboard): add data fetching"

# 5. Main in Feature-Branch mergen
git checkout main
git pull origin main
git checkout feature/new-dashboard
git merge main  # Oder: git rebase main

# 6. Push und PR
git push origin feature/new-dashboard
# Pull Request auf GitHub erstellen
```

#### Hotfix-Workflow

```bash
# 1. Von main branchen
git checkout main
git pull origin main
git checkout -b hotfix/critical-bug

# 2. Fix implementieren
git add .
git commit -m "hotfix: resolve critical security issue"

# 3. Zu main mergen (schnell!)
git checkout main
git merge hotfix/critical-bug
git push origin main

# 4. Auch zu develop mergen
git checkout develop
git merge hotfix/critical-bug
git push origin develop

# 5. Hotfix-Branch löschen
git branch -d hotfix/critical-bug
```

### Sicherheits-Best Practices

```bash
# ❌ NIEMALS committen:
# - Passwörter
# - API-Keys
# - Tokens
# - Private Keys
# - Credentials

# ✅ Verwenden Sie:
# - .env-Dateien (in .gitignore!)
# - Environment Variables
# - Secret Management Tools (z.B. GitHub Secrets)
```

**Falls versehentlich committed:**

```bash
# 1. Aus Historie entfernen
git filter-branch --force --index-filter \
  "git rm --cached --ignore-unmatch path/to/secret.txt" \
  --prune-empty --tag-name-filter cat -- --all

# 2. Force Push (VORSICHT!)
git push --force --all

# 3. API-Key sofort ändern/invalidieren!
```

---

## Fortgeschrittene Git-Befehle

### Git Stash

Temporäres Speichern von Änderungen ohne Commit:

```bash
# Änderungen stashen
git stash

# Mit Message
git stash save "WIP: working on feature"

# Stash-Liste anzeigen
git stash list

# Stash anwenden (behält Stash)
git stash apply

# Stash anwenden und entfernen
git stash pop

# Spezifischen Stash anwenden
git stash apply stash@{2}

# Stash löschen
git stash drop stash@{0}

# Alle Stashes löschen
git stash clear

# Stash mit neuen Dateien
git stash -u  # include untracked

# Stash mit ignorierten Dateien
git stash -a  # include all
```

**Häufiger Use Case:**

```bash
# Arbeit auf Feature-Branch
git checkout feature-branch
# ... Code ändern ...

# Plötzlich: Hotfix nötig!
git stash save "WIP: feature implementation"

# Hotfix machen
git checkout main
# ... Hotfix ...
git commit -am "hotfix: critical bug"

# Zurück zu Feature
git checkout feature-branch
git stash pop

# Weiter arbeiten
```

### Cherry-Pick

Einzelne Commits von einem Branch in einen anderen kopieren:

```bash
# Commit-Hash finden
git log --oneline

# Cherry-pick
git checkout main
git cherry-pick abc123

# Mehrere Commits
git cherry-pick abc123 def456 ghi789

# Cherry-pick mit Edit
git cherry-pick -e abc123

# Konflikt beheben
git cherry-pick --continue
# oder abbrechen
git cherry-pick --abort
```

### Git Bisect

Binary Search für Bugs:

```bash
# Bisect starten
git bisect start

# Aktueller Commit ist schlecht (Bug vorhanden)
git bisect bad

# Commit wo es funktioniert hat
git bisect good abc123

# Git checkt mittleren Commit aus
# Testen...

# Wenn Bug vorhanden
git bisect bad

# Wenn Bug nicht vorhanden
git bisect good

# Wiederholen bis Bug gefunden
# Git zeigt den schuldigen Commit

# Bisect beenden
git bisect reset
```

**Automatisiertes Bisect:**

```bash
git bisect start HEAD abc123
git bisect run npm test

# Git führt Tests automatisch aus
# und findet den Bug-Commit
```

### Git Reflog

Historie aller HEAD-Änderungen (auch gelöschte Commits):

```bash
# Reflog anzeigen
git reflog

# Zu bestimmtem State zurückkehren
git reset --hard HEAD@{5}

# Use Case: Versehentlich gelöschten Branch wiederherstellen
git reflog
# Finde den Commit-Hash
git checkout -b recovered-branch abc123
```

### Git Reset vs Revert

#### Reset (ändert Historie)

```bash
# Soft: Behält Änderungen im Staging
git reset --soft HEAD~1

# Mixed (Standard): Behält Änderungen im Working Directory
git reset HEAD~1
git reset --mixed HEAD~1

# Hard: Verwirft alle Änderungen
git reset --hard HEAD~1

# Zu spezifischem Commit
git reset --hard abc123
```

#### Revert (erstellt neuen Commit)

```bash
# Commit rückgängig machen (sicher für öffentliche Branches)
git revert abc123

# Letzten Commit reverten
git revert HEAD

# Mehrere Commits reverten
git revert abc123..def456

# Ohne automatischen Commit
git revert -n abc123
```

**Wann was verwenden?**

- **Reset**: Nur für lokale, nicht-gepushte Commits
- **Revert**: Für gepushte Commits (sicherer)

### Git Tag

Tags für Releases und wichtige Commits:

```bash
# Leichter Tag
git tag v1.0.0

# Annotierter Tag (empfohlen)
git tag -a v1.0.0 -m "Release version 1.0.0"

# Tag für bestimmten Commit
git tag -a v1.0.0 abc123 -m "Release 1.0.0"

# Tags anzeigen
git tag
git tag -l "v1.*"

# Tag-Details anzeigen
git show v1.0.0

# Tag pushen
git push origin v1.0.0

# Alle Tags pushen
git push origin --tags

# Tag löschen
git tag -d v1.0.0

# Remote Tag löschen
git push origin --delete v1.0.0

# Tag auschecken
git checkout v1.0.0
```

### Git Submodules

Andere Repositories als Submodule einbinden:

```bash
# Submodule hinzufügen
git submodule add https://github.com/user/repo.git path/to/submodule

# Repository mit Submodulen klonen
git clone --recursive https://github.com/user/main-repo.git

# Oder nach dem Klonen:
git clone https://github.com/user/main-repo.git
git submodule init
git submodule update

# Submodule aktualisieren
git submodule update --remote

# Alle Submodule durchlaufen
git submodule foreach git pull origin main
```

### Git Worktree

Mehrere Working Directories für ein Repository:

```bash
# Neuen Worktree erstellen
git worktree add ../project-feature feature-branch

# Jetzt kannst du in beiden Verzeichnissen arbeiten:
# - ./project (main)
# - ../project-feature (feature-branch)

# Worktrees anzeigen
git worktree list

# Worktree entfernen
git worktree remove ../project-feature

# Use Case: Gleichzeitig an mehreren Features arbeiten
```

---

## GitHub Features

### GitHub Actions (CI/CD)

Automatisierung direkt in GitHub.

**.github/workflows/ci.yml:**

```yaml
name: CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
    
    - name: Install dependencies
      run: npm ci
    
    - name: Run tests
      run: npm test
    
    - name: Run linter
      run: npm run lint
    
    - name: Build
      run: npm run build

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Deploy to production
      run: |
        echo "Deploying to production..."
        # Dein Deploy-Script
```

### GitHub Packages

NPM-Packages auf GitHub hosten:

```bash
# .npmrc erstellen
echo "@username:registry=https://npm.pkg.github.com" > .npmrc

# In package.json:
{
  "name": "@username/package-name",
  "publishConfig": {
    "registry": "https://npm.pkg.github.com"
  }
}

# Package publishen
npm publish
```

### GitHub Pages

Statische Website direkt aus Repository:

```bash
# 1. Branch erstellen
git checkout -b gh-pages

# 2. HTML-Dateien committen
git add .
git commit -m "Add website files"
git push origin gh-pages

# 3. In GitHub Settings → Pages → Source → gh-pages auswählen

# Website verfügbar unter:
# https://username.github.io/repository-name
```

**Mit GitHub Actions deployen:**

```yaml
# .github/workflows/deploy.yml
name: Deploy to GitHub Pages

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Build
      run: npm run build
    
    - name: Deploy
      uses: peaceiris/actions-gh-pages@v3
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: ./dist
```

### GitHub Discussions

Forum für Community-Diskussionen:

1. Repository → Settings → Features → Discussions aktivieren
2. Discussions-Tab erscheint
3. Kategorien erstellen (Q&A, Ideas, Announcements)

### GitHub Wiki

Dokumentation direkt im Repository:

```bash
# Wiki klonen
git clone https://github.com/username/repo.wiki.git

# Markdown-Dateien erstellen
echo "# Home" > Home.md
git add Home.md
git commit -m "Add home page"
git push
```

### GitHub Security

#### Dependabot

Automatische Dependency-Updates:

**.github/dependabot.yml:**

```yaml
version: 2
updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
    open-pull-requests-limit: 10
```

#### Security Advisories

Private Kommunikation über Sicherheitslücken:

1. Security → Advisories → New draft advisory
2. Details ausfüllen
3. Mit Team koordinieren
4. Publish wenn gefixt

#### Code Scanning

Automatische Code-Analyse:

```yaml
# .github/workflows/codeql.yml
name: "CodeQL"

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  analyze:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - uses: github/codeql-action/init@v2
    - uses: github/codeql-action/analyze@v2
```

### GitHub CLI

GitHub vom Terminal aus steuern:

```bash
# Installation
# Windows: choco install gh
# macOS: brew install gh
# Linux: siehe https://cli.github.com

# Login
gh auth login

# Repository erstellen
gh repo create my-project --public

# Repository klonen
gh repo clone username/repo

# Issue erstellen
gh issue create --title "Bug found" --body "Description"

# Issues anzeigen
gh issue list

# Pull Request erstellen
gh pr create --title "New feature" --body "Description"

# Pull Requests anzeigen
gh pr list

# PR checkout
gh pr checkout 123

# PR mergen
gh pr merge 123

# Workflow anzeigen
gh workflow list
gh workflow run ci.yml

# Release erstellen
gh release create v1.0.0 --notes "Release notes"
```

---

## Praktische Szenarien

### Szenario 1: Feature entwickeln

```bash
# 1. Aktuellen Stand holen
git checkout main
git pull origin main

# 2. Feature-Branch erstellen
git checkout -b feature/shopping-cart

# 3. Entwickeln
# ... Code schreiben ...
git add src/cart.js src/cart.css
git commit -m "feat(cart): add shopping cart component"

# ... mehr Arbeit ...
git add tests/cart.test.js
git commit -m "test(cart): add shopping cart tests"

# 4. Main in Feature-Branch mergen (vor PR)
git checkout main
git pull origin main
git checkout feature/shopping-cart
git rebase main

# 5. Push und PR erstellen
git push origin feature/shopping-cart
# Auf GitHub: Pull Request erstellen

# 6. Nach Review und Merge: Branch löschen
git checkout main
git pull origin main
git branch -d feature/shopping-cart
```

### Szenario 2: Bug fixen

```bash
# 1. Vom main-Branch branchen
git checkout main
git pull origin main
git checkout -b bugfix/checkout-error

# 2. Bug reproduzieren und fixen
# ... Code ändern ...
git add src/checkout.js
git commit -m "fix(checkout): resolve payment processing error"

# 3. Tests hinzufügen
git add tests/checkout.test.js
git commit -m "test(checkout): add regression test for payment error"

# 4. Push und PR
git push origin bugfix/checkout-error
# Pull Request erstellen mit "Fixes #123" in der Beschreibung

# 5. Nach Merge: aufräumen
git checkout main
git pull origin main
git branch -d bugfix/checkout-error
```

### Szenario 3: An fremdem Open-Source-Projekt mitarbeiten

```bash
# 1. Repository forken (auf GitHub)

# 2. Fork klonen
git clone git@github.com:dein-username/project.git
cd project

# 3. Upstream hinzufügen
git remote add upstream git@github.com:original-owner/project.git

# 4. Feature-Branch erstellen
git checkout -b feature/my-contribution

# 5. Entwickeln
git add .
git commit -m "feat: add awesome feature"

# 6. Push zu deinem Fork
git push origin feature/my-contribution

# 7. Pull Request zum Original-Repo erstellen

# 8. Fork aktualisieren
git fetch upstream
git checkout main
git merge upstream/main
git push origin main
```

### Szenario 4: Versehentliche Commits rückgängig machen

```bash
# Letzten Commit rückgängig (Änderungen behalten)
git reset --soft HEAD~1

# Mehrere Commits rückgängig
git reset --soft HEAD~3

# Commit-Message ändern (letzter Commit)
git commit --amend -m "Neue Message"

# Datei zu letztem Commit hinzufügen
git add vergessene-datei.js
git commit --amend --no-edit

# Versehentlich gepushten Commit rückgängig
git revert HEAD
git push origin main
```

### Szenario 5: Großes Refactoring

```bash
# 1. Refactoring-Branch erstellen
git checkout -b refactor/clean-architecture

# 2. Kleine, logische Commits
git commit -m "refactor(models): extract user model"
git commit -m "refactor(services): create user service"
git commit -m "refactor(controllers): update user controller"

# 3. Regelmäßig mit main synchronisieren
git fetch origin
git rebase origin/main

# 4. Tests sicherstellen
npm test

# 5. Push und großen PR mit ausführlicher Beschreibung
git push origin refactor/clean-architecture
```

### Szenario 6: Hotfix während Feature-Entwicklung

```bash
# Situation: Du arbeitest an Feature, Hotfix ist nötig

# 1. Aktuelle Arbeit stashen
git stash save "WIP: feature in progress"

# 2. Hotfix-Branch erstellen
git checkout main
git pull origin main
git checkout -b hotfix/critical-bug

# 3. Hotfix implementieren
git add .
git commit -m "hotfix: resolve critical security issue"

# 4. Push und schneller Merge
git push origin hotfix/critical-bug
# Schnelles Review und Merge

# 5. Zurück zu Feature
git checkout feature-branch
git stash pop

# 6. Hotfix in Feature-Branch mergen
git merge main
```

### Szenario 7: Git-Historie aufräumen vor PR

```bash
# Du hast viele kleine Commits, möchtest sie aber zu wenigen zusammenfassen

# Interaktives Rebase
git rebase -i HEAD~5

# Editor öffnet sich:
# pick abc123 WIP: start feature
# pick def456 fix typo
# pick ghi789 add more code
# pick jkl012 fix bug
# pick mno345 finish feature

# Ändern zu:
# pick abc123 WIP: start feature
# squash def456 fix typo
# squash ghi789 add more code
# squash jkl012 fix bug
# squash mno345 finish feature

# Resultat: Ein einzelner Commit mit kombinierter Message

# Force Push (nur für nicht-geteilte Branches!)
git push --force-with-lease origin feature-branch
```

---

## Zusammenfassung

Du hast jetzt gelernt:

✅ **Git Basics**: Init, add, commit, log, diff
✅ **Branching**: Erstellen, mergen, rebasen
✅ **Remote**: Push, pull, fetch, clone
✅ **GitHub**: Repositories, Issues, Projects
✅ **Collaboration**: Fork-Workflow, Shared-Repo-Workflow
✅ **Pull Requests**: Erstellen, reviewen, mergen
✅ **Konflikte**: Erkennen und lösen
✅ **Best Practices**: Commits, Branches, Workflows
✅ **Fortgeschritten**: Stash, cherry-pick, bisect, reflog
✅ **GitHub Features**: Actions, Pages, CLI

### Git Cheat Sheet

```bash
# Setup
git config --global user.name "Name"
git config --global user.email "email"

# Basics
git init                    # Repo erstellen
git clone <url>            # Repo klonen
git status                 # Status anzeigen
git add .                  # Alle Dateien stagen
git commit -m "message"    # Commit erstellen
git log --oneline          # Historie anzeigen

# Branching
git branch                 # Branches anzeigen
git branch <name>          # Branch erstellen
git checkout <name>        # Branch wechseln
git checkout -b <name>     # Erstellen und wechseln
git merge <branch>         # Branch mergen
git branch -d <name>       # Branch löschen

# Remote
git remote add origin <url>  # Remote hinzufügen
git push origin main         # Push
git pull origin main         # Pull
git fetch origin             # Fetch

# Undo
git restore <file>           # Änderungen verwerfen
git reset --soft HEAD~1      # Commit rückgängig
git revert HEAD              # Commit reverten

# Advanced
git stash                    # Änderungen stashen
git stash pop                # Stash anwenden
git cherry-pick <hash>       # Commit kopieren
git rebase <branch>          # Rebasen
```

### Hilfreiche Ressourcen

- [Git Documentation](https://git-scm.com/doc)
- [GitHub Guides](https://guides.github.com)
- [Git Branching Visualization](https://learngitbranching.js.org)
- [Oh Shit, Git!?!](https://ohshitgit.com/)
- [GitHub Skills](https://skills.github.com)

Viel Erfolg mit Git und GitHub! 🚀
