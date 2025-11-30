# Web-APIs - Kompletter Walkthrough

## Inhaltsverzeichnis
1. [Einführung](#einführung)
2. [REST API Grundlagen](#rest-api-grundlagen)
3. [HTTP-Methoden](#http-methoden)
4. [Status Codes](#status-codes)
5. [API Design Best Practices](#api-design-best-practices)
6. [Authentication & Authorization](#authentication--authorization)
7. [Express.js API erstellen](#expressjs-api-erstellen)
8. [Middleware](#middleware)
9. [Error Handling](#error-handling)
10. [Validation](#validation)
11. [CORS](#cors)
12. [Rate Limiting](#rate-limiting)
13. [API Documentation](#api-documentation)
14. [GraphQL Einführung](#graphql-einführung)
15. [Praktische Projekte](#praktische-projekte)

---

## Einführung

### Was sind Web-APIs?

**API (Application Programming Interface)** ist eine Schnittstelle, die es verschiedenen Anwendungen ermöglicht, miteinander zu kommunizieren.

**Web-API** ist eine API, die über HTTP/HTTPS zugänglich ist.

### API-Typen

#### REST (Representational State Transfer)
- Am weitesten verbreitet
- Verwendet HTTP-Methoden
- Ressourcen-orientiert
- Stateless

#### GraphQL
- Flexible Queries
- Keine Over-/Under-fetching
- Single Endpoint
- Strongly Typed

#### SOAP (Simple Object Access Protocol)
- XML-basiert
- Strikte Standards
- Enterprise-fokussiert

#### gRPC
- Binäres Protokoll
- Sehr performant
- Microservices

### Warum APIs wichtig sind

✅ **Separation of Concerns**: Frontend ≠ Backend
✅ **Multiple Clients**: Web, Mobile, Desktop, IoT
✅ **Skalierbarkeit**: Unabhängiges Scaling
✅ **Integration**: Verschiedene Services verbinden
✅ **Wiederverwendbarkeit**: Code-Reuse

---

## REST API Grundlagen

### REST-Prinzipien

1. **Client-Server**: Getrennte Verantwortlichkeiten
2. **Stateless**: Jede Anfrage enthält alle nötigen Informationen
3. **Cacheable**: Responses können gecacht werden
4. **Uniform Interface**: Einheitliche Schnittstelle
5. **Layered System**: Mehrere Schichten möglich
6. **Code on Demand** (optional): Server kann Code senden

### Ressourcen

REST basiert auf **Ressourcen** (Entities), die über URLs identifiziert werden.

```
Ressource: User
URL: /users
Einzelner User: /users/123

Ressource: Post
URL: /posts
Einzelner Post: /posts/456
```

### URL-Struktur

```
Basis-URL: https://api.example.com/v1

Ressourcen-Sammlung:
GET https://api.example.com/v1/users

Einzelne Ressource:
GET https://api.example.com/v1/users/123

Nested Resources:
GET https://api.example.com/v1/users/123/posts
GET https://api.example.com/v1/users/123/posts/456
```

### REST Naming Conventions

```bash
# ✅ Gut - Plural, Kleinbuchstaben, Bindestriche
GET /users
GET /blog-posts
GET /product-categories

# ❌ Schlecht - Singular, CamelCase, Verben
GET /user
GET /blogPosts
GET /getUsers
```

---

## HTTP-Methoden

### GET - Ressource abrufen

```http
GET /api/users
GET /api/users/123
GET /api/users?page=1&limit=10
```

**Eigenschaften:**
- Sicher (keine Änderungen)
- Idempotent (mehrfache Aufrufe = gleiches Ergebnis)
- Cacheable

### POST - Ressource erstellen

```http
POST /api/users
Content-Type: application/json

{
  "name": "Max Mustermann",
  "email": "max@example.com"
}
```

**Eigenschaften:**
- Nicht sicher
- Nicht idempotent
- Erstellt neue Ressource

### PUT - Ressource ersetzen

```http
PUT /api/users/123
Content-Type: application/json

{
  "name": "Max Müller",
  "email": "max.mueller@example.com"
}
```

**Eigenschaften:**
- Nicht sicher
- Idempotent
- Ersetzt komplette Ressource

### PATCH - Ressource teilweise aktualisieren

```http
PATCH /api/users/123
Content-Type: application/json

{
  "name": "Max Müller"
}
```

**Eigenschaften:**
- Nicht sicher
- Idempotent (meist)
- Aktualisiert nur angegebene Felder

### DELETE - Ressource löschen

```http
DELETE /api/users/123
```

**Eigenschaften:**
- Nicht sicher
- Idempotent
- Löscht Ressource

### Zusammenfassung

| Methode | Verwendung | Idempotent | Sicher |
|---------|-----------|------------|--------|
| GET     | Lesen     | ✅         | ✅     |
| POST    | Erstellen | ❌         | ❌     |
| PUT     | Ersetzen  | ✅         | ❌     |
| PATCH   | Update    | ✅         | ❌     |
| DELETE  | Löschen   | ✅         | ❌     |

---

## Status Codes

### 2xx - Erfolg

```
200 OK                - Erfolgreiche GET/PUT/PATCH/DELETE
201 Created           - Erfolgreiche POST (Ressource erstellt)
202 Accepted          - Anfrage akzeptiert, wird verarbeitet
204 No Content        - Erfolgreich, keine Response-Body
```

### 3xx - Umleitung

```
301 Moved Permanently - Ressource permanent verschoben
302 Found             - Ressource temporär verschoben
304 Not Modified      - Ressource nicht geändert (Cache)
```

### 4xx - Client-Fehler

```
400 Bad Request       - Ungültige Anfrage
401 Unauthorized      - Nicht authentifiziert
403 Forbidden         - Nicht autorisiert
404 Not Found         - Ressource nicht gefunden
405 Method Not Allowed - HTTP-Methode nicht erlaubt
409 Conflict          - Konflikt (z.B. Duplikat)
422 Unprocessable Entity - Validierungsfehler
429 Too Many Requests - Rate Limit überschritten
```

### 5xx - Server-Fehler

```
500 Internal Server Error - Allgemeiner Serverfehler
502 Bad Gateway          - Ungültige Gateway-Response
503 Service Unavailable  - Service nicht verfügbar
504 Gateway Timeout      - Gateway Timeout
```

### Beispiel-Responses

```typescript
// 200 OK
{
  "id": 1,
  "name": "Max Mustermann",
  "email": "max@example.com"
}

// 201 Created
{
  "id": 123,
  "name": "Neuer User",
  "email": "neu@example.com",
  "createdAt": "2024-01-15T10:30:00Z"
}

// 400 Bad Request
{
  "error": "Bad Request",
  "message": "Email is required",
  "statusCode": 400
}

// 404 Not Found
{
  "error": "Not Found",
  "message": "User with id 999 not found",
  "statusCode": 404
}

// 422 Unprocessable Entity
{
  "error": "Validation Error",
  "message": "Request validation failed",
  "statusCode": 422,
  "details": [
    {
      "field": "email",
      "message": "Email must be a valid email address"
    },
    {
      "field": "age",
      "message": "Age must be at least 18"
    }
  ]
}
```

---

## API Design Best Practices

### 1. Versionierung

```bash
# URL-basiert (empfohlen)
https://api.example.com/v1/users
https://api.example.com/v2/users

# Header-basiert
GET /users
Accept: application/vnd.api.v1+json

# Query-Parameter
GET /users?version=1
```

### 2. Pagination

```bash
# Offset-based
GET /users?offset=0&limit=20

# Cursor-based (besser für große Datasets)
GET /users?cursor=abc123&limit=20

# Page-based
GET /users?page=1&limit=20
```

**Response:**

```json
{
  "data": [...],
  "pagination": {
    "total": 1000,
    "page": 1,
    "limit": 20,
    "totalPages": 50,
    "hasNext": true,
    "hasPrev": false
  }
}
```

### 3. Filtering

```bash
GET /users?status=active
GET /users?role=admin&status=active
GET /products?minPrice=10&maxPrice=100
```

### 4. Sorting

```bash
# Aufsteigend
GET /users?sort=name

# Absteigend
GET /users?sort=-createdAt

# Mehrere Felder
GET /users?sort=name,-createdAt
```

### 5. Field Selection (Sparse Fieldsets)

```bash
# Nur bestimmte Felder
GET /users?fields=id,name,email

# Für Nested Resources
GET /users?fields=id,name&fields[posts]=title,createdAt
```

### 6. Include Related Resources

```bash
# User mit Posts
GET /users/123?include=posts

# Mehrere Relations
GET /users/123?include=posts,comments,profile
```

### 7. Konsistente Response-Struktur

```json
{
  "success": true,
  "data": {
    "id": 1,
    "name": "Max"
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z"
  }
}
```

### 8. Error Response Format

```json
{
  "success": false,
  "error": {
    "code": "USER_NOT_FOUND",
    "message": "User with id 999 not found",
    "statusCode": 404,
    "details": []
  },
  "meta": {
    "timestamp": "2024-01-15T10:30:00Z",
    "requestId": "abc-123"
  }
}
```

---

## Authentication & Authorization

### Authentication (Wer bist du?)

#### 1. API Keys

```http
GET /api/users
X-API-Key: your-api-key-here
```

**Vorteile:** Einfach
**Nachteile:** Nicht sicher für User-spezifische Daten

#### 2. Basic Auth

```http
GET /api/users
Authorization: Basic base64(username:password)
```

**Vorteile:** Einfach
**Nachteile:** Credentials in jedem Request

#### 3. Bearer Token (JWT)

```http
GET /api/users
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

**Vorteile:** Stateless, sicher
**Nachteile:** Token-Management

#### 4. OAuth 2.0

```http
GET /api/users
Authorization: Bearer oauth2-token
```

**Vorteile:** Standard für Third-Party Auth
**Nachteile:** Komplex

### JWT (JSON Web Token)

#### JWT-Struktur

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.    ← Header
eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6Ik...  ← Payload
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c   ← Signature
```

**Header:**
```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

**Payload:**
```json
{
  "sub": "1234567890",
  "name": "Max Mustermann",
  "email": "max@example.com",
  "role": "admin",
  "iat": 1516239022,
  "exp": 1516242622
}
```

### Authorization (Was darfst du?)

#### Role-Based Access Control (RBAC)

```typescript
// Rollen definieren
enum Role {
  USER = 'user',
  ADMIN = 'admin',
  MODERATOR = 'moderator'
}

// Middleware für Autorisierung
function authorize(roles: Role[]) {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    next();
  };
}

// Verwendung
app.delete('/users/:id', 
  authenticate,
  authorize([Role.ADMIN]),
  deleteUser
);
```

#### Permission-Based

```typescript
// Permissions
enum Permission {
  READ_USERS = 'read:users',
  WRITE_USERS = 'write:users',
  DELETE_USERS = 'delete:users'
}

// Middleware
function requirePermission(permission: Permission) {
  return (req, res, next) => {
    if (!req.user.permissions.includes(permission)) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    next();
  };
}
```

---

## Express.js API erstellen

### Setup

```bash
npm init -y
npm install express
npm install --save-dev typescript @types/node @types/express ts-node nodemon

npx tsc --init
```

### Basis-Server

```typescript
// src/index.ts
import express, { Request, Response } from 'express';

const app = express();
const PORT = process.env.PORT || 3000;

// Middleware
app.use(express.json());

// Routes
app.get('/', (req: Request, res: Response) => {
  res.json({ message: 'Hello World!' });
});

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

### CRUD API

```typescript
// src/types/user.ts
export interface User {
  id: number;
  name: string;
  email: string;
  createdAt: Date;
}

// src/data/users.ts
let users: User[] = [
  { id: 1, name: 'Max', email: 'max@example.com', createdAt: new Date() }
];

// src/routes/users.ts
import express, { Request, Response } from 'express';

const router = express.Router();

// GET /api/users - Alle Users
router.get('/', (req: Request, res: Response) => {
  res.json({
    success: true,
    data: users
  });
});

// GET /api/users/:id - User by ID
router.get('/:id', (req: Request, res: Response) => {
  const id = parseInt(req.params.id);
  const user = users.find(u => u.id === id);

  if (!user) {
    return res.status(404).json({
      success: false,
      error: 'User not found'
    });
  }

  res.json({
    success: true,
    data: user
  });
});

// POST /api/users - User erstellen
router.post('/', (req: Request, res: Response) => {
  const { name, email } = req.body;

  if (!name || !email) {
    return res.status(400).json({
      success: false,
      error: 'Name and email are required'
    });
  }

  const newUser: User = {
    id: users.length + 1,
    name,
    email,
    createdAt: new Date()
  };

  users.push(newUser);

  res.status(201).json({
    success: true,
    data: newUser
  });
});

// PUT /api/users/:id - User aktualisieren
router.put('/:id', (req: Request, res: Response) => {
  const id = parseInt(req.params.id);
  const { name, email } = req.body;

  const userIndex = users.findIndex(u => u.id === id);

  if (userIndex === -1) {
    return res.status(404).json({
      success: false,
      error: 'User not found'
    });
  }

  users[userIndex] = {
    ...users[userIndex],
    name,
    email
  };

  res.json({
    success: true,
    data: users[userIndex]
  });
});

// PATCH /api/users/:id - User teilweise aktualisieren
router.patch('/:id', (req: Request, res: Response) => {
  const id = parseInt(req.params.id);
  const updates = req.body;

  const userIndex = users.findIndex(u => u.id === id);

  if (userIndex === -1) {
    return res.status(404).json({
      success: false,
      error: 'User not found'
    });
  }

  users[userIndex] = {
    ...users[userIndex],
    ...updates
  };

  res.json({
    success: true,
    data: users[userIndex]
  });
});

// DELETE /api/users/:id - User löschen
router.delete('/:id', (req: Request, res: Response) => {
  const id = parseInt(req.params.id);
  const userIndex = users.findIndex(u => u.id === id);

  if (userIndex === -1) {
    return res.status(404).json({
      success: false,
      error: 'User not found'
    });
  }

  users.splice(userIndex, 1);

  res.status(204).send();
});

export default router;

// src/index.ts
import userRoutes from './routes/users';

app.use('/api/users', userRoutes);
```

---

## Middleware

### Was ist Middleware?

Middleware sind Funktionen, die Zugriff auf Request und Response haben und die Request-Response-Pipeline modifizieren können.

```
Request → Middleware 1 → Middleware 2 → Route Handler → Response
```

### Logging Middleware

```typescript
// src/middleware/logger.ts
import { Request, Response, NextFunction } from 'express';

export function logger(req: Request, res: Response, next: NextFunction) {
  const start = Date.now();

  res.on('finish', () => {
    const duration = Date.now() - start;
    console.log(
      `${req.method} ${req.path} ${res.statusCode} - ${duration}ms`
    );
  });

  next();
}

// Verwendung
app.use(logger);
```

### Authentication Middleware

```typescript
// src/middleware/auth.ts
import { Request, Response, NextFunction } from 'express';
import jwt from 'jsonwebtoken';

interface AuthRequest extends Request {
  user?: any;
}

export function authenticate(
  req: AuthRequest,
  res: Response,
  next: NextFunction
) {
  const authHeader = req.headers.authorization;

  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'No token provided' });
  }

  const token = authHeader.substring(7);

  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET!);
    req.user = decoded;
    next();
  } catch (error) {
    return res.status(401).json({ error: 'Invalid token' });
  }
}

// Verwendung
app.get('/api/profile', authenticate, (req: AuthRequest, res) => {
  res.json({ user: req.user });
});
```

### Request Validation Middleware

```typescript
// src/middleware/validate.ts
import { Request, Response, NextFunction } from 'express';
import { z, ZodSchema } from 'zod';

export function validate(schema: ZodSchema) {
  return (req: Request, res: Response, next: NextFunction) => {
    try {
      schema.parse(req.body);
      next();
    } catch (error) {
      if (error instanceof z.ZodError) {
        return res.status(422).json({
          error: 'Validation Error',
          details: error.errors
        });
      }
      next(error);
    }
  };
}

// Schema definieren
const userSchema = z.object({
  name: z.string().min(2).max(100),
  email: z.string().email(),
  age: z.number().min(18).optional()
});

// Verwendung
app.post('/api/users', validate(userSchema), createUser);
```

---

## Error Handling

### Zentrale Error Handler

```typescript
// src/middleware/errorHandler.ts
import { Request, Response, NextFunction } from 'express';

export class AppError extends Error {
  constructor(
    public statusCode: number,
    public message: string,
    public isOperational = true
  ) {
    super(message);
    Object.setPrototypeOf(this, AppError.prototype);
  }
}

export function errorHandler(
  err: Error,
  req: Request,
  res: Response,
  next: NextFunction
) {
  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      success: false,
      error: {
        message: err.message,
        statusCode: err.statusCode
      }
    });
  }

  // Unerwarteter Fehler
  console.error('ERROR:', err);

  res.status(500).json({
    success: false,
    error: {
      message: 'Internal Server Error',
      statusCode: 500
    }
  });
}

// Am Ende der Middleware-Chain
app.use(errorHandler);
```

### Async Error Handling

```typescript
// src/utils/asyncHandler.ts
import { Request, Response, NextFunction } from 'express';

export function asyncHandler(fn: Function) {
  return (req: Request, res: Response, next: NextFunction) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
}

// Verwendung
app.get('/api/users/:id', asyncHandler(async (req, res) => {
  const user = await getUserById(req.params.id);
  
  if (!user) {
    throw new AppError(404, 'User not found');
  }
  
  res.json({ success: true, data: user });
}));
```

### 404 Handler

```typescript
// src/middleware/notFound.ts
import { Request, Response } from 'express';

export function notFound(req: Request, res: Response) {
  res.status(404).json({
    success: false,
    error: {
      message: `Route ${req.method} ${req.path} not found`,
      statusCode: 404
    }
  });
}

// Vor dem Error Handler
app.use(notFound);
app.use(errorHandler);
```

---

## Validation

### Mit Zod

```typescript
import { z } from 'zod';

// User Schema
const userSchema = z.object({
  name: z.string()
    .min(2, 'Name must be at least 2 characters')
    .max(100, 'Name must be at most 100 characters'),
  email: z.string()
    .email('Invalid email address'),
  age: z.number()
    .int('Age must be an integer')
    .min(18, 'Must be at least 18 years old')
    .optional(),
  role: z.enum(['user', 'admin', 'moderator'])
    .default('user')
});

// Post Schema
const postSchema = z.object({
  title: z.string().min(5).max(200),
  content: z.string().min(10),
  tags: z.array(z.string()).min(1).max(5),
  published: z.boolean().default(false)
});

// Nested Schema
const commentSchema = z.object({
  content: z.string().min(1).max(500),
  author: userSchema,
  postId: z.number().positive()
});
```

### Custom Validation

```typescript
const passwordSchema = z.string()
  .min(8, 'Password must be at least 8 characters')
  .refine(
    (password) => /[A-Z]/.test(password),
    'Password must contain at least one uppercase letter'
  )
  .refine(
    (password) => /[0-9]/.test(password),
    'Password must contain at least one number'
  )
  .refine(
    (password) => /[!@#$%^&*]/.test(password),
    'Password must contain at least one special character'
  );

const registerSchema = z.object({
  email: z.string().email(),
  password: passwordSchema,
  confirmPassword: z.string()
}).refine(
  (data) => data.password === data.confirmPassword,
  {
    message: "Passwords don't match",
    path: ['confirmPassword']
  }
);
```

---

## CORS

### Was ist CORS?

**Cross-Origin Resource Sharing** erlaubt Anfragen von anderen Domains.

### CORS Setup

```bash
npm install cors
npm install --save-dev @types/cors
```

```typescript
import cors from 'cors';

// Alle Origins erlauben (Entwicklung)
app.use(cors());

// Spezifische Origins
app.use(cors({
  origin: 'https://example.com'
}));

// Mehrere Origins
app.use(cors({
  origin: ['https://example.com', 'https://app.example.com']
}));

// Dynamische Origins
app.use(cors({
  origin: (origin, callback) => {
    const allowedOrigins = ['https://example.com', 'http://localhost:3000'];
    
    if (!origin || allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error('Not allowed by CORS'));
    }
  }
}));

// Mit Credentials
app.use(cors({
  origin: 'https://example.com',
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));
```

---

## Rate Limiting

### Mit express-rate-limit

```bash
npm install express-rate-limit
```

```typescript
import rateLimit from 'express-rate-limit';

// Basis Rate Limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 Minuten
  max: 100, // Max 100 Requests
  message: 'Too many requests, please try again later',
  standardHeaders: true,
  legacyHeaders: false,
});

app.use('/api/', limiter);

// Stricter für Login
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  message: 'Too many login attempts, please try again later',
  skipSuccessfulRequests: true
});

app.post('/api/login', loginLimiter, login);

// Custom Key Generator (z.B. nach User)
const userLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 10,
  keyGenerator: (req) => {
    return req.user?.id || req.ip;
  }
});
```

---

## API Documentation

### Mit Swagger/OpenAPI

```bash
npm install swagger-ui-express swagger-jsdoc
npm install --save-dev @types/swagger-ui-express @types/swagger-jsdoc
```

```typescript
// src/swagger.ts
import swaggerJsdoc from 'swagger-jsdoc';
import swaggerUi from 'swagger-ui-express';

const options = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'My API',
      version: '1.0.0',
      description: 'API Documentation',
    },
    servers: [
      {
        url: 'http://localhost:3000',
        description: 'Development server',
      },
    ],
    components: {
      securitySchemes: {
        bearerAuth: {
          type: 'http',
          scheme: 'bearer',
          bearerFormat: 'JWT',
        },
      },
    },
  },
  apis: ['./src/routes/*.ts'],
};

const specs = swaggerJsdoc(options);

export function setupSwagger(app: any) {
  app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(specs));
}
```

### JSDoc Annotations

```typescript
/**
 * @swagger
 * /api/users:
 *   get:
 *     summary: Get all users
 *     tags: [Users]
 *     parameters:
 *       - in: query
 *         name: page
 *         schema:
 *           type: integer
 *         description: Page number
 *     responses:
 *       200:
 *         description: List of users
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
 */
router.get('/', getUsers);

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
 *     responses:
 *       200:
 *         description: User found
 *       404:
 *         description: User not found
 */
router.get('/:id', getUserById);

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
 *         name:
 *           type: string
 *         email:
 *           type: string
 *           format: email
 *         createdAt:
 *           type: string
 *           format: date-time
 */
```

---

## GraphQL Einführung

### Was ist GraphQL?

GraphQL ist eine Query-Sprache für APIs, entwickelt von Facebook.

**Vorteile:**
- Flexible Queries
- Kein Over-/Under-fetching
- Strongly Typed
- Single Endpoint

### Setup mit Apollo Server

```bash
npm install apollo-server-express graphql
npm install --save-dev @types/graphql
```

```typescript
// src/graphql/schema.ts
import { gql } from 'apollo-server-express';

export const typeDefs = gql`
  type User {
    id: ID!
    name: String!
    email: String!
    posts: [Post!]!
  }

  type Post {
    id: ID!
    title: String!
    content: String!
    author: User!
  }

  type Query {
    users: [User!]!
    user(id: ID!): User
    posts: [Post!]!
    post(id: ID!): Post
  }

  type Mutation {
    createUser(name: String!, email: String!): User!
    createPost(title: String!, content: String!, authorId: ID!): Post!
    updateUser(id: ID!, name: String, email: String): User
    deleteUser(id: ID!): Boolean!
  }
`;
```

```typescript
// src/graphql/resolvers.ts
export const resolvers = {
  Query: {
    users: () => users,
    user: (_, { id }) => users.find(u => u.id === id),
    posts: () => posts,
    post: (_, { id }) => posts.find(p => p.id === id),
  },

  Mutation: {
    createUser: (_, { name, email }) => {
      const newUser = {
        id: String(users.length + 1),
        name,
        email,
        posts: []
      };
      users.push(newUser);
      return newUser;
    },

    createPost: (_, { title, content, authorId }) => {
      const newPost = {
        id: String(posts.length + 1),
        title,
        content,
        authorId
      };
      posts.push(newPost);
      return newPost;
    },
  },

  User: {
    posts: (user) => posts.filter(p => p.authorId === user.id),
  },

  Post: {
    author: (post) => users.find(u => u.id === post.authorId),
  },
};
```

```typescript
// src/index.ts
import { ApolloServer } from 'apollo-server-express';
import { typeDefs } from './graphql/schema';
import { resolvers } from './graphql/resolvers';

const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: ({ req }) => {
    // Context für alle Resolver
    return { user: req.user };
  },
});

await server.start();
server.applyMiddleware({ app, path: '/graphql' });
```

### GraphQL Queries

```graphql
# Alle Users mit Namen
query {
  users {
    id
    name
  }
}

# User mit Posts
query {
  user(id: "1") {
    id
    name
    email
    posts {
      id
      title
    }
  }
}

# Mutation - User erstellen
mutation {
  createUser(name: "Max", email: "max@example.com") {
    id
    name
    email
  }
}

# Mit Variablen
mutation CreateUser($name: String!, $email: String!) {
  createUser(name: $name, email: $email) {
    id
    name
    email
  }
}
```

---

## Praktische Projekte

### Projekt 1: Blog API

```typescript
// src/types/blog.ts
export interface User {
  id: number;
  username: string;
  email: string;
  password: string;
  createdAt: Date;
}

export interface Post {
  id: number;
  title: string;
  content: string;
  authorId: number;
  published: boolean;
  createdAt: Date;
  updatedAt: Date;
}

export interface Comment {
  id: number;
  content: string;
  postId: number;
  userId: number;
  createdAt: Date;
}
```

```typescript
// src/routes/posts.ts
import express from 'express';
import { authenticate } from '../middleware/auth';
import { validate } from '../middleware/validate';
import { z } from 'zod';

const router = express.Router();

const postSchema = z.object({
  title: z.string().min(5).max(200),
  content: z.string().min(10),
  published: z.boolean().optional()
});

// GET /api/posts
router.get('/', async (req, res) => {
  const { page = 1, limit = 10, author, published } = req.query;
  
  // Pagination
  const offset = (Number(page) - 1) * Number(limit);
  
  // Filter
  let query = 'SELECT * FROM posts WHERE 1=1';
  const params: any[] = [];
  
  if (author) {
    query += ' AND authorId = ?';
    params.push(author);
  }
  
  if (published !== undefined) {
    query += ' AND published = ?';
    params.push(published === 'true');
  }
  
  query += ' LIMIT ? OFFSET ?';
  params.push(Number(limit), offset);
  
  const posts = await db.query(query, params);
  const total = await db.query('SELECT COUNT(*) FROM posts')[0].count;
  
  res.json({
    success: true,
    data: posts,
    pagination: {
      page: Number(page),
      limit: Number(limit),
      total,
      totalPages: Math.ceil(total / Number(limit))
    }
  });
});

// POST /api/posts
router.post('/', authenticate, validate(postSchema), async (req, res) => {
  const { title, content, published = false } = req.body;
  const authorId = req.user.id;
  
  const post = await db.insert('posts', {
    title,
    content,
    authorId,
    published,
    createdAt: new Date(),
    updatedAt: new Date()
  });
  
  res.status(201).json({
    success: true,
    data: post
  });
});

// GET /api/posts/:id
router.get('/:id', async (req, res) => {
  const post = await db.findOne('posts', { id: req.params.id });
  
  if (!post) {
    return res.status(404).json({
      success: false,
      error: 'Post not found'
    });
  }
  
  // Include author and comments
  const author = await db.findOne('users', { id: post.authorId });
  const comments = await db.find('comments', { postId: post.id });
  
  res.json({
    success: true,
    data: {
      ...post,
      author,
      comments
    }
  });
});

export default router;
```

### Projekt 2: E-Commerce API

```typescript
// src/routes/products.ts
import express from 'express';

const router = express.Router();

// GET /api/products
router.get('/', async (req, res) => {
  const {
    category,
    minPrice,
    maxPrice,
    sort = '-createdAt',
    page = 1,
    limit = 20
  } = req.query;
  
  let query: any = {};
  
  if (category) {
    query.category = category;
  }
  
  if (minPrice || maxPrice) {
    query.price = {};
    if (minPrice) query.price.$gte = Number(minPrice);
    if (maxPrice) query.price.$lte = Number(maxPrice);
  }
  
  const products = await Product.find(query)
    .sort(sort)
    .skip((Number(page) - 1) * Number(limit))
    .limit(Number(limit))
    .populate('category');
  
  const total = await Product.countDocuments(query);
  
  res.json({
    success: true,
    data: products,
    pagination: {
      page: Number(page),
      limit: Number(limit),
      total,
      totalPages: Math.ceil(total / Number(limit))
    }
  });
});

// POST /api/products/:id/reviews
router.post('/:id/reviews', authenticate, async (req, res) => {
  const { rating, comment } = req.body;
  const productId = req.params.id;
  const userId = req.user.id;
  
  // Check if product exists
  const product = await Product.findById(productId);
  if (!product) {
    return res.status(404).json({
      success: false,
      error: 'Product not found'
    });
  }
  
  // Check if user already reviewed
  const existingReview = await Review.findOne({
    productId,
    userId
  });
  
  if (existingReview) {
    return res.status(409).json({
      success: false,
      error: 'You have already reviewed this product'
    });
  }
  
  const review = await Review.create({
    productId,
    userId,
    rating,
    comment
  });
  
  // Update product rating
  const reviews = await Review.find({ productId });
  const avgRating = reviews.reduce((sum, r) => sum + r.rating, 0) / reviews.length;
  
  await Product.findByIdAndUpdate(productId, {
    rating: avgRating,
    numReviews: reviews.length
  });
  
  res.status(201).json({
    success: true,
    data: review
  });
});

export default router;
```

---

## Zusammenfassung

Du hast nun gelernt:

✅ **REST API Grundlagen**: Prinzipien, Ressourcen, URL-Struktur
✅ **HTTP-Methoden**: GET, POST, PUT, PATCH, DELETE
✅ **Status Codes**: 2xx, 3xx, 4xx, 5xx
✅ **API Design**: Versionierung, Pagination, Filtering, Sorting
✅ **Authentication**: JWT, OAuth, Bearer Tokens
✅ **Authorization**: RBAC, Permissions
✅ **Express.js**: Server Setup, CRUD-Operationen
✅ **Middleware**: Logging, Auth, Validation
✅ **Error Handling**: Zentrale Handler, Async Errors
✅ **Validation**: Zod, Custom Validators
✅ **CORS**: Cross-Origin Requests
✅ **Rate Limiting**: Request-Beschränkungen
✅ **Documentation**: Swagger/OpenAPI
✅ **GraphQL**: Alternative zu REST

### Hilfreiche Ressourcen

- [REST API Tutorial](https://restfulapi.net/)
- [Express.js Documentation](https://expressjs.com/)
- [HTTP Status Codes](https://httpstatuses.com/)
- [JWT.io](https://jwt.io/)
- [Swagger](https://swagger.io/)
- [GraphQL](https://graphql.org/)

Viel Erfolg mit Web-APIs! 🚀
