# CI/CD-Workflows - Kompletter Walkthrough

## Inhaltsverzeichnis
1. [Einführung in CI/CD](#einführung-in-cicd)
2. [CI vs CD vs CD](#ci-vs-cd-vs-cd)
3. [GitHub Actions Grundlagen](#github-actions-grundlagen)
4. [Workflows erstellen](#workflows-erstellen)
5. [Jobs & Steps](#jobs--steps)
6. [Events & Triggers](#events--triggers)
7. [Environment Variables & Secrets](#environment-variables--secrets)
8. [Caching & Artifacts](#caching--artifacts)
9. [Matrix Builds](#matrix-builds)
10. [Testing automatisieren](#testing-automatisieren)
11. [Deployment automatisieren](#deployment-automatisieren)
12. [Docker in CI/CD](#docker-in-cicd)
13. [GitLab CI/CD](#gitlab-cicd)
14. [Best Practices](#best-practices)
15. [Praktische Projekte](#praktische-projekte)

---

## Einführung in CI/CD

### Was ist CI/CD?

**CI/CD** steht für **Continuous Integration** und **Continuous Delivery/Deployment**.

### Probleme ohne CI/CD

❌ Manuelle Tests vor jedem Deploy
❌ "Works on my machine" Probleme
❌ Fehler erst in Production
❌ Zeitaufwändige Deployments
❌ Inkonsistente Builds

### Vorteile mit CI/CD

✅ Automatische Tests bei jedem Push
✅ Frühe Fehlererkennung
✅ Konsistente Builds
✅ Schnellere Deployments
✅ Höhere Code-Qualität
✅ Weniger manuelle Arbeit

### CI/CD Pipeline

```
Code Push
    ↓
Build & Compile
    ↓
Run Tests (Unit, Integration, E2E)
    ↓
Code Quality Checks (Linting, Security)
    ↓
Build Docker Image
    ↓
Deploy to Staging
    ↓
Smoke Tests
    ↓
Deploy to Production
```

---

## CI vs CD vs CD

### Continuous Integration (CI)

**Automatisches Testen und Bauen bei jedem Code-Push**

```yaml
# Beispiel: CI Pipeline
1. Code wird gepusht
2. Tests laufen automatisch
3. Code wird gebaut
4. Feedback an Entwickler
```

**Ziele:**
- Code-Qualität sicherstellen
- Bugs früh finden
- Integration testen

### Continuous Delivery (CD)

**Code ist jederzeit deploybar, aber manuelles Deployment**

```yaml
# Beispiel: Continuous Delivery
1. CI Pipeline läuft durch
2. Build-Artifacts werden erstellt
3. Deploy auf Staging
4. Manueller Button-Click für Production
```

**Ziele:**
- Code ist immer release-ready
- Manuelles Release-Management
- Kontrolle über Production-Deploys

### Continuous Deployment (CD)

**Automatisches Deployment nach erfolgreicher Pipeline**

```yaml
# Beispiel: Continuous Deployment
1. CI Pipeline läuft durch
2. Build-Artifacts werden erstellt
3. Automatisches Deploy auf Staging
4. Automatisches Deploy auf Production
```

**Ziele:**
- Vollständig automatisiert
- Schnellste Time-to-Market
- Keine manuellen Schritte

---

## GitHub Actions Grundlagen

### Was sind GitHub Actions?

GitHub Actions ist eine **CI/CD-Plattform** direkt in GitHub integriert.

### Konzepte

```
Workflow         → YAML-Datei in .github/workflows/
  ├─ Event       → Was triggert den Workflow? (push, PR, schedule)
  ├─ Jobs        → Was wird ausgeführt?
  │   ├─ Steps   → Einzelne Schritte
  │   └─ Actions → Wiederverwendbare Komponenten
  └─ Runners     → Wo läuft es? (Ubuntu, Windows, macOS)
```

### Workflow-Struktur

```yaml
name: CI                    # Workflow-Name

on: [push, pull_request]   # Events, die den Workflow triggern

jobs:                       # Jobs definieren
  build:                    # Job-Name
    runs-on: ubuntu-latest  # Runner-OS
    
    steps:                  # Steps innerhalb des Jobs
      - name: Checkout      # Step-Name
        uses: actions/checkout@v4  # Action verwenden
      
      - name: Run Tests
        run: npm test       # Command ausführen
```

### GitHub Actions Marketplace

Tausende vorgefertigte Actions:
- `actions/checkout@v4` - Code auschecken
- `actions/setup-node@v4` - Node.js installieren
- `actions/cache@v4` - Dependencies cachen
- `docker/build-push-action@v5` - Docker Build & Push

---

## Workflows erstellen

### Basis-Workflow

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run tests
        run: npm test
```

### TypeScript-Projekt

```yaml
# .github/workflows/typescript.yml
name: TypeScript CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Type check
        run: npm run type-check
      
      - name: Lint
        run: npm run lint
      
      - name: Build
        run: npm run build
      
      - name: Run tests
        run: npm test
```

### Next.js-Projekt

```yaml
# .github/workflows/nextjs.yml
name: Next.js CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Lint
        run: npm run lint
      
      - name: Build
        run: npm run build
      
      - name: Run tests
        run: npm test
```

---

## Jobs & Steps

### Mehrere Jobs

```yaml
name: Multi-Job CI

on: [push]

jobs:
  # Job 1: Linting
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run lint
  
  # Job 2: Tests
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm test
  
  # Job 3: Build (abhängig von lint und test)
  build:
    runs-on: ubuntu-latest
    needs: [lint, test]  # Wartet auf lint und test
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run build
```

### Conditional Steps

```yaml
steps:
  - name: Checkout
    uses: actions/checkout@v4
  
  # Step nur auf main Branch
  - name: Deploy to Production
    if: github.ref == 'refs/heads/main'
    run: npm run deploy:prod
  
  # Step nur bei PR
  - name: Comment on PR
    if: github.event_name == 'pull_request'
    run: echo "Thanks for the PR!"
  
  # Step nur bei Erfolg
  - name: Success notification
    if: success()
    run: echo "All tests passed!"
  
  # Step nur bei Fehler
  - name: Failure notification
    if: failure()
    run: echo "Tests failed!"
```

### Continue on Error

```yaml
steps:
  - name: Lint
    run: npm run lint
    continue-on-error: true  # Workflow stoppt nicht bei Fehler
  
  - name: Tests
    run: npm test
```

### Timeout

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    timeout-minutes: 10  # Job-Timeout
    
    steps:
      - name: Long running task
        run: npm test
        timeout-minutes: 5  # Step-Timeout
```

---

## Events & Triggers

### Push Event

```yaml
# Bei jedem Push
on: push

# Nur bestimmte Branches
on:
  push:
    branches:
      - main
      - develop
      - 'feature/*'

# Branches ausschließen
on:
  push:
    branches-ignore:
      - 'experimental/**'

# Nur bei bestimmten Dateien
on:
  push:
    paths:
      - 'src/**'
      - 'package.json'

# Dateien ignorieren
on:
  push:
    paths-ignore:
      - 'docs/**'
      - '**.md'
```

### Pull Request Event

```yaml
on:
  pull_request:
    types:
      - opened
      - synchronize
      - reopened
    branches:
      - main
```

### Schedule (Cron)

```yaml
on:
  schedule:
    # Jeden Tag um 2:00 UTC
    - cron: '0 2 * * *'
    # Jeden Montag um 9:00 UTC
    - cron: '0 9 * * 1'
```

### Manual Trigger (workflow_dispatch)

```yaml
on:
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy to'
        required: true
        type: choice
        options:
          - staging
          - production
      debug:
        description: 'Enable debug logging'
        required: false
        type: boolean
        default: false

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        run: |
          echo "Deploying to ${{ inputs.environment }}"
          echo "Debug: ${{ inputs.debug }}"
```

### Release Event

```yaml
on:
  release:
    types: [published]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy release
        run: echo "Deploying ${{ github.event.release.tag_name }}"
```

### Mehrere Events

```yaml
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  schedule:
    - cron: '0 2 * * *'
  workflow_dispatch:
```

---

## Environment Variables & Secrets

### Environment Variables

```yaml
env:
  # Workflow-level
  NODE_ENV: production
  API_URL: https://api.example.com

jobs:
  build:
    runs-on: ubuntu-latest
    
    env:
      # Job-level
      DATABASE_URL: postgres://localhost/db
    
    steps:
      - name: Build
        env:
          # Step-level
          BUILD_ID: ${{ github.run_number }}
        run: |
          echo "NODE_ENV: $NODE_ENV"
          echo "API_URL: $API_URL"
          echo "DATABASE_URL: $DATABASE_URL"
          echo "BUILD_ID: $BUILD_ID"
```

### GitHub Context Variables

```yaml
steps:
  - name: Print context
    run: |
      echo "Repository: ${{ github.repository }}"
      echo "Branch: ${{ github.ref }}"
      echo "Commit SHA: ${{ github.sha }}"
      echo "Actor: ${{ github.actor }}"
      echo "Event: ${{ github.event_name }}"
      echo "Run ID: ${{ github.run_id }}"
      echo "Run Number: ${{ github.run_number }}"
```

### Secrets

```yaml
# Secrets in Repository Settings → Secrets and variables → Actions

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Deploy
        env:
          API_KEY: ${{ secrets.API_KEY }}
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          JWT_SECRET: ${{ secrets.JWT_SECRET }}
        run: |
          echo "Deploying with API key..."
          # Secrets werden in Logs maskiert: ***
```

### Environments

```yaml
# Settings → Environments → New environment

jobs:
  deploy-staging:
    runs-on: ubuntu-latest
    environment: staging
    steps:
      - name: Deploy
        env:
          API_URL: ${{ secrets.API_URL }}  # staging-spezifisch
        run: npm run deploy
  
  deploy-production:
    runs-on: ubuntu-latest
    environment: production
    needs: deploy-staging
    steps:
      - name: Deploy
        env:
          API_URL: ${{ secrets.API_URL }}  # production-spezifisch
        run: npm run deploy
```

---

## Caching & Artifacts

### Dependency Caching

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      # Node.js mit automatischem Cache
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - run: npm ci
      - run: npm test
```

### Manuelles Caching

```yaml
steps:
  - uses: actions/checkout@v4
  
  - name: Cache node_modules
    uses: actions/cache@v4
    with:
      path: node_modules
      key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
      restore-keys: |
        ${{ runner.os }}-node-
  
  - run: npm ci
  - run: npm test
```

### Multiple Cache Paths

```yaml
- name: Cache dependencies
  uses: actions/cache@v4
  with:
    path: |
      node_modules
      ~/.npm
      .next/cache
    key: ${{ runner.os }}-${{ hashFiles('**/package-lock.json') }}
```

### Artifacts

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run build
      
      # Build-Artifacts hochladen
      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build-files
          path: dist/
          retention-days: 7
  
  deploy:
    runs-on: ubuntu-latest
    needs: build
    steps:
      # Artifacts herunterladen
      - name: Download build artifacts
        uses: actions/download-artifact@v4
        with:
          name: build-files
          path: dist/
      
      - name: Deploy
        run: |
          ls -la dist/
          # Deploy dist/
```

### Test Results

```yaml
- name: Run tests
  run: npm test
  
- name: Upload test results
  if: always()
  uses: actions/upload-artifact@v4
  with:
    name: test-results
    path: |
      coverage/
      test-results.xml
```

---

## Matrix Builds

### Node.js Versionen testen

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    
    strategy:
      matrix:
        node-version: [18, 20, 22]
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      
      - run: npm ci
      - run: npm test
```

### Mehrere Betriebssysteme

```yaml
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
        node-version: [18, 20]
    
    runs-on: ${{ matrix.os }}
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - run: npm ci
      - run: npm test
```

### Komplexe Matrix

```yaml
jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest]
        node-version: [18, 20]
        database: [postgres, mysql]
        include:
          # Extra Kombinationen hinzufügen
          - os: ubuntu-latest
            node-version: 22
            database: postgres
        exclude:
          # Kombinationen ausschließen
          - os: windows-latest
            database: mysql
    
    runs-on: ${{ matrix.os }}
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node-version }}
      - name: Test with ${{ matrix.database }}
        run: npm test
```

### Fail-Fast

```yaml
strategy:
  fail-fast: false  # Alle Matrix-Jobs laufen, auch wenn einer fehlschlägt
  matrix:
    node-version: [18, 20, 22]
```

---

## Testing automatisieren

### Unit Tests

```yaml
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - run: npm ci
      
      - name: Run unit tests
        run: npm test
      
      - name: Upload coverage
        uses: codecov/codecov-action@v4
        with:
          file: ./coverage/coverage-final.json
```

### Integration Tests

```yaml
jobs:
  integration-test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - run: npm ci
      
      - name: Run integration tests
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/test
        run: npm run test:integration
```

### E2E Tests mit Playwright

```yaml
jobs:
  e2e:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - run: npm ci
      
      - name: Install Playwright
        run: npx playwright install --with-deps
      
      - name: Run E2E tests
        run: npm run test:e2e
      
      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/
```

### Test mit Docker Compose

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Start services
        run: docker-compose up -d
      
      - name: Wait for services
        run: |
          sleep 10
          docker-compose ps
      
      - name: Run tests
        run: docker-compose exec -T app npm test
      
      - name: Stop services
        if: always()
        run: docker-compose down
```

---

## Deployment automatisieren

### Deploy zu Vercel

```yaml
name: Deploy to Vercel

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Deploy to Vercel
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.VERCEL_ORG_ID }}
          vercel-project-id: ${{ secrets.VERCEL_PROJECT_ID }}
          vercel-args: '--prod'
```

### Deploy zu Netlify

```yaml
name: Deploy to Netlify

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - run: npm ci
      - run: npm run build
      
      - name: Deploy to Netlify
        uses: nwtgck/actions-netlify@v3
        with:
          publish-dir: './dist'
          production-branch: main
          github-token: ${{ secrets.GITHUB_TOKEN }}
          deploy-message: "Deploy from GitHub Actions"
        env:
          NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
          NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID }}
```

### Deploy zu AWS S3

```yaml
name: Deploy to S3

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      
      - run: npm ci
      - run: npm run build
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: eu-central-1
      
      - name: Deploy to S3
        run: aws s3 sync ./dist s3://my-bucket --delete
```

### Deploy zu Docker Hub

```yaml
name: Build and Push Docker Image

on:
  push:
    branches: [main]
    tags:
      - 'v*'

jobs:
  docker:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: username/my-app
          tags: |
            type=ref,event=branch
            type=semver,pattern={{version}}
            type=semver,pattern={{major}}.{{minor}}
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

---

## Docker in CI/CD

### Docker Build

```yaml
jobs:
  docker:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Build Docker image
        run: docker build -t my-app:latest .
      
      - name: Run tests in container
        run: docker run --rm my-app:latest npm test
```

### Multi-Platform Builds

```yaml
jobs:
  docker:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up QEMU
        uses: docker/setup-qemu-action@v3
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Build multi-platform
        uses: docker/build-push-action@v5
        with:
          context: .
          platforms: linux/amd64,linux/arm64
          push: false
          tags: my-app:latest
```

### Docker Compose in CI

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Build services
        run: docker-compose build
      
      - name: Start services
        run: docker-compose up -d
      
      - name: Wait for services
        run: |
          timeout 60 bash -c 'until docker-compose exec -T app curl -f http://localhost:3000/health; do sleep 1; done'
      
      - name: Run tests
        run: docker-compose exec -T app npm test
      
      - name: Show logs
        if: always()
        run: docker-compose logs
      
      - name: Cleanup
        if: always()
        run: docker-compose down -v
```

---

## GitLab CI/CD

### .gitlab-ci.yml Grundlagen

```yaml
# .gitlab-ci.yml

stages:
  - build
  - test
  - deploy

# Globale Variablen
variables:
  NODE_ENV: production

# Vor allen Jobs
before_script:
  - npm ci

# Build Job
build:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 hour

# Test Job
test:
  stage: test
  script:
    - npm run lint
    - npm test
  coverage: '/Coverage: \d+\.\d+%/'

# Deploy Job
deploy:
  stage: deploy
  script:
    - npm run deploy
  only:
    - main
  environment:
    name: production
    url: https://example.com
```

### Mit Docker

```yaml
image: node:20

stages:
  - test
  - build
  - deploy

test:
  stage: test
  script:
    - npm ci
    - npm test

build-docker:
  stage: build
  image: docker:latest
  services:
    - docker:dind
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
  only:
    - main
```

### GitLab vs GitHub Actions

| Feature | GitLab CI/CD | GitHub Actions |
|---------|--------------|----------------|
| **Config File** | `.gitlab-ci.yml` | `.github/workflows/*.yml` |
| **Syntax** | `stages`, `script` | `jobs`, `steps`, `run` |
| **Runner** | `image: node:20` | `runs-on: ubuntu-latest` |
| **Parallelisierung** | `parallel: 3` | `strategy.matrix` |
| **Artifacts** | `artifacts.paths` | `upload-artifact` |
| **Cache** | `cache.paths` | `actions/cache` |

---

## Best Practices

### 1. Fail Fast

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm run lint  # Schneller Check zuerst
  
  test:
    needs: lint  # Nur wenn lint erfolgreich
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: npm ci
      - run: npm test
```

### 2. Caching nutzen

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'npm'  # Automatisches Caching
```

### 3. Secrets sicher verwenden

```yaml
# ❌ Nicht so
- run: echo "Secret: ${{ secrets.API_KEY }}"

# ✅ Besser
- run: |
    # Secret wird maskiert in Logs
    export API_KEY="${{ secrets.API_KEY }}"
    npm run deploy
```

### 4. Matrix Builds für Kompatibilität

```yaml
strategy:
  matrix:
    node-version: [18, 20, 22]
    os: [ubuntu-latest, windows-latest, macos-latest]
```

### 5. Conditional Deployments

```yaml
deploy:
  if: github.ref == 'refs/heads/main' && github.event_name == 'push'
  steps:
    - run: npm run deploy
```

### 6. Workflow-Zusammenfassungen

```yaml
- name: Add summary
  run: |
    echo "## Test Results" >> $GITHUB_STEP_SUMMARY
    echo "✅ All tests passed" >> $GITHUB_STEP_SUMMARY
    echo "Coverage: 85%" >> $GITHUB_STEP_SUMMARY
```

### 7. Concurrency Control

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true  # Cancel alte Runs
```

### 8. Reusable Workflows

```yaml
# .github/workflows/reusable-test.yml
name: Reusable Test Workflow

on:
  workflow_call:
    inputs:
      node-version:
        required: true
        type: string

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ inputs.node-version }}
      - run: npm ci
      - run: npm test

# .github/workflows/ci.yml
name: CI

on: [push]

jobs:
  test-node-18:
    uses: ./.github/workflows/reusable-test.yml
    with:
      node-version: '18'
  
  test-node-20:
    uses: ./.github/workflows/reusable-test.yml
    with:
      node-version: '20'
```

---

## Praktische Projekte

### Projekt 1: Vollständige CI/CD Pipeline

```yaml
# .github/workflows/complete-pipeline.yml
name: Complete CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  NODE_VERSION: '20'

jobs:
  # 1. Code Quality Checks
  quality:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - run: npm ci
      
      - name: TypeScript check
        run: npm run type-check
      
      - name: Lint
        run: npm run lint
      
      - name: Format check
        run: npm run format:check
  
  # 2. Unit Tests
  test-unit:
    runs-on: ubuntu-latest
    needs: quality
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - run: npm ci
      
      - name: Run unit tests
        run: npm run test:unit
      
      - name: Upload coverage
        uses: codecov/codecov-action@v4
        with:
          files: ./coverage/coverage-final.json
  
  # 3. Integration Tests
  test-integration:
    runs-on: ubuntu-latest
    needs: quality
    
    services:
      postgres:
        image: postgres:16
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: testdb
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 5432:5432
      
      redis:
        image: redis:7
        options: >-
          --health-cmd "redis-cli ping"
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
        ports:
          - 6379:6379
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - run: npm ci
      
      - name: Run integration tests
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/testdb
          REDIS_URL: redis://localhost:6379
        run: npm run test:integration
  
  # 4. E2E Tests
  test-e2e:
    runs-on: ubuntu-latest
    needs: quality
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - run: npm ci
      
      - name: Install Playwright
        run: npx playwright install --with-deps
      
      - name: Build app
        run: npm run build
      
      - name: Run E2E tests
        run: npm run test:e2e
      
      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v4
        with:
          name: playwright-report
          path: playwright-report/
  
  # 5. Build
  build:
    runs-on: ubuntu-latest
    needs: [test-unit, test-integration, test-e2e]
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: ${{ env.NODE_VERSION }}
          cache: 'npm'
      
      - run: npm ci
      
      - name: Build
        run: npm run build
      
      - name: Upload build artifacts
        uses: actions/upload-artifact@v4
        with:
          name: build
          path: dist/
  
  # 6. Docker Build & Push
  docker:
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: myusername/myapp
          tags: |
            type=sha
            type=ref,event=branch
            latest
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
  
  # 7. Deploy to Staging
  deploy-staging:
    runs-on: ubuntu-latest
    needs: docker
    if: github.ref == 'refs/heads/develop'
    environment:
      name: staging
      url: https://staging.example.com
    steps:
      - name: Deploy to staging
        run: |
          echo "Deploying to staging..."
          # Deploy-Script hier
  
  # 8. Deploy to Production
  deploy-production:
    runs-on: ubuntu-latest
    needs: docker
    if: github.ref == 'refs/heads/main'
    environment:
      name: production
      url: https://example.com
    steps:
      - name: Deploy to production
        run: |
          echo "Deploying to production..."
          # Deploy-Script hier
      
      - name: Notify deployment
        run: |
          echo "✅ Deployed to production successfully"
```

### Projekt 2: Monorepo Pipeline

```yaml
# .github/workflows/monorepo.yml
name: Monorepo CI/CD

on: [push, pull_request]

jobs:
  # Detect changed packages
  detect-changes:
    runs-on: ubuntu-latest
    outputs:
      frontend: ${{ steps.changes.outputs.frontend }}
      backend: ${{ steps.changes.outputs.backend }}
      shared: ${{ steps.changes.outputs.shared }}
    steps:
      - uses: actions/checkout@v4
      
      - uses: dorny/paths-filter@v3
        id: changes
        with:
          filters: |
            frontend:
              - 'packages/frontend/**'
            backend:
              - 'packages/backend/**'
            shared:
              - 'packages/shared/**'
  
  # Test Frontend
  test-frontend:
    needs: detect-changes
    if: needs.detect-changes.outputs.frontend == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - run: npm ci
      - run: npm run test --workspace=packages/frontend
  
  # Test Backend
  test-backend:
    needs: detect-changes
    if: needs.detect-changes.outputs.backend == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - run: npm ci
      - run: npm run test --workspace=packages/backend
  
  # Build all if shared changed
  build-all:
    needs: detect-changes
    if: needs.detect-changes.outputs.shared == 'true'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - run: npm ci
      - run: npm run build --workspaces
```

---

## Zusammenfassung

Du hast nun gelernt:

✅ **CI/CD Grundlagen**: Continuous Integration, Delivery, Deployment
✅ **GitHub Actions**: Workflows, Jobs, Steps, Events
✅ **Workflow-Erstellung**: YAML-Syntax, Triggers, Conditions
✅ **Jobs & Steps**: Multiple Jobs, Dependencies, Conditional Execution
✅ **Events**: Push, PR, Schedule, Manual Triggers
✅ **Secrets & Variables**: Environment Variables, GitHub Secrets
✅ **Caching & Artifacts**: Dependencies cachen, Build-Artifacts teilen
✅ **Matrix Builds**: Mehrere Versionen/OS parallel testen
✅ **Testing**: Unit, Integration, E2E automatisieren
✅ **Deployment**: Vercel, Netlify, AWS, Docker Hub
✅ **Docker**: Build, Push, Multi-Platform
✅ **GitLab CI/CD**: Alternative zu GitHub Actions
✅ **Best Practices**: Fail Fast, Caching, Secrets, Reusable Workflows
✅ **Praktische Projekte**: Vollständige Pipeline, Monorepo

### Workflow-Checkliste

Gute CI/CD Pipeline sollte haben:

- [ ] Automatische Tests bei jedem Push
- [ ] Linting & Type Checking
- [ ] Code Coverage
- [ ] Caching für schnellere Builds
- [ ] Matrix Builds für Kompatibilität
- [ ] Conditional Deployments
- [ ] Secrets Management
- [ ] Artifact Management
- [ ] Notifications bei Failures

### Hilfreiche Ressourcen

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [GitHub Actions Marketplace](https://github.com/marketplace?type=actions)
- [GitLab CI/CD Documentation](https://docs.gitlab.com/ee/ci/)
- [Awesome Actions](https://github.com/sdras/awesome-actions)

Viel Erfolg mit CI/CD! 🚀
