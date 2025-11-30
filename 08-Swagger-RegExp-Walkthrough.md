# Swagger & Reguläre Ausdrücke (RegExp) - Kompletter Walkthrough

## Inhaltsverzeichnis

### Teil 1: Swagger / OpenAPI
1. [Einführung in API-Dokumentation](#einführung-in-api-dokumentation)
2. [OpenAPI Specification](#openapi-specification)
3. [Swagger Tools](#swagger-tools)
4. [Swagger UI Setup](#swagger-ui-setup)
5. [OpenAPI-Dokument erstellen](#openapi-dokument-erstellen)
6. [JSDoc Annotations](#jsdoc-annotations)
7. [Swagger mit Express](#swagger-mit-express)
8. [Request/Response Schemas](#requestresponse-schemas)
9. [Authentication in Swagger](#authentication-in-swagger)
10. [Best Practices](#best-practices-swagger)

### Teil 2: Reguläre Ausdrücke
11. [RegExp Grundlagen](#regexp-grundlagen)
12. [Zeichenklassen & Quantifiers](#zeichenklassen--quantifiers)
13. [Gruppen & Capturing](#gruppen--capturing)
14. [Lookahead & Lookbehind](#lookahead--lookbehind)
15. [Praktische Regex-Patterns](#praktische-regex-patterns)
16. [RegExp in JavaScript/TypeScript](#regexp-in-javascripttypescript)
17. [Validation mit RegExp](#validation-mit-regexp)
18. [RegExp Performance](#regexp-performance)
19. [Debugging & Testing](#debugging--testing)
20. [Praktische Projekte](#praktische-projekte)

---

# Teil 1: Swagger / OpenAPI

## Einführung in API-Dokumentation

### Warum API-Dokumentation wichtig ist

**Ohne Dokumentation:**
```typescript
// Entwickler muss im Code suchen
app.post('/api/users', createUser); // Was für Daten? Welches Format?
```

**Mit Dokumentation:**
- 📖 Klare API-Struktur
- 🔧 Interaktive Tests
- 🚀 Schnellere Integration
- ✅ Validierung
- 📝 Standards

### Swagger vs OpenAPI

**OpenAPI** = Specification (Standard)
**Swagger** = Tools für OpenAPI

```
OpenAPI Specification
  ↓
Swagger UI      → Visualisierung
Swagger Editor  → Editor
Swagger Codegen → Code-Generierung
```

---

## OpenAPI Specification

### OpenAPI-Struktur

```yaml
openapi: 3.0.0
info:
  title: My API
  version: 1.0.0
  description: API Documentation

servers:
  - url: http://localhost:3000/api
    description: Development server

paths:
  /users:
    get:
      summary: Get all users
      responses:
        '200':
          description: Success

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
        name:
          type: string
```

### OpenAPI 3.0 Hauptkomponenten

```yaml
openapi: 3.0.0           # Version
info: {...}              # API Info
servers: [...]           # Server URLs
paths: {...}             # Endpoints
components:              # Wiederverwendbare Komponenten
  schemas: {...}         # Data Models
  parameters: {...}      # Query/Path/Header Parameters
  responses: {...}       # Responses
  securitySchemes: {...} # Auth Schemes
security: [...]          # Global Security
tags: [...]              # Gruppierung
```

---

## Swagger Tools

### Swagger UI

Interaktive API-Dokumentation:
- Alle Endpoints visualisiert
- "Try it out"-Funktion
- Request/Response Beispiele
- Automatische Validierung

### Swagger Editor

Online Editor: https://editor.swagger.io/

Features:
- Live Preview
- Validierung
- Code-Generierung
- Import/Export

### Swagger Codegen

Generiert Code aus OpenAPI:
```bash
# Client-Code generieren
swagger-codegen generate -i openapi.yaml -l typescript-fetch -o ./client

# Server-Code generieren
swagger-codegen generate -i openapi.yaml -l nodejs-server -o ./server
```

---

## Swagger UI Setup

### Installation

```bash
npm install swagger-ui-express swagger-jsdoc
npm install --save-dev @types/swagger-ui-express @types/swagger-jsdoc
```

### Basis-Setup

```typescript
// src/swagger.ts
import swaggerJsdoc from 'swagger-jsdoc';
import swaggerUi from 'swagger-ui-express';
import { Express } from 'express';

const options: swaggerJsdoc.Options = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'My API Documentation',
      version: '1.0.0',
      description: 'A comprehensive API for my application',
      contact: {
        name: 'API Support',
        email: 'support@example.com',
        url: 'https://example.com/support'
      },
      license: {
        name: 'MIT',
        url: 'https://opensource.org/licenses/MIT'
      }
    },
    servers: [
      {
        url: 'http://localhost:3000',
        description: 'Development server'
      },
      {
        url: 'https://api.example.com',
        description: 'Production server'
      }
    ]
  },
  apis: ['./src/routes/*.ts', './src/controllers/*.ts']
};

const specs = swaggerJsdoc(options);

export function setupSwagger(app: Express) {
  // Swagger UI
  app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(specs, {
    explorer: true,
    customCss: '.swagger-ui .topbar { display: none }',
    customSiteTitle: 'My API Docs'
  }));

  // JSON Spec
  app.get('/api-docs.json', (req, res) => {
    res.setHeader('Content-Type', 'application/json');
    res.send(specs);
  });
}

// src/index.ts
import express from 'express';
import { setupSwagger } from './swagger';

const app = express();

app.use(express.json());

// Routes
import userRoutes from './routes/users';
app.use('/api/users', userRoutes);

// Swagger Setup
setupSwagger(app);

app.listen(3000, () => {
  console.log('Server: http://localhost:3000');
  console.log('Docs:   http://localhost:3000/api-docs');
});
```

---

## OpenAPI-Dokument erstellen

### Manuelle OpenAPI-Datei

```yaml
# openapi.yaml
openapi: 3.0.0

info:
  title: User Management API
  version: 1.0.0
  description: API for managing users

servers:
  - url: http://localhost:3000/api
    description: Development

paths:
  /users:
    get:
      summary: Get all users
      tags:
        - Users
      parameters:
        - in: query
          name: page
          schema:
            type: integer
            default: 1
          description: Page number
        - in: query
          name: limit
          schema:
            type: integer
            default: 10
          description: Items per page
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                type: object
                properties:
                  success:
                    type: boolean
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/User'
                  pagination:
                    $ref: '#/components/schemas/Pagination'
    
    post:
      summary: Create a new user
      tags:
        - Users
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateUserDto'
      responses:
        '201':
          description: User created
          content:
            application/json:
              schema:
                type: object
                properties:
                  success:
                    type: boolean
                  data:
                    $ref: '#/components/schemas/User'
        '400':
          description: Bad request
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'

  /users/{id}:
    get:
      summary: Get user by ID
      tags:
        - Users
      parameters:
        - in: path
          name: id
          required: true
          schema:
            type: integer
          description: User ID
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                type: object
                properties:
                  success:
                    type: boolean
                  data:
                    $ref: '#/components/schemas/User'
        '404':
          description: User not found
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/Error'
    
    put:
      summary: Update user
      tags:
        - Users
      parameters:
        - in: path
          name: id
          required: true
          schema:
            type: integer
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/UpdateUserDto'
      responses:
        '200':
          description: User updated
        '404':
          description: User not found
    
    delete:
      summary: Delete user
      tags:
        - Users
      parameters:
        - in: path
          name: id
          required: true
          schema:
            type: integer
      responses:
        '204':
          description: User deleted
        '404':
          description: User not found

components:
  schemas:
    User:
      type: object
      required:
        - id
        - name
        - email
      properties:
        id:
          type: integer
          example: 1
        name:
          type: string
          example: Max Mustermann
        email:
          type: string
          format: email
          example: max@example.com
        age:
          type: integer
          minimum: 18
          example: 25
        role:
          type: string
          enum: [user, admin, moderator]
          default: user
        createdAt:
          type: string
          format: date-time
          example: 2024-01-15T10:30:00Z

    CreateUserDto:
      type: object
      required:
        - name
        - email
      properties:
        name:
          type: string
          minLength: 2
          maxLength: 100
        email:
          type: string
          format: email
        age:
          type: integer
          minimum: 18
        role:
          type: string
          enum: [user, admin, moderator]

    UpdateUserDto:
      type: object
      properties:
        name:
          type: string
          minLength: 2
          maxLength: 100
        email:
          type: string
          format: email
        age:
          type: integer
          minimum: 18

    Pagination:
      type: object
      properties:
        page:
          type: integer
        limit:
          type: integer
        total:
          type: integer
        totalPages:
          type: integer

    Error:
      type: object
      properties:
        success:
          type: boolean
          example: false
        error:
          type: object
          properties:
            message:
              type: string
            statusCode:
              type: integer

  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

security:
  - bearerAuth: []
```

---

## JSDoc Annotations

### Basis-Annotations

```typescript
/**
 * @swagger
 * /api/users:
 *   get:
 *     summary: Get all users
 *     tags: [Users]
 *     responses:
 *       200:
 *         description: Success
 */
router.get('/', getUsers);
```

### Schema-Definition

```typescript
/**
 * @swagger
 * components:
 *   schemas:
 *     User:
 *       type: object
 *       required:
 *         - name
 *         - email
 *       properties:
 *         id:
 *           type: integer
 *           description: User ID
 *         name:
 *           type: string
 *           description: User's full name
 *         email:
 *           type: string
 *           format: email
 *           description: User's email address
 *         createdAt:
 *           type: string
 *           format: date-time
 *       example:
 *         id: 1
 *         name: Max Mustermann
 *         email: max@example.com
 *         createdAt: 2024-01-15T10:30:00Z
 */
```

### GET mit Query-Parameters

```typescript
/**
 * @swagger
 * /api/users:
 *   get:
 *     summary: Get all users with pagination
 *     tags: [Users]
 *     parameters:
 *       - in: query
 *         name: page
 *         schema:
 *           type: integer
 *           minimum: 1
 *           default: 1
 *         description: Page number
 *       - in: query
 *         name: limit
 *         schema:
 *           type: integer
 *           minimum: 1
 *           maximum: 100
 *           default: 10
 *         description: Number of items per page
 *       - in: query
 *         name: search
 *         schema:
 *           type: string
 *         description: Search query
 *       - in: query
 *         name: sort
 *         schema:
 *           type: string
 *           enum: [name, email, createdAt, -name, -email, -createdAt]
 *         description: Sort field (prefix with - for descending)
 *     responses:
 *       200:
 *         description: Success
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 success:
 *                   type: boolean
 *                 data:
 *                   type: array
 *                   items:
 *                     $ref: '#/components/schemas/User'
 *                 pagination:
 *                   type: object
 *                   properties:
 *                     page:
 *                       type: integer
 *                     limit:
 *                       type: integer
 *                     total:
 *                       type: integer
 */
router.get('/', getUsers);
```

### POST mit Request Body

```typescript
/**
 * @swagger
 * /api/users:
 *   post:
 *     summary: Create a new user
 *     tags: [Users]
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             required:
 *               - name
 *               - email
 *             properties:
 *               name:
 *                 type: string
 *                 minLength: 2
 *                 maxLength: 100
 *               email:
 *                 type: string
 *                 format: email
 *               age:
 *                 type: integer
 *                 minimum: 18
 *           example:
 *             name: Max Mustermann
 *             email: max@example.com
 *             age: 25
 *     responses:
 *       201:
 *         description: User created successfully
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 success:
 *                   type: boolean
 *                 data:
 *                   $ref: '#/components/schemas/User'
 *       400:
 *         description: Bad request
 *       422:
 *         description: Validation error
 */
router.post('/', createUser);
```

### Path Parameters

```typescript
/**
 * @swagger
 * /api/users/{id}:
 *   get:
 *     summary: Get user by ID
 *     tags: [Users]
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: integer
 *         description: User ID
 *     responses:
 *       200:
 *         description: Success
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/User'
 *       404:
 *         description: User not found
 */
router.get('/:id', getUserById);
```

---

## Swagger mit Express

### Komplettes Beispiel

```typescript
// src/swagger.ts
import swaggerJsdoc from 'swagger-jsdoc';

const options: swaggerJsdoc.Options = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'User Management API',
      version: '1.0.0',
      description: 'Complete API for user management with authentication'
    },
    servers: [
      {
        url: 'http://localhost:3000',
        description: 'Development server'
      }
    ],
    components: {
      securitySchemes: {
        bearerAuth: {
          type: 'http',
          scheme: 'bearer',
          bearerFormat: 'JWT'
        }
      }
    },
    security: [
      {
        bearerAuth: []
      }
    ]
  },
  apis: ['./src/routes/*.ts']
};

export const specs = swaggerJsdoc(options);

// src/routes/users.ts
import express from 'express';

const router = express.Router();

/**
 * @swagger
 * tags:
 *   name: Users
 *   description: User management endpoints
 */

/**
 * @swagger
 * /api/users:
 *   get:
 *     summary: Get all users
 *     tags: [Users]
 *     security:
 *       - bearerAuth: []
 *     parameters:
 *       - in: query
 *         name: page
 *         schema:
 *           type: integer
 *           default: 1
 *       - in: query
 *         name: limit
 *         schema:
 *           type: integer
 *           default: 10
 *     responses:
 *       200:
 *         description: List of users
 *       401:
 *         description: Unauthorized
 */
router.get('/', authenticate, getUsers);

/**
 * @swagger
 * /api/users:
 *   post:
 *     summary: Create a new user
 *     tags: [Users]
 *     security:
 *       - bearerAuth: []
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             required:
 *               - name
 *               - email
 *             properties:
 *               name:
 *                 type: string
 *               email:
 *                 type: string
 *                 format: email
 *     responses:
 *       201:
 *         description: User created
 *       400:
 *         description: Bad request
 */
router.post('/', authenticate, createUser);

export default router;

// src/index.ts
import express from 'express';
import swaggerUi from 'swagger-ui-express';
import { specs } from './swagger';
import userRoutes from './routes/users';

const app = express();

app.use(express.json());

// API Routes
app.use('/api/users', userRoutes);

// Swagger Documentation
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(specs));

app.listen(3000, () => {
  console.log('Server: http://localhost:3000');
  console.log('Docs:   http://localhost:3000/api-docs');
});
```

---

## Request/Response Schemas

### Komplexe Schemas

```typescript
/**
 * @swagger
 * components:
 *   schemas:
 *     Address:
 *       type: object
 *       properties:
 *         street:
 *           type: string
 *         city:
 *           type: string
 *         zipCode:
 *           type: string
 *         country:
 *           type: string
 *
 *     Profile:
 *       type: object
 *       properties:
 *         bio:
 *           type: string
 *         avatar:
 *           type: string
 *           format: uri
 *         website:
 *           type: string
 *           format: uri
 *
 *     User:
 *       type: object
 *       required:
 *         - name
 *         - email
 *       properties:
 *         id:
 *           type: integer
 *         name:
 *           type: string
 *         email:
 *           type: string
 *           format: email
 *         address:
 *           $ref: '#/components/schemas/Address'
 *         profile:
 *           $ref: '#/components/schemas/Profile'
 *         tags:
 *           type: array
 *           items:
 *             type: string
 *         metadata:
 *           type: object
 *           additionalProperties:
 *             type: string
 */
```

### Polymorphism (oneOf, anyOf, allOf)

```typescript
/**
 * @swagger
 * components:
 *   schemas:
 *     Cat:
 *       type: object
 *       properties:
 *         type:
 *           type: string
 *           enum: [cat]
 *         meow:
 *           type: boolean
 *
 *     Dog:
 *       type: object
 *       properties:
 *         type:
 *           type: string
 *           enum: [dog]
 *         bark:
 *           type: boolean
 *
 *     Pet:
 *       oneOf:
 *         - $ref: '#/components/schemas/Cat'
 *         - $ref: '#/components/schemas/Dog'
 *       discriminator:
 *         propertyName: type
 */
```

---

## Authentication in Swagger

### Bearer Token

```typescript
const options: swaggerJsdoc.Options = {
  definition: {
    openapi: '3.0.0',
    components: {
      securitySchemes: {
        bearerAuth: {
          type: 'http',
          scheme: 'bearer',
          bearerFormat: 'JWT',
          description: 'Enter your JWT token'
        }
      }
    }
  }
};

/**
 * @swagger
 * /api/profile:
 *   get:
 *     summary: Get user profile
 *     security:
 *       - bearerAuth: []
 *     responses:
 *       200:
 *         description: Success
 *       401:
 *         description: Unauthorized
 */
```

### API Key

```typescript
components: {
  securitySchemes: {
    apiKey: {
      type: 'apiKey',
      in: 'header',
      name: 'X-API-Key'
    }
  }
}
```

### OAuth 2.0

```typescript
components: {
  securitySchemes: {
    oauth2: {
      type: 'oauth2',
      flows: {
        authorizationCode: {
          authorizationUrl: 'https://example.com/oauth/authorize',
          tokenUrl: 'https://example.com/oauth/token',
          scopes: {
            'read:users': 'Read user information',
            'write:users': 'Modify user information'
          }
        }
      }
    }
  }
}
```

---

## Best Practices (Swagger)

### 1. Versionierung

```typescript
info: {
  title: 'My API',
  version: '1.0.0',  // Semantic Versioning
}

servers: [
  { url: 'https://api.example.com/v1' },
  { url: 'https://api.example.com/v2' }
]
```

### 2. Klare Beschreibungen

```typescript
/**
 * @swagger
 * /api/users/{id}:
 *   delete:
 *     summary: Delete a user
 *     description: |
 *       Permanently deletes a user and all associated data.
 *       This action cannot be undone.
 *       
 *       **Warning:** Deleting a user will also delete:
 *       - All user posts
 *       - All user comments
 *       - User profile information
 *     tags: [Users]
 */
```

### 3. Beispiele hinzufügen

```typescript
/**
 * @swagger
 * components:
 *   schemas:
 *     User:
 *       type: object
 *       properties:
 *         name:
 *           type: string
 *         email:
 *           type: string
 *       example:
 *         name: Max Mustermann
 *         email: max@example.com
 */
```

### 4. Error Responses dokumentieren

```typescript
/**
 * @swagger
 * /api/users:
 *   post:
 *     responses:
 *       201:
 *         description: User created
 *       400:
 *         description: Bad request - Invalid input
 *       409:
 *         description: Conflict - Email already exists
 *       422:
 *         description: Validation error
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 error:
 *                   type: string
 *                 details:
 *                   type: array
 *                   items:
 *                     type: object
 *                     properties:
 *                       field:
 *                         type: string
 *                       message:
 *                         type: string
 */
```

---

# Teil 2: Reguläre Ausdrücke

## RegExp Grundlagen

### Was sind Reguläre Ausdrücke?

**Regular Expressions (RegEx)** sind Muster zum Suchen und Bearbeiten von Text.

### Syntax

```javascript
// Literal Notation
const regex = /pattern/flags;

// Constructor
const regex = new RegExp('pattern', 'flags');
```

### Flags

```javascript
/pattern/g   // global - alle Matches
/pattern/i   // case-insensitive
/pattern/m   // multiline
/pattern/s   // dotAll - . matched auch \n
/pattern/u   // unicode
/pattern/y   // sticky - match ab lastIndex
```

### Basis-Methoden

```javascript
// Test (boolean)
/hello/.test('hello world');  // true

// Match (array)
'hello world'.match(/hello/);  // ['hello']

// Replace
'hello world'.replace(/world/, 'universe');  // 'hello universe'

// Split
'a,b,c'.split(/,/);  // ['a', 'b', 'c']

// Search (index)
'hello world'.search(/world/);  // 6
```

---

## Zeichenklassen & Quantifiers

### Zeichenklassen

```javascript
.       // Beliebiges Zeichen (außer \n)
\d      // Digit [0-9]
\D      // Nicht-Digit [^0-9]
\w      // Word-Character [a-zA-Z0-9_]
\W      // Nicht-Word-Character
\s      // Whitespace [ \t\n\r]
\S      // Nicht-Whitespace

// Custom Character Classes
[abc]   // a oder b oder c
[^abc]  // Nicht a, b oder c
[a-z]   // a bis z
[A-Z]   // A bis Z
[0-9]   // 0 bis 9
[a-zA-Z0-9]  // Alphanumerisch
```

### Quantifiers

```javascript
*       // 0 oder mehr
+       // 1 oder mehr
?       // 0 oder 1 (optional)
{n}     // Genau n mal
{n,}    // Mindestens n mal
{n,m}   // n bis m mal

// Beispiele
/a*/    // '', 'a', 'aa', 'aaa'
/a+/    // 'a', 'aa', 'aaa'
/a?/    // '', 'a'
/a{3}/  // 'aaa'
/a{2,4}/  // 'aa', 'aaa', 'aaaa'
```

### Greedy vs Non-Greedy

```javascript
// Greedy (default)
'<div>Hello</div>'.match(/<.*>/);
// ['<div>Hello</div>']

// Non-greedy (lazy)
'<div>Hello</div>'.match(/<.*?>/);
// ['<div>']

// Non-greedy Quantifiers
*?      // 0 oder mehr (lazy)
+?      // 1 oder mehr (lazy)
??      // 0 oder 1 (lazy)
{n,m}?  // n bis m (lazy)
```

---

## Gruppen & Capturing

### Capturing Groups

```javascript
// Einfache Gruppe
const regex = /(\d{4})-(\d{2})-(\d{2})/;
const match = '2024-01-15'.match(regex);
// match[0]: '2024-01-15' (vollständiger Match)
// match[1]: '2024' (Jahr)
// match[2]: '01' (Monat)
// match[3]: '15' (Tag)

// Named Groups
const regex = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
const match = '2024-01-15'.match(regex);
// match.groups.year: '2024'
// match.groups.month: '01'
// match.groups.day: '15'
```

### Non-Capturing Groups

```javascript
// Non-capturing group (?:...)
/(?:https?):\/\/([\w.]+)/
// Captured nur die Domain, nicht das Protokoll

'https://example.com'.match(/(?:https?):\/\/([\w.]+)/);
// ['https://example.com', 'example.com']
```

### Backreferences

```javascript
// Backreference auf Gruppe
/(\w+)\s\1/  // Wiederholtes Wort
'hello hello'.match(/(\w+)\s\1/);  // ['hello hello', 'hello']

// Named Backreference
/(?<word>\w+)\s\k<word>/
'hello hello'.match(/(?<word>\w+)\s\k<word>/);
```

### Alternation

```javascript
// OR-Operator |
/cat|dog/    // 'cat' oder 'dog'
/gr(a|e)y/   // 'gray' oder 'grey'
```

---

## Lookahead & Lookbehind

### Positive Lookahead (?=...)

```javascript
// Match nur wenn gefolgt von...
/\d+(?= Euro)/     // Zahlen gefolgt von " Euro"
'100 Euro'.match(/\d+(?= Euro)/);  // ['100']
'100 Dollar'.match(/\d+(?= Euro)/);  // null
```

### Negative Lookahead (?!...)

```javascript
// Match nur wenn NICHT gefolgt von...
/\d+(?! Euro)/     // Zahlen NICHT gefolgt von " Euro"
'100 Euro'.match(/\d+(?! Euro)/);    // null
'100 Dollar'.match(/\d+(?! Euro)/);  // ['100']
```

### Positive Lookbehind (?<=...)

```javascript
// Match nur wenn vorangegangen von...
/(?<=\$)\d+/       // Zahlen nach $
'$100'.match(/(?<=\$)\d+/);  // ['100']
'€100'.match(/(?<=\$)\d+/);  // null
```

### Negative Lookbehind (?<!...)

```javascript
// Match nur wenn NICHT vorangegangen von...
/(?<!\$)\d+/       // Zahlen NICHT nach $
'$100'.match(/(?<!\$)\d+/);  // null (eigentlich ['00'])
'€100'.match(/(?<!\$)\d+/);  // ['100']
```

### Kombiniert

```javascript
// Password: mind. 8 Zeichen, mind. 1 Zahl, mind. 1 Großbuchstabe
/^(?=.*[A-Z])(?=.*\d).{8,}$/

// Erklärung:
// ^              Anfang
// (?=.*[A-Z])    Lookahead: enthält Großbuchstaben
// (?=.*\d)       Lookahead: enthält Zahl
// .{8,}          Mindestens 8 Zeichen
// $              Ende
```

---

## Praktische Regex-Patterns

### Email-Validierung

```javascript
// Basis
/^[\w.-]+@[\w.-]+\.\w+$/

// Besser
/^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/

// RFC 5322 (vereinfacht)
/^[a-zA-Z0-9.!#$%&'*+/=?^_`{|}~-]+@[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?(?:\.[a-zA-Z0-9](?:[a-zA-Z0-9-]{0,61}[a-zA-Z0-9])?)*$/
```

### URL-Validierung

```javascript
// Basis
/^https?:\/\/[\w.-]+\.\w{2,}(\/.*)?$/

// Mit optionalem Port
/^https?:\/\/[\w.-]+(:\d+)?(\/.*)?$/

// Komplett
/^(https?:\/\/)?([\da-z\.-]+)\.([a-z\.]{2,6})([\/\w \.-]*)*\/?$/
```

### Telefonnummer

```javascript
// Deutsche Telefonnummer
/^(\+49|0)[1-9]\d{1,14}$/

// International
/^\+?[1-9]\d{1,14}$/

// Mit Formatierung
/^(\+\d{1,3}[- ]?)?\(?\d{3}\)?[- ]?\d{3}[- ]?\d{4}$/
// Matches: +1-123-456-7890, (123) 456-7890, 123-456-7890
```

### Postleitzahl

```javascript
// Deutschland
/^\d{5}$/

// USA
/^\d{5}(-\d{4})?$/

// UK
/^[A-Z]{1,2}\d{1,2}[A-Z]?\s?\d[A-Z]{2}$/
```

### Kreditkarte

```javascript
// Visa
/^4\d{12}(?:\d{3})?$/

// Mastercard
/^5[1-5]\d{14}$/

// American Express
/^3[47]\d{13}$/

// Alle (mit optionalen Leerzeichen/Bindestrichen)
/^[\d\s-]{13,19}$/
```

### IBAN

```javascript
// Deutsche IBAN
/^DE\d{20}$/

// Europäische IBAN
/^[A-Z]{2}\d{2}[A-Z0-9]{1,30}$/
```

### Datum

```javascript
// YYYY-MM-DD
/^\d{4}-\d{2}-\d{2}$/

// DD.MM.YYYY
/^\d{2}\.\d{2}\.\d{4}$/

// MM/DD/YYYY
/^\d{2}\/\d{2}\/\d{4}$/

// Mit Validierung (vereinfacht)
/^(19|20)\d{2}-(0[1-9]|1[0-2])-(0[1-9]|[12]\d|3[01])$/
```

### Zeit

```javascript
// HH:MM (24h)
/^([01]\d|2[0-3]):([0-5]\d)$/

// HH:MM:SS
/^([01]\d|2[0-3]):([0-5]\d):([0-5]\d)$/

// 12h mit AM/PM
/^(0?[1-9]|1[0-2]):([0-5]\d)\s?(AM|PM)$/i
```

### IP-Adresse

```javascript
// IPv4 (einfach)
/^\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}$/

// IPv4 (mit Validierung)
/^(25[0-5]|2[0-4]\d|1\d{2}|[1-9]?\d)(\.(25[0-5]|2[0-4]\d|1\d{2}|[1-9]?\d)){3}$/

// IPv6 (vereinfacht)
/^([0-9a-fA-F]{1,4}:){7}[0-9a-fA-F]{1,4}$/
```

### HTML Tags entfernen

```javascript
/<[^>]*>/g

// Beispiel
'<p>Hello <strong>World</strong></p>'.replace(/<[^>]*>/g, '');
// 'Hello World'
```

### Hex-Color

```javascript
// #RGB oder #RRGGBB
/^#([A-Fa-f0-9]{6}|[A-Fa-f0-9]{3})$/

'#FFF'.test(/^#([A-Fa-f0-9]{6}|[A-Fa-f0-9]{3})$/);  // true
'#FF5733'.test(/^#([A-Fa-f0-9]{6}|[A-Fa-f0-9]{3})$/);  // true
```

---

## RegExp in JavaScript/TypeScript

### String-Methoden

```typescript
// test() - Boolean
const hasNumber = /\d/.test('hello123');  // true

// match() - Array oder null
const matches = 'hello world'.match(/\w+/g);  // ['hello', 'world']

// matchAll() - Iterator
const text = 'test1 test2 test3';
const regex = /test(\d)/g;
for (const match of text.matchAll(regex)) {
  console.log(match[1]);  // '1', '2', '3'
}

// replace() - String
'hello world'.replace(/world/, 'universe');  // 'hello universe'

// replaceAll() - String
'hello hello hello'.replaceAll(/hello/g, 'hi');  // 'hi hi hi'

// search() - Index
'hello world'.search(/world/);  // 6

// split() - Array
'a,b,c'.split(/,/);  // ['a', 'b', 'c']
```

### RegExp-Methoden

```typescript
const regex = /\w+/g;
const text = 'hello world';

// test() - Boolean
regex.test(text);  // true

// exec() - Array oder null
const match = regex.exec(text);
// match[0]: vollständiger Match
// match.index: Position
// match.input: Original-String
```

### Replace mit Funktion

```typescript
// Callback-Funktion
const text = 'hello world';
const result = text.replace(/\w+/g, (match) => {
  return match.toUpperCase();
});
// 'HELLO WORLD'

// Mit Gruppen
const text = '2024-01-15';
const result = text.replace(/(\d{4})-(\d{2})-(\d{2})/, (match, year, month, day) => {
  return `${day}.${month}.${year}`;
});
// '15.01.2024'

// Named Groups
const text = '2024-01-15';
const result = text.replace(
  /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/,
  (match, ...args) => {
    const groups = args[args.length - 1];
    return `${groups.day}.${groups.month}.${groups.year}`;
  }
);
```

### TypeScript Types

```typescript
// RegExp Type
const regex: RegExp = /test/;

// Match Result
const match: RegExpMatchArray | null = 'test'.match(/test/);

// Exec Result
const execResult: RegExpExecArray | null = /test/.exec('test');

// Type Guard
function isValidEmail(email: string): boolean {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return emailRegex.test(email);
}
```

---

## Validation mit RegExp

### Validation-Funktionen

```typescript
// Email
export function isValidEmail(email: string): boolean {
  const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
  return emailRegex.test(email);
}

// Password (mind. 8 Zeichen, 1 Großbuchstabe, 1 Zahl, 1 Sonderzeichen)
export function isValidPassword(password: string): boolean {
  const passwordRegex = /^(?=.*[A-Z])(?=.*[a-z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/;
  return passwordRegex.test(password);
}

// Username (3-20 Zeichen, nur alphanumerisch und _)
export function isValidUsername(username: string): boolean {
  const usernameRegex = /^[a-zA-Z0-9_]{3,20}$/;
  return usernameRegex.test(username);
}

// URL
export function isValidUrl(url: string): boolean {
  const urlRegex = /^https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)$/;
  return urlRegex.test(url);
}

// Telefonnummer (international)
export function isValidPhone(phone: string): boolean {
  const phoneRegex = /^\+?[1-9]\d{1,14}$/;
  return phoneRegex.test(phone.replace(/[\s-]/g, ''));
}

// Kreditkarte
export function isValidCreditCard(card: string): boolean {
  const cardRegex = /^[\d\s-]{13,19}$/;
  return cardRegex.test(card);
}

// IBAN
export function isValidIBAN(iban: string): boolean {
  const ibanRegex = /^[A-Z]{2}\d{2}[A-Z0-9]{1,30}$/;
  return ibanRegex.test(iban.replace(/\s/g, ''));
}
```

### Validation mit Zod

```typescript
import { z } from 'zod';

const emailSchema = z.string().regex(
  /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/,
  'Invalid email address'
);

const passwordSchema = z.string().regex(
  /^(?=.*[A-Z])(?=.*[a-z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/,
  'Password must be at least 8 characters and contain uppercase, lowercase, number and special character'
);

const userSchema = z.object({
  username: z.string().regex(/^[a-zA-Z0-9_]{3,20}$/, 'Invalid username'),
  email: emailSchema,
  password: passwordSchema,
  phone: z.string().regex(/^\+?[1-9]\d{1,14}$/, 'Invalid phone number').optional()
});

// Verwendung
try {
  userSchema.parse({
    username: 'max123',
    email: 'max@example.com',
    password: 'Secure123!'
  });
} catch (error) {
  console.error(error);
}
```

---

## RegExp Performance

### Performance-Tipps

```javascript
// ❌ Schlecht - Catastrophic Backtracking
/(a+)+b/
/(a*)*b/
/(a|a)*b/

// ✅ Gut - Effizient
/a+b/

// ❌ Schlecht - Unnecessarily complex
/^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/

// ✅ Besser - Optimiert
/^[\w.+-]+@[\w.-]+\.\w{2,}$/
```

### Anchors verwenden

```javascript
// ❌ Ohne Anchors - scannt ganzen String
/test/

// ✅ Mit Anchors - stoppt bei Mismatch
/^test$/
```

### Non-Capturing Groups

```javascript
// ❌ Capturing (langsamer)
/(https?):\/\/(www\.)?([a-z]+)\.com/

// ✅ Non-Capturing (schneller)
/(?:https?):\/\/(?:www\.)?([a-z]+)\.com/
```

### Regex-Objekt wiederverwenden

```javascript
// ❌ Schlecht - Regex wird jedes Mal neu erstellt
function validate(text: string) {
  return /^test$/.test(text);
}

// ✅ Gut - Regex wird wiederverwendet
const VALIDATION_REGEX = /^test$/;
function validate(text: string) {
  return VALIDATION_REGEX.test(text);
}
```

---

## Debugging & Testing

### Regex Testen

```javascript
// Online Tools:
// - regex101.com
// - regexr.com
// - regexpal.com

// Node.js / Browser Console
const regex = /\d+/g;
const text = 'I have 2 apples and 3 oranges';
console.log(text.match(regex));  // ['2', '3']
```

### Visualisierung

```javascript
// regex101.com zeigt:
// - Matches
// - Groups
// - Erklärung
// - Performance
```

### Testing mit Jest

```typescript
// regex.test.ts
describe('Email Validation', () => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

  it('should validate correct emails', () => {
    expect(emailRegex.test('test@example.com')).toBe(true);
    expect(emailRegex.test('user.name@example.co.uk')).toBe(true);
  });

  it('should reject invalid emails', () => {
    expect(emailRegex.test('invalid')).toBe(false);
    expect(emailRegex.test('@example.com')).toBe(false);
    expect(emailRegex.test('test@')).toBe(false);
  });
});

describe('Password Validation', () => {
  const passwordRegex = /^(?=.*[A-Z])(?=.*\d).{8,}$/;

  it('should validate strong passwords', () => {
    expect(passwordRegex.test('Password123')).toBe(true);
    expect(passwordRegex.test('Secure2024!')).toBe(true);
  });

  it('should reject weak passwords', () => {
    expect(passwordRegex.test('short')).toBe(false);
    expect(passwordRegex.test('nouppercaseornumber')).toBe(false);
    expect(passwordRegex.test('NoNumber')).toBe(false);
  });
});
```

---

## Praktische Projekte

### Projekt 1: Form Validator

```typescript
// src/validators/formValidator.ts
export interface ValidationResult {
  valid: boolean;
  errors: { field: string; message: string }[];
}

export class FormValidator {
  private static readonly patterns = {
    email: /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/,
    password: /^(?=.*[A-Z])(?=.*[a-z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/,
    username: /^[a-zA-Z0-9_]{3,20}$/,
    phone: /^\+?[1-9]\d{1,14}$/,
    url: /^https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b/,
    zipCode: /^\d{5}$/,
    creditCard: /^\d{13,19}$/
  };

  static validateEmail(email: string): boolean {
    return this.patterns.email.test(email);
  }

  static validatePassword(password: string): boolean {
    return this.patterns.password.test(password);
  }

  static validateUsername(username: string): boolean {
    return this.patterns.username.test(username);
  }

  static validatePhone(phone: string): boolean {
    return this.patterns.phone.test(phone.replace(/[\s-]/g, ''));
  }

  static validateForm(data: Record<string, any>): ValidationResult {
    const errors: { field: string; message: string }[] = [];

    if (data.email && !this.validateEmail(data.email)) {
      errors.push({ field: 'email', message: 'Invalid email address' });
    }

    if (data.password && !this.validatePassword(data.password)) {
      errors.push({
        field: 'password',
        message: 'Password must be at least 8 characters with uppercase, lowercase, number and special character'
      });
    }

    if (data.username && !this.validateUsername(data.username)) {
      errors.push({
        field: 'username',
        message: 'Username must be 3-20 characters, alphanumeric and underscores only'
      });
    }

    if (data.phone && !this.validatePhone(data.phone)) {
      errors.push({ field: 'phone', message: 'Invalid phone number' });
    }

    return {
      valid: errors.length === 0,
      errors
    };
  }
}

// Verwendung
const result = FormValidator.validateForm({
  email: 'test@example.com',
  password: 'Secure123!',
  username: 'max_123',
  phone: '+4915112345678'
});

if (!result.valid) {
  console.error('Validation errors:', result.errors);
}
```

### Projekt 2: Text Parser

```typescript
// src/parsers/textParser.ts
export class TextParser {
  // URLs extrahieren
  static extractUrls(text: string): string[] {
    const urlRegex = /https?:\/\/(www\.)?[-a-zA-Z0-9@:%._\+~#=]{1,256}\.[a-zA-Z0-9()]{1,6}\b([-a-zA-Z0-9()@:%_\+.~#?&//=]*)/g;
    return text.match(urlRegex) || [];
  }

  // Email-Adressen extrahieren
  static extractEmails(text: string): string[] {
    const emailRegex = /[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}/g;
    return text.match(emailRegex) || [];
  }

  // Hashtags extrahieren
  static extractHashtags(text: string): string[] {
    const hashtagRegex = /#[\w]+/g;
    return text.match(hashtagRegex) || [];
  }

  // Mentions extrahieren
  static extractMentions(text: string): string[] {
    const mentionRegex = /@[\w]+/g;
    return text.match(mentionRegex) || [];
  }

  // Telefonnummern extrahieren
  static extractPhones(text: string): string[] {
    const phoneRegex = /\+?\d{1,3}[-.\s]?\(?\d{3}\)?[-.\s]?\d{3}[-.\s]?\d{4}/g;
    return text.match(phoneRegex) || [];
  }

  // Datum extrahieren (DD.MM.YYYY oder YYYY-MM-DD)
  static extractDates(text: string): string[] {
    const dateRegex = /(\d{2}\.\d{2}\.\d{4}|\d{4}-\d{2}-\d{2})/g;
    return text.match(dateRegex) || [];
  }

  // HTML Tags entfernen
  static stripHtml(text: string): string {
    return text.replace(/<[^>]*>/g, '');
  }

  // Markdown Links zu HTML
  static markdownLinksToHtml(text: string): string {
    return text.replace(/\[([^\]]+)\]\(([^\)]+)\)/g, '<a href="$2">$1</a>');
  }

  // Text highlighten
  static highlight(text: string, query: string): string {
    const regex = new RegExp(`(${query})`, 'gi');
    return text.replace(regex, '<mark>$1</mark>');
  }

  // Wörter zählen
  static countWords(text: string): number {
    const words = text.match(/\b\w+\b/g);
    return words ? words.length : 0;
  }

  // Sätze extrahieren
  static extractSentences(text: string): string[] {
    return text.match(/[^.!?]+[.!?]+/g) || [];
  }
}

// Verwendung
const text = `
  Check out https://example.com and contact me at test@example.com
  #javascript #typescript @johnDoe
  Call me at +49-151-12345678
  Meeting on 15.01.2024 or 2024-01-15
`;

console.log('URLs:', TextParser.extractUrls(text));
console.log('Emails:', TextParser.extractEmails(text));
console.log('Hashtags:', TextParser.extractHashtags(text));
console.log('Mentions:', TextParser.extractMentions(text));
console.log('Phones:', TextParser.extractPhones(text));
console.log('Dates:', TextParser.extractDates(text));
```

---

## Zusammenfassung

### Swagger / OpenAPI

✅ **API-Dokumentation**: Klare, interaktive Dokumentation
✅ **OpenAPI Spec**: Standard für API-Beschreibungen
✅ **Swagger UI**: Visualisierung und Testing
✅ **JSDoc Annotations**: Code-basierte Dokumentation
✅ **Schemas**: Request/Response-Validierung
✅ **Authentication**: Bearer Token, API Keys, OAuth
✅ **Best Practices**: Versionierung, Beispiele, Error Responses

### Reguläre Ausdrücke

✅ **Grundlagen**: Patterns, Flags, Methoden
✅ **Zeichenklassen**: \d, \w, \s, [abc]
✅ **Quantifiers**: *, +, ?, {n,m}
✅ **Gruppen**: Capturing, Non-Capturing, Named Groups
✅ **Lookahead/Lookbehind**: Assertions
✅ **Praktische Patterns**: Email, URL, Telefon, Datum
✅ **Validation**: FormValidator, Zod Integration
✅ **Performance**: Optimierungen, Backtracking vermeiden

### Hilfreiche Ressourcen

**Swagger:**
- [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Swagger Documentation](https://swagger.io/docs/)
- [Swagger Editor](https://editor.swagger.io/)

**RegExp:**
- [regex101.com](https://regex101.com/) - Testing & Debugging
- [regexr.com](https://regexr.com/) - Learn & Test
- [MDN RegExp](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions)

Viel Erfolg mit Swagger und RegExp! 🚀
