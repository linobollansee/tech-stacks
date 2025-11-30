# Docker & Testing - Kompletter Walkthrough

## Inhaltsverzeichnis

### Docker
1. [Docker Einführung](#docker-einführung)
2. [Docker Installation](#docker-installation)
3. [Docker Grundlagen](#docker-grundlagen)
4. [Dockerfile erstellen](#dockerfile-erstellen)
5. [Docker Compose](#docker-compose)
6. [Docker Best Practices](#docker-best-practices)
7. [Multi-Stage Builds](#multi-stage-builds)

### Testing
8. [Testing Grundlagen](#testing-grundlagen)
9. [Unit Testing mit Jest](#unit-testing-mit-jest)
10. [Integration Testing](#integration-testing)
11. [E2E Testing mit Playwright](#e2e-testing-mit-playwright)
12. [Test-Driven Development (TDD)](#test-driven-development-tdd)
13. [Testing Best Practices](#testing-best-practices)
14. [Code Coverage](#code-coverage)
15. [Praktische Projekte](#praktische-projekte)

---

## Docker Einführung

### Was ist Docker?

**Docker** ist eine Plattform zur Containerisierung von Anwendungen. Container sind leichtgewichtige, portable und isolierte Umgebungen.

### Container vs Virtual Machines

```
Virtual Machines:              Docker Containers:
┌─────────────────┐           ┌─────────────────┐
│   App A │ App B │           │ App A │ App B   │
├─────────┼───────┤           ├───────┼─────────┤
│  Guest  │ Guest │           │ Container Runtime│
│   OS    │  OS   │           ├─────────────────┤
├─────────┴───────┤           │    Host OS      │
│   Hypervisor    │           ├─────────────────┤
├─────────────────┤           │   Infrastructure│
│    Host OS      │           └─────────────────┘
├─────────────────┤
│ Infrastructure  │
└─────────────────┘

VMs: Mehrere OS          Container: Ein OS, isoliert
Größe: GBs               Größe: MBs
Start: Minuten           Start: Sekunden
```

### Warum Docker?

✅ **Konsistenz**: "Works on my machine" Problem gelöst
✅ **Isolation**: Jede App in eigener Umgebung
✅ **Portabilität**: Läuft überall (Dev, Test, Prod)
✅ **Skalierung**: Schnelles Hoch-/Runterskalieren
✅ **Effizienz**: Weniger Ressourcen als VMs
✅ **Versionierung**: Images sind versioniert

### Docker-Komponenten

- **Image**: Blueprint für Container (wie eine Klasse)
- **Container**: Laufende Instanz eines Images (wie ein Objekt)
- **Dockerfile**: Anweisungen zum Erstellen eines Images
- **Docker Hub**: Registry für Images (wie npm für Packages)
- **Docker Compose**: Tool für Multi-Container-Apps

---

## Docker Installation

### Windows

```powershell
# Docker Desktop herunterladen
# https://www.docker.com/products/docker-desktop/

# Nach Installation WSL2 aktivieren (falls nicht aktiv)
wsl --install

# Docker starten und testen
docker --version
docker run hello-world
```

### macOS

```bash
# Docker Desktop herunterladen
# https://www.docker.com/products/docker-desktop/

# Mit Homebrew
brew install --cask docker

# Docker starten und testen
docker --version
docker run hello-world
```

### Linux (Ubuntu/Debian)

```bash
# Repository einrichten
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg

# Docker GPG Key hinzufügen
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

# Repository hinzufügen
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# Docker installieren
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Ohne sudo verwenden
sudo usermod -aG docker $USER

# Docker testen
docker --version
docker run hello-world
```

### Installation verifizieren

```bash
# Version anzeigen
docker --version

# Docker Info
docker info

# Test-Container ausführen
docker run hello-world
```

---

## Docker Grundlagen

### Images verwalten

```bash
# Images herunterladen
docker pull nginx
docker pull node:18
docker pull postgres:16

# Lokale Images anzeigen
docker images
docker image ls

# Image Details anzeigen
docker image inspect nginx

# Image löschen
docker rmi nginx
docker image rm nginx

# Ungenutzte Images löschen
docker image prune
```

### Container verwalten

```bash
# Container starten
docker run nginx

# Container im Hintergrund starten (-d = detached)
docker run -d nginx

# Container mit Namen
docker run -d --name my-nginx nginx

# Container mit Port-Mapping
docker run -d -p 8080:80 nginx
# Host:Container -> localhost:8080 → Container:80

# Container mit Environment-Variablen
docker run -d -e POSTGRES_PASSWORD=secret postgres

# Laufende Container anzeigen
docker ps

# Alle Container anzeigen (auch gestoppte)
docker ps -a

# Container stoppen
docker stop my-nginx

# Container starten
docker start my-nginx

# Container neustarten
docker restart my-nginx

# Container löschen
docker rm my-nginx

# Laufenden Container löschen (force)
docker rm -f my-nginx

# Alle gestoppten Container löschen
docker container prune
```

### Container interaktiv verwenden

```bash
# Interaktive Shell (-it)
docker run -it ubuntu bash

# In laufenden Container einsteigen
docker exec -it my-nginx bash

# Kommando im Container ausführen
docker exec my-nginx ls /usr/share/nginx/html

# Container-Logs anzeigen
docker logs my-nginx

# Logs in Echtzeit verfolgen
docker logs -f my-nginx

# Letzte 100 Zeilen
docker logs --tail 100 my-nginx
```

### Volumes (Persistente Daten)

```bash
# Named Volume erstellen
docker volume create my-data

# Volumes anzeigen
docker volume ls

# Container mit Volume
docker run -d -v my-data:/data nginx

# Bind Mount (lokaler Ordner)
docker run -d -v /path/on/host:/path/in/container nginx

# Windows PowerShell (aktuelles Verzeichnis)
docker run -d -v ${PWD}:/app node:18

# Volume löschen
docker volume rm my-data

# Ungenutzte Volumes löschen
docker volume prune
```

### Netzwerk

```bash
# Netzwerke anzeigen
docker network ls

# Netzwerk erstellen
docker network create my-network

# Container in Netzwerk starten
docker run -d --network my-network --name db postgres
docker run -d --network my-network --name app node:18

# Im Container "app" kann man jetzt "db" als Hostname verwenden

# Netzwerk löschen
docker network rm my-network
```

---

## Dockerfile erstellen

### Basis-Dockerfile

```dockerfile
# Basis-Image
FROM node:18

# Working Directory setzen
WORKDIR /app

# Package.json kopieren
COPY package*.json ./

# Dependencies installieren
RUN npm install

# App-Code kopieren
COPY . .

# Port exposieren (dokumentarisch)
EXPOSE 3000

# Start-Kommando
CMD ["npm", "start"]
```

### Dockerfile Instructions

```dockerfile
# FROM - Basis-Image
FROM node:18-alpine

# LABEL - Metadata
LABEL maintainer="max@example.com"
LABEL version="1.0"

# ENV - Environment-Variablen
ENV NODE_ENV=production
ENV PORT=3000

# WORKDIR - Working Directory
WORKDIR /app

# COPY - Dateien kopieren
COPY package.json .
COPY src/ ./src/

# ADD - Dateien kopieren + entpacken
ADD archive.tar.gz /app

# RUN - Befehle ausführen (Build-Time)
RUN npm install
RUN npm run build

# EXPOSE - Port dokumentieren
EXPOSE 3000

# USER - User wechseln (Security)
USER node

# CMD - Default-Kommando (Runtime)
CMD ["node", "server.js"]

# ENTRYPOINT - Fixiertes Kommando
ENTRYPOINT ["npm", "run"]
CMD ["start"]  # Default Argument für ENTRYPOINT

# VOLUME - Volume Mount Point
VOLUME /data

# ARG - Build-Zeit Variablen
ARG VERSION=1.0
RUN echo "Building version ${VERSION}"
```

### Image bauen

```bash
# Image bauen
docker build -t my-app:1.0 .

# Mit anderem Dockerfile
docker build -f Dockerfile.dev -t my-app:dev .

# Build-Argument übergeben
docker build --build-arg VERSION=2.0 -t my-app:2.0 .

# Ohne Cache
docker build --no-cache -t my-app:1.0 .

# Image starten
docker run -d -p 3000:3000 my-app:1.0
```

### .dockerignore

```
# .dockerignore
node_modules
npm-debug.log
.git
.gitignore
.env
.DS_Store
*.md
Dockerfile
docker-compose.yml
dist
build
coverage
.vscode
.idea
```

---

## Docker Compose

### Was ist Docker Compose?

Docker Compose ermöglicht es, Multi-Container-Anwendungen zu definieren und zu starten.

### docker-compose.yml

```yaml
version: '3.8'

services:
  # PostgreSQL Datenbank
  db:
    image: postgres:16
    container_name: my-postgres
    environment:
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
      POSTGRES_DB: mydb
    volumes:
      - postgres-data:/var/lib/postgresql/data
    ports:
      - "5432:5432"
    networks:
      - app-network

  # Redis Cache
  redis:
    image: redis:7-alpine
    container_name: my-redis
    ports:
      - "6379:6379"
    networks:
      - app-network

  # Node.js API
  api:
    build:
      context: ./api
      dockerfile: Dockerfile
    container_name: my-api
    environment:
      NODE_ENV: development
      DATABASE_URL: postgresql://myuser:mypassword@db:5432/mydb
      REDIS_URL: redis://redis:6379
    ports:
      - "3000:3000"
    volumes:
      - ./api:/app
      - /app/node_modules
    depends_on:
      - db
      - redis
    networks:
      - app-network
    command: npm run dev

  # Next.js Frontend
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    container_name: my-frontend
    environment:
      NEXT_PUBLIC_API_URL: http://localhost:3000
    ports:
      - "3001:3000"
    volumes:
      - ./frontend:/app
      - /app/node_modules
      - /app/.next
    depends_on:
      - api
    networks:
      - app-network
    command: npm run dev

volumes:
  postgres-data:

networks:
  app-network:
    driver: bridge
```

### Docker Compose Befehle

```bash
# Services starten
docker-compose up

# Im Hintergrund starten
docker-compose up -d

# Bestimmten Service starten
docker-compose up db

# Services stoppen
docker-compose down

# Mit Volumes löschen
docker-compose down -v

# Services neu bauen
docker-compose build

# Bauen und starten
docker-compose up --build

# Logs anzeigen
docker-compose logs

# Logs eines Services
docker-compose logs api

# Logs in Echtzeit
docker-compose logs -f

# Laufende Services anzeigen
docker-compose ps

# In Service einsteigen
docker-compose exec api bash

# Kommando in Service ausführen
docker-compose exec db psql -U myuser -d mydb

# Services neustarten
docker-compose restart

# Service skalieren
docker-compose up -d --scale api=3
```

### Environment-Variablen (.env)

```env
# .env
POSTGRES_USER=myuser
POSTGRES_PASSWORD=mypassword
POSTGRES_DB=mydb
NODE_ENV=development
API_PORT=3000
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  db:
    image: postgres:16
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
```

---

## Docker Best Practices

### 1. Kleine Images verwenden

```dockerfile
# ❌ Groß (1GB+)
FROM node:18

# ✅ Klein (100MB)
FROM node:18-alpine
```

### 2. Layer Caching optimieren

```dockerfile
# ❌ Schlecht - Jede Code-Änderung installiert Dependencies neu
FROM node:18-alpine
WORKDIR /app
COPY . .
RUN npm install
CMD ["npm", "start"]

# ✅ Gut - Dependencies werden gecacht
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["npm", "start"]
```

### 3. .dockerignore verwenden

```
# .dockerignore
node_modules
.git
.env
*.log
coverage
.vscode
```

### 4. Non-root User verwenden

```dockerfile
FROM node:18-alpine

# User erstellen
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nodejs -u 1001

WORKDIR /app
COPY --chown=nodejs:nodejs . .

# Als non-root User laufen
USER nodejs

CMD ["node", "server.js"]
```

### 5. Multi-Stage für Production

```dockerfile
# Build Stage
FROM node:18-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
RUN npm run build

# Production Stage
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
USER node
CMD ["node", "dist/server.js"]
```

### 6. Health Checks

```dockerfile
FROM node:18-alpine

WORKDIR /app
COPY . .

# Health Check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD node healthcheck.js

CMD ["node", "server.js"]
```

```javascript
// healthcheck.js
const http = require('http');

const options = {
  host: 'localhost',
  port: 3000,
  path: '/health',
  timeout: 2000
};

const request = http.request(options, (res) => {
  if (res.statusCode == 200) {
    process.exit(0);
  } else {
    process.exit(1);
  }
});

request.on('error', () => process.exit(1));
request.end();
```

### 7. Secrets nicht committen

```dockerfile
# ❌ Niemals so
ENV API_KEY=secret123

# ✅ Via Environment beim Start
docker run -e API_KEY=secret123 my-app

# ✅ Via Docker Secrets (Swarm)
docker secret create api_key secret.txt
```

---

## Multi-Stage Builds

### Next.js Production Build

```dockerfile
# Dockerfile für Next.js
FROM node:18-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

FROM node:18-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
RUN npm run build

FROM node:18-alpine AS runner
WORKDIR /app

ENV NODE_ENV production

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000

ENV PORT 3000

CMD ["node", "server.js"]
```

### TypeScript Backend

```dockerfile
FROM node:18-alpine AS builder

WORKDIR /app

COPY package*.json ./
COPY tsconfig.json ./

RUN npm ci

COPY src ./src

RUN npm run build

FROM node:18-alpine AS runner

WORKDIR /app

ENV NODE_ENV=production

RUN addgroup -g 1001 -S nodejs && \
    adduser -S nodejs -u 1001

COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./

USER nodejs

EXPOSE 3000

CMD ["node", "dist/index.js"]
```

---

## Testing Grundlagen

### Was ist Testing?

Testing ist der Prozess, Code auf Korrektheit, Qualität und Funktionalität zu überprüfen.

### Warum Testen?

✅ **Fehler früh finden**: Billiger zu fixen
✅ **Refactoring-Sicherheit**: Änderungen ohne Angst
✅ **Dokumentation**: Tests zeigen, wie Code funktioniert
✅ **Code-Qualität**: Testbarer Code ist besserer Code
✅ **Regression Prevention**: Alte Bugs bleiben gefixt

### Test-Pyramide

```
        /\
       /E2E\      <- Wenige, langsame, teure Tests
      /------\
     /Integr.\   <- Mittlere Anzahl
    /----------\
   /   Unit     \ <- Viele, schnelle, günstige Tests
  /--------------\
```

### Test-Arten

#### Unit Tests
- Testen einzelne Funktionen/Methoden
- Sehr schnell
- Keine Abhängigkeiten
- Meiste Tests sollten Unit Tests sein

#### Integration Tests
- Testen Zusammenspiel mehrerer Komponenten
- Mittelschnell
- Mit echten/Mock-Abhängigkeiten
- Wichtig für API-Endpunkte

#### E2E Tests (End-to-End)
- Testen komplette User-Flows
- Langsam
- Im echten Browser
- Wenige, aber kritische Tests

---

## Unit Testing mit Jest

### Installation

```bash
npm install --save-dev jest @types/jest ts-jest

# Für TypeScript
npx ts-jest config:init
```

### Jest konfigurieren

```javascript
// jest.config.js
module.exports = {
  preset: 'ts-jest',
  testEnvironment: 'node',
  roots: ['<rootDir>/src'],
  testMatch: ['**/__tests__/**/*.ts', '**/?(*.)+(spec|test).ts'],
  transform: {
    '^.+\\.ts$': 'ts-jest',
  },
  collectCoverageFrom: [
    'src/**/*.ts',
    '!src/**/*.d.ts',
    '!src/**/*.test.ts',
  ],
};
```

```json
// package.json
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  }
}
```

### Basis-Test

```typescript
// src/utils/math.ts
export function add(a: number, b: number): number {
  return a + b;
}

export function subtract(a: number, b: number): number {
  return a - b;
}

export function multiply(a: number, b: number): number {
  return a * b;
}

export function divide(a: number, b: number): number {
  if (b === 0) {
    throw new Error('Division by zero');
  }
  return a / b;
}
```

```typescript
// src/utils/math.test.ts
import { add, subtract, multiply, divide } from './math';

describe('Math Utils', () => {
  describe('add', () => {
    it('should add two positive numbers', () => {
      expect(add(2, 3)).toBe(5);
    });

    it('should add negative numbers', () => {
      expect(add(-2, -3)).toBe(-5);
    });

    it('should add zero', () => {
      expect(add(5, 0)).toBe(5);
    });
  });

  describe('subtract', () => {
    it('should subtract two numbers', () => {
      expect(subtract(5, 3)).toBe(2);
    });
  });

  describe('multiply', () => {
    it('should multiply two numbers', () => {
      expect(multiply(3, 4)).toBe(12);
    });
  });

  describe('divide', () => {
    it('should divide two numbers', () => {
      expect(divide(10, 2)).toBe(5);
    });

    it('should throw error on division by zero', () => {
      expect(() => divide(10, 0)).toThrow('Division by zero');
    });
  });
});
```

### Matchers (Assertions)

```typescript
// Gleichheit
expect(value).toBe(5);              // Exakte Gleichheit (===)
expect(value).toEqual({ a: 1 });    // Deep Equality für Objekte
expect(value).not.toBe(10);         // Negation

// Wahrheitswerte
expect(value).toBeTruthy();
expect(value).toBeFalsy();
expect(value).toBeNull();
expect(value).toBeUndefined();
expect(value).toBeDefined();

// Zahlen
expect(value).toBeGreaterThan(10);
expect(value).toBeGreaterThanOrEqual(10);
expect(value).toBeLessThan(10);
expect(value).toBeLessThanOrEqual(10);
expect(value).toBeCloseTo(0.3, 1); // Für Floats

// Strings
expect(text).toMatch(/pattern/);
expect(text).toContain('substring');

// Arrays
expect(array).toContain('item');
expect(array).toHaveLength(3);
expect(array).toEqual(expect.arrayContaining([1, 2]));

// Objekte
expect(obj).toHaveProperty('key');
expect(obj).toHaveProperty('key', 'value');
expect(obj).toMatchObject({ a: 1 });

// Funktionen
expect(fn).toThrow();
expect(fn).toThrow('error message');
expect(fn).toThrow(Error);
expect(fn).toHaveBeenCalled();
expect(fn).toHaveBeenCalledWith(arg1, arg2);
expect(fn).toHaveBeenCalledTimes(2);
```

### Async Testing

```typescript
// src/services/user.service.ts
export async function fetchUser(id: number) {
  const response = await fetch(`https://api.example.com/users/${id}`);
  if (!response.ok) {
    throw new Error('User not found');
  }
  return response.json();
}
```

```typescript
// src/services/user.service.test.ts
import { fetchUser } from './user.service';

describe('UserService', () => {
  describe('fetchUser', () => {
    it('should fetch user by id', async () => {
      // Mit async/await
      const user = await fetchUser(1);
      expect(user).toHaveProperty('id', 1);
    });

    it('should throw error for invalid user', async () => {
      // Mit async/await und toThrow
      await expect(fetchUser(999)).rejects.toThrow('User not found');
    });

    it('should fetch user using promises', () => {
      // Mit Promises
      return fetchUser(1).then(user => {
        expect(user).toHaveProperty('id', 1);
      });
    });
  });
});
```

### Mocking

```typescript
// src/services/email.service.ts
export class EmailService {
  async sendEmail(to: string, subject: string, body: string): Promise<void> {
    // Echte Email-Logik
    console.log(`Sending email to ${to}`);
  }
}
```

```typescript
// src/services/user.service.ts
import { EmailService } from './email.service';

export class UserService {
  constructor(private emailService: EmailService) {}

  async createUser(email: string, name: string) {
    const user = { id: 1, email, name };
    
    await this.emailService.sendEmail(
      email,
      'Welcome!',
      `Hello ${name}`
    );
    
    return user;
  }
}
```

```typescript
// src/services/user.service.test.ts
import { UserService } from './user.service';
import { EmailService } from './email.service';

// Mock erstellen
jest.mock('./email.service');

describe('UserService', () => {
  let userService: UserService;
  let emailService: jest.Mocked<EmailService>;

  beforeEach(() => {
    // Mock-Instanz erstellen
    emailService = new EmailService() as jest.Mocked<EmailService>;
    userService = new UserService(emailService);
  });

  it('should create user and send email', async () => {
    const user = await userService.createUser('test@example.com', 'Test');

    expect(user).toEqual({
      id: 1,
      email: 'test@example.com',
      name: 'Test'
    });

    expect(emailService.sendEmail).toHaveBeenCalledWith(
      'test@example.com',
      'Welcome!',
      'Hello Test'
    );
  });
});
```

### Test Hooks

```typescript
describe('Test Suite', () => {
  // Vor allen Tests (einmal)
  beforeAll(() => {
    console.log('Setup before all tests');
  });

  // Nach allen Tests (einmal)
  afterAll(() => {
    console.log('Cleanup after all tests');
  });

  // Vor jedem Test
  beforeEach(() => {
    console.log('Setup before each test');
  });

  // Nach jedem Test
  afterEach(() => {
    console.log('Cleanup after each test');
  });

  it('test 1', () => {
    expect(true).toBe(true);
  });

  it('test 2', () => {
    expect(true).toBe(true);
  });
});
```

---

## Integration Testing

### API Testing mit Supertest

```bash
npm install --save-dev supertest @types/supertest
```

```typescript
// src/app.ts
import express from 'express';

const app = express();
app.use(express.json());

app.get('/health', (req, res) => {
  res.json({ status: 'ok' });
});

app.post('/users', (req, res) => {
  const { name, email } = req.body;
  
  if (!name || !email) {
    return res.status(400).json({ error: 'Name and email required' });
  }
  
  const user = { id: 1, name, email };
  res.status(201).json(user);
});

export default app;
```

```typescript
// src/app.test.ts
import request from 'supertest';
import app from './app';

describe('API Integration Tests', () => {
  describe('GET /health', () => {
    it('should return health status', async () => {
      const response = await request(app)
        .get('/health')
        .expect(200);

      expect(response.body).toEqual({ status: 'ok' });
    });
  });

  describe('POST /users', () => {
    it('should create a new user', async () => {
      const userData = {
        name: 'Max Mustermann',
        email: 'max@example.com'
      };

      const response = await request(app)
        .post('/users')
        .send(userData)
        .expect(201);

      expect(response.body).toMatchObject({
        id: expect.any(Number),
        name: userData.name,
        email: userData.email
      });
    });

    it('should return 400 for missing data', async () => {
      const response = await request(app)
        .post('/users')
        .send({ name: 'Max' })
        .expect(400);

      expect(response.body).toHaveProperty('error');
    });
  });
});
```

### Database Testing

```typescript
// src/__tests__/setup.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

beforeAll(async () => {
  // Test-Datenbank vorbereiten
  await prisma.$connect();
});

afterAll(async () => {
  // Aufräumen
  await prisma.user.deleteMany();
  await prisma.$disconnect();
});

beforeEach(async () => {
  // Vor jedem Test aufräumen
  await prisma.user.deleteMany();
});
```

```typescript
// src/services/user.service.test.ts
import { PrismaClient } from '@prisma/client';
import { UserService } from './user.service';

const prisma = new PrismaClient();
const userService = new UserService(prisma);

describe('UserService Integration Tests', () => {
  beforeEach(async () => {
    await prisma.user.deleteMany();
  });

  describe('createUser', () => {
    it('should create user in database', async () => {
      const userData = {
        email: 'test@example.com',
        name: 'Test User'
      };

      const user = await userService.createUser(userData);

      expect(user).toMatchObject(userData);
      expect(user.id).toBeDefined();

      // Verifizieren in DB
      const dbUser = await prisma.user.findUnique({
        where: { id: user.id }
      });

      expect(dbUser).toMatchObject(userData);
    });

    it('should throw error for duplicate email', async () => {
      const userData = {
        email: 'test@example.com',
        name: 'Test User'
      };

      await userService.createUser(userData);

      await expect(
        userService.createUser(userData)
      ).rejects.toThrow();
    });
  });
});
```

---

## E2E Testing mit Playwright

### Installation

```bash
npm install --save-dev @playwright/test
npx playwright install
```

### Playwright konfigurieren

```typescript
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test';

export default defineConfig({
  testDir: './e2e',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },

  projects: [
    {
      name: 'chromium',
      use: { ...devices['Desktop Chrome'] },
    },
    {
      name: 'firefox',
      use: { ...devices['Desktop Firefox'] },
    },
    {
      name: 'webkit',
      use: { ...devices['Desktop Safari'] },
    },
  ],

  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
  },
});
```

### Basis E2E Test

```typescript
// e2e/homepage.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Homepage', () => {
  test('should display title', async ({ page }) => {
    await page.goto('/');
    
    await expect(page).toHaveTitle(/My App/);
  });

  test('should have navigation links', async ({ page }) => {
    await page.goto('/');
    
    await expect(page.getByRole('link', { name: 'Home' })).toBeVisible();
    await expect(page.getByRole('link', { name: 'About' })).toBeVisible();
  });
});
```

### Login Flow testen

```typescript
// e2e/auth.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Authentication', () => {
  test('should login successfully', async ({ page }) => {
    await page.goto('/login');

    // Formular ausfüllen
    await page.fill('input[name="email"]', 'test@example.com');
    await page.fill('input[name="password"]', 'password123');

    // Submit klicken
    await page.click('button[type="submit"]');

    // Auf Weiterleitung warten
    await page.waitForURL('/dashboard');

    // Erfolgsmeldung prüfen
    await expect(page.getByText('Welcome back!')).toBeVisible();
  });

  test('should show error for invalid credentials', async ({ page }) => {
    await page.goto('/login');

    await page.fill('input[name="email"]', 'wrong@example.com');
    await page.fill('input[name="password"]', 'wrongpassword');
    await page.click('button[type="submit"]');

    // Fehlermeldung prüfen
    await expect(page.getByText('Invalid credentials')).toBeVisible();
    
    // Immer noch auf Login-Seite
    expect(page.url()).toContain('/login');
  });
});
```

### Interaktive Tests

```typescript
// e2e/todo.spec.ts
import { test, expect } from '@playwright/test';

test.describe('Todo App', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('/todos');
  });

  test('should add new todo', async ({ page }) => {
    // Input finden und ausfüllen
    const input = page.getByPlaceholder('Add a todo');
    await input.fill('Buy groceries');
    await input.press('Enter');

    // Neues Todo sollte sichtbar sein
    await expect(page.getByText('Buy groceries')).toBeVisible();
  });

  test('should mark todo as complete', async ({ page }) => {
    // Todo hinzufügen
    await page.getByPlaceholder('Add a todo').fill('Test todo');
    await page.getByPlaceholder('Add a todo').press('Enter');

    // Checkbox klicken
    const checkbox = page.getByRole('checkbox').first();
    await checkbox.check();

    // Todo sollte durchgestrichen sein
    const todo = page.getByText('Test todo');
    await expect(todo).toHaveCSS('text-decoration', /line-through/);
  });

  test('should delete todo', async ({ page }) => {
    // Todo hinzufügen
    await page.getByPlaceholder('Add a todo').fill('Delete me');
    await page.getByPlaceholder('Add a todo').press('Enter');

    // Delete-Button klicken
    await page.getByRole('button', { name: 'Delete' }).click();

    // Todo sollte nicht mehr da sein
    await expect(page.getByText('Delete me')).not.toBeVisible();
  });
});
```

### Screenshots und Videos

```typescript
test('should take screenshot on failure', async ({ page }) => {
  await page.goto('/');
  
  // Screenshot manuell erstellen
  await page.screenshot({ path: 'screenshot.png' });
  
  // Fullpage Screenshot
  await page.screenshot({ path: 'fullpage.png', fullPage: true });
});

// Automatisch bei Fehlern (in playwright.config.ts)
use: {
  screenshot: 'only-on-failure',
  video: 'retain-on-failure',
}
```

---

## Test-Driven Development (TDD)

### TDD-Zyklus: Red-Green-Refactor

```
1. RED    → Test schreiben (schlägt fehl)
2. GREEN  → Minimalen Code schreiben (Test besteht)
3. REFACTOR → Code verbessern
```

### TDD Beispiel: String Calculator

```typescript
// Schritt 1: RED - Test schreiben
// src/calculator.test.ts
import { StringCalculator } from './calculator';

describe('StringCalculator', () => {
  let calculator: StringCalculator;

  beforeEach(() => {
    calculator = new StringCalculator();
  });

  it('should return 0 for empty string', () => {
    expect(calculator.add('')).toBe(0);
  });
});

// Test läuft → FEHLER (Klasse existiert nicht)
```

```typescript
// Schritt 2: GREEN - Minimaler Code
// src/calculator.ts
export class StringCalculator {
  add(numbers: string): number {
    return 0;
  }
}

// Test läuft → ERFOLG
```

```typescript
// Schritt 3: Nächster Test (RED)
it('should return the number itself for single number', () => {
  expect(calculator.add('5')).toBe(5);
});

// Test läuft → FEHLER
```

```typescript
// Schritt 4: Code erweitern (GREEN)
export class StringCalculator {
  add(numbers: string): number {
    if (numbers === '') return 0;
    return parseInt(numbers);
  }
}

// Test läuft → ERFOLG
```

```typescript
// Schritt 5: Mehr Tests
it('should sum two numbers', () => {
  expect(calculator.add('1,2')).toBe(3);
});

it('should sum multiple numbers', () => {
  expect(calculator.add('1,2,3,4,5')).toBe(15);
});

// Tests laufen → FEHLER
```

```typescript
// Schritt 6: Implementierung
export class StringCalculator {
  add(numbers: string): number {
    if (numbers === '') return 0;
    
    const nums = numbers.split(',').map(n => parseInt(n));
    return nums.reduce((sum, n) => sum + n, 0);
  }
}

// Tests laufen → ERFOLG
```

```typescript
// Schritt 7: REFACTOR - Code verbessern
export class StringCalculator {
  add(numbers: string): number {
    if (!numbers) return 0;
    
    return this.parseNumbers(numbers)
      .reduce((sum, n) => sum + n, 0);
  }

  private parseNumbers(numbers: string): number[] {
    return numbers.split(',').map(Number);
  }
}

// Tests laufen → Immer noch ERFOLG
```

---

## Testing Best Practices

### 1. AAA Pattern (Arrange-Act-Assert)

```typescript
test('should calculate discount', () => {
  // Arrange - Setup
  const product = { price: 100 };
  const discountPercent = 20;

  // Act - Ausführen
  const result = calculateDiscount(product.price, discountPercent);

  // Assert - Überprüfen
  expect(result).toBe(80);
});
```

### 2. Ein Konzept pro Test

```typescript
// ❌ Schlecht - Mehrere Konzepte
test('user operations', () => {
  const user = createUser();
  expect(user).toBeDefined();
  
  updateUser(user.id, { name: 'New Name' });
  expect(user.name).toBe('New Name');
  
  deleteUser(user.id);
  expect(getUser(user.id)).toBeNull();
});

// ✅ Gut - Separate Tests
test('should create user', () => {
  const user = createUser();
  expect(user).toBeDefined();
});

test('should update user', () => {
  const user = createUser();
  updateUser(user.id, { name: 'New Name' });
  expect(user.name).toBe('New Name');
});

test('should delete user', () => {
  const user = createUser();
  deleteUser(user.id);
  expect(getUser(user.id)).toBeNull();
});
```

### 3. Beschreibende Test-Namen

```typescript
// ❌ Schlecht
test('user test 1', () => {});
test('test', () => {});

// ✅ Gut
test('should create user with valid data', () => {});
test('should throw error when email is invalid', () => {});
test('should return empty array when no users exist', () => {});
```

### 4. Unabhängige Tests

```typescript
// ❌ Schlecht - Tests hängen voneinander ab
let userId: number;

test('create user', () => {
  userId = createUser();
  expect(userId).toBeDefined();
});

test('get user', () => {
  const user = getUser(userId); // Hängt von vorherigem Test ab!
  expect(user).toBeDefined();
});

// ✅ Gut - Unabhängige Tests
test('should create user', () => {
  const userId = createUser();
  expect(userId).toBeDefined();
});

test('should get user by id', () => {
  const userId = createUser(); // Eigenes Setup
  const user = getUser(userId);
  expect(user).toBeDefined();
});
```

### 5. Test Doubles verwenden

```typescript
// Mock - Vollständiges Objekt ersetzen
const mockEmailService = {
  sendEmail: jest.fn()
};

// Stub - Vordefinierte Antworten
const stubUserRepository = {
  findById: jest.fn().mockResolvedValue({ id: 1, name: 'Test' })
};

// Spy - Echtes Objekt beobachten
const spy = jest.spyOn(service, 'method');
```

---

## Code Coverage

### Coverage aktivieren

```bash
npm test -- --coverage
```

```json
// package.json
{
  "scripts": {
    "test:coverage": "jest --coverage"
  },
  "jest": {
    "collectCoverageFrom": [
      "src/**/*.{ts,tsx}",
      "!src/**/*.d.ts",
      "!src/**/*.test.{ts,tsx}"
    ],
    "coverageThreshold": {
      "global": {
        "branches": 80,
        "functions": 80,
        "lines": 80,
        "statements": 80
      }
    }
  }
}
```

### Coverage-Report

```
-----------------|---------|----------|---------|---------|
File             | % Stmts | % Branch | % Funcs | % Lines |
-----------------|---------|----------|---------|---------|
All files        |   85.71 |       75 |      80 |   85.71 |
 calculator.ts   |     100 |      100 |     100 |     100 |
 user.service.ts |   71.42 |       50 |      60 |   71.42 |
-----------------|---------|----------|---------|---------|
```

**Metriken:**
- **Statements**: Prozentsatz ausgeführter Anweisungen
- **Branches**: Prozentsatz getesteter if/else-Zweige
- **Functions**: Prozentsatz aufgerufener Funktionen
- **Lines**: Prozentsatz ausgeführter Zeilen

---

## Praktische Projekte

### Projekt 1: API mit Tests

```typescript
// src/app.ts
import express from 'express';
import { PrismaClient } from '@prisma/client';

const app = express();
const prisma = new PrismaClient();

app.use(express.json());

app.post('/api/users', async (req, res) => {
  try {
    const { email, name } = req.body;
    
    if (!email || !name) {
      return res.status(400).json({ error: 'Email and name required' });
    }

    const user = await prisma.user.create({
      data: { email, name }
    });

    res.status(201).json(user);
  } catch (error) {
    res.status(500).json({ error: 'Internal server error' });
  }
});

app.get('/api/users/:id', async (req, res) => {
  try {
    const { id } = req.params;
    
    const user = await prisma.user.findUnique({
      where: { id: parseInt(id) }
    });

    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }

    res.json(user);
  } catch (error) {
    res.status(500).json({ error: 'Internal server error' });
  }
});

export default app;
```

```typescript
// src/app.test.ts
import request from 'supertest';
import app from './app';
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

beforeAll(async () => {
  await prisma.$connect();
});

afterAll(async () => {
  await prisma.user.deleteMany();
  await prisma.$disconnect();
});

beforeEach(async () => {
  await prisma.user.deleteMany();
});

describe('User API', () => {
  describe('POST /api/users', () => {
    it('should create a new user', async () => {
      const userData = {
        email: 'test@example.com',
        name: 'Test User'
      };

      const response = await request(app)
        .post('/api/users')
        .send(userData)
        .expect(201);

      expect(response.body).toMatchObject(userData);
      expect(response.body.id).toBeDefined();
    });

    it('should return 400 for missing email', async () => {
      await request(app)
        .post('/api/users')
        .send({ name: 'Test' })
        .expect(400);
    });
  });

  describe('GET /api/users/:id', () => {
    it('should get user by id', async () => {
      const user = await prisma.user.create({
        data: {
          email: 'test@example.com',
          name: 'Test User'
        }
      });

      const response = await request(app)
        .get(`/api/users/${user.id}`)
        .expect(200);

      expect(response.body).toMatchObject({
        id: user.id,
        email: user.email,
        name: user.name
      });
    });

    it('should return 404 for non-existent user', async () => {
      await request(app)
        .get('/api/users/999')
        .expect(404);
    });
  });
});
```

### Projekt 2: Docker + Tests

```dockerfile
# Dockerfile
FROM node:18-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build

# Test Stage
FROM builder AS test
RUN npm test

# Production Stage
FROM node:18-alpine AS production

WORKDIR /app

COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./

USER node

CMD ["node", "dist/index.js"]
```

```yaml
# docker-compose.test.yml
version: '3.8'

services:
  test-db:
    image: postgres:16
    environment:
      POSTGRES_USER: test
      POSTGRES_PASSWORD: test
      POSTGRES_DB: testdb
    ports:
      - "5433:5432"

  test-app:
    build:
      context: .
      target: test
    environment:
      DATABASE_URL: postgresql://test:test@test-db:5432/testdb
    depends_on:
      - test-db
    command: npm test
```

```bash
# Tests in Docker ausführen
docker-compose -f docker-compose.test.yml up --abort-on-container-exit
```

---

## Zusammenfassung

Du hast jetzt gelernt:

### Docker
✅ **Grundlagen**: Images, Container, Volumes, Networks
✅ **Dockerfile**: Images erstellen und optimieren
✅ **Docker Compose**: Multi-Container-Apps
✅ **Best Practices**: Sicherheit, Performance, Multi-Stage
✅ **Production**: Optimierte Builds

### Testing
✅ **Unit Tests**: Jest, Matchers, Mocking
✅ **Integration Tests**: API Testing, Database Testing
✅ **E2E Tests**: Playwright, User Flows
✅ **TDD**: Red-Green-Refactor Zyklus
✅ **Best Practices**: AAA Pattern, Unabhängigkeit
✅ **Coverage**: Code-Abdeckung messen

### Hilfreiche Ressourcen

**Docker:**
- [Docker Documentation](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [Best Practices](https://docs.docker.com/develop/dev-best-practices/)

**Testing:**
- [Jest Documentation](https://jestjs.io/)
- [Playwright Documentation](https://playwright.dev/)
- [Testing Library](https://testing-library.com/)

Viel Erfolg mit Docker und Testing! 🚀
