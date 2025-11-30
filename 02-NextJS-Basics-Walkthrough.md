# Next.js Basics - Kompletter Walkthrough

## Inhaltsverzeichnis
1. [Einführung](#einführung)
2. [Installation und Setup](#installation-und-setup)
3. [Projektstruktur](#projektstruktur)
4. [App Router vs Pages Router](#app-router-vs-pages-router)
5. [Routing](#routing)
6. [Server Components vs Client Components](#server-components-vs-client-components)
7. [Data Fetching](#data-fetching)
8. [API Routes](#api-routes)
9. [Styling](#styling)
10. [Layouts und Templates](#layouts-und-templates)
11. [Navigation](#navigation)
12. [Metadata und SEO](#metadata-und-seo)
13. [Loading und Error Handling](#loading-und-error-handling)
14. [Praktische Projekte](#praktische-projekte)

---

## Einführung

### Was ist Next.js?

Next.js ist ein React-Framework für Production, entwickelt von Vercel. Es bietet eine optimale Entwicklererfahrung mit allen Features, die man für Production braucht.

### Warum Next.js?

✅ **Server-Side Rendering (SSR)**: Bessere Performance und SEO
✅ **Static Site Generation (SSG)**: Blitzschnelle Ladezeiten
✅ **File-based Routing**: Einfaches Routing ohne zusätzliche Konfiguration
✅ **API Routes**: Backend-Funktionen direkt im Projekt
✅ **Image Optimization**: Automatische Bildoptimierung
✅ **TypeScript Support**: Erstklassige TypeScript-Unterstützung
✅ **Hot Reload**: Sofortige Aktualisierung während der Entwicklung

### Next.js vs React

| Feature | React | Next.js |
|---------|-------|---------|
| Routing | Manuell (React Router) | Automatisch (File-based) |
| Rendering | Client-Side (CSR) | SSR, SSG, ISR, CSR |
| SEO | Schwierig | Optimiert |
| Performance | Gut | Exzellent |
| Setup | Komplex | Einfach |
| API | Separater Server | Integriert |

---

## Installation und Setup

### Voraussetzungen

- **Node.js** 18.17 oder höher
- **npm**, **yarn**, oder **pnpm**
- Code-Editor (empfohlen: VS Code)

### Neues Next.js Projekt erstellen

#### Option 1: Mit create-next-app (Empfohlen)

```bash
# Mit npm
npx create-next-app@latest mein-next-projekt

# Mit yarn
yarn create next-app mein-next-projekt

# Mit pnpm
pnpm create next-app mein-next-projekt
```

#### Interaktive Konfiguration

Bei der Installation wirst du nach folgenden Optionen gefragt:

```
✔ Would you like to use TypeScript? … Yes
✔ Would you like to use ESLint? … Yes
✔ Would you like to use Tailwind CSS? … Yes
✔ Would you like to use `src/` directory? … No
✔ Would you like to use App Router? (recommended) … Yes
✔ Would you like to customize the default import alias (@/*)? … No
```

#### Option 2: Manuelles Setup

```bash
# Projektordner erstellen
mkdir mein-next-projekt
cd mein-next-projekt

# package.json initialisieren
npm init -y

# Next.js installieren
npm install next@latest react@latest react-dom@latest

# TypeScript installieren (optional)
npm install --save-dev typescript @types/react @types/node
```

**package.json Scripts hinzufügen:**

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  }
}
```

### Projekt starten

```bash
# Development Server starten
npm run dev
```

Öffne deinen Browser und navigiere zu `http://localhost:3000`

---

## Projektstruktur

### Standard-Projektstruktur (App Router)

```
mein-next-projekt/
├── app/
│   ├── layout.tsx          # Root Layout
│   ├── page.tsx            # Startseite (/)
│   ├── globals.css         # Globale Styles
│   ├── about/
│   │   └── page.tsx        # About-Seite (/about)
│   └── api/
│       └── route.ts        # API Route
├── public/
│   ├── images/
│   └── favicon.ico
├── components/
│   └── Header.tsx
├── lib/
│   └── utils.ts
├── .next/                  # Build-Output (automatisch generiert)
├── node_modules/
├── next.config.js          # Next.js Konfiguration
├── tsconfig.json           # TypeScript Konfiguration
├── package.json
└── .gitignore
```

### Wichtige Dateien erklärt

#### `app/layout.tsx` - Root Layout
Das Haupt-Layout, das alle Seiten umschließt.

```typescript
import type { Metadata } from 'next'
import './globals.css'

export const metadata: Metadata = {
  title: 'Meine Next.js App',
  description: 'Erstellt mit Next.js',
}

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="de">
      <body>{children}</body>
    </html>
  )
}
```

#### `app/page.tsx` - Startseite
Die Hauptseite deiner Anwendung.

```typescript
export default function Home() {
  return (
    <main>
      <h1>Willkommen bei Next.js!</h1>
      <p>Dies ist die Startseite.</p>
    </main>
  )
}
```

#### `next.config.js` - Konfiguration

```javascript
/** @type {import('next').NextConfig} */
const nextConfig = {
  reactStrictMode: true,
  images: {
    domains: ['example.com'],
  },
}

module.exports = nextConfig
```

---

## App Router vs Pages Router

### App Router (Next.js 13+) - Empfohlen

Der neue **App Router** nutzt das `app/`-Verzeichnis und React Server Components.

**Vorteile:**
- Server Components by default
- Bessere Performance
- Layouts und Nested Routing
- Streaming und Suspense
- Moderne React Features

**Struktur:**
```
app/
├── layout.tsx       # Layout
├── page.tsx         # Route
├── loading.tsx      # Loading UI
├── error.tsx        # Error UI
└── not-found.tsx    # 404 Page
```

### Pages Router (Legacy)

Der klassische **Pages Router** nutzt das `pages/`-Verzeichnis.

**Struktur:**
```
pages/
├── _app.tsx         # Custom App
├── _document.tsx    # Custom Document
├── index.tsx        # Route (/)
└── about.tsx        # Route (/about)
```

> **Hinweis**: Dieser Guide fokussiert sich auf den modernen **App Router**.

---

## Routing

### File-based Routing

Next.js nutzt das Dateisystem für Routing. Jede `page.tsx` im `app/`-Verzeichnis wird zu einer Route.

#### Einfache Routes

```
app/
├── page.tsx                    → /
├── about/
│   └── page.tsx               → /about
├── blog/
│   └── page.tsx               → /blog
└── kontakt/
    └── page.tsx               → /kontakt
```

**Beispiel: `app/about/page.tsx`**

```typescript
export default function AboutPage() {
  return (
    <div>
      <h1>Über uns</h1>
      <p>Willkommen auf unserer About-Seite!</p>
    </div>
  )
}
```

### Dynamische Routes

Dynamische Routen werden mit eckigen Klammern `[param]` erstellt.

```
app/
└── blog/
    ├── page.tsx               → /blog
    └── [slug]/
        └── page.tsx           → /blog/:slug
```

**Beispiel: `app/blog/[slug]/page.tsx`**

```typescript
interface PageProps {
  params: {
    slug: string
  }
}

export default function BlogPost({ params }: PageProps) {
  return (
    <div>
      <h1>Blog Post: {params.slug}</h1>
      <p>Dies ist der Artikel mit dem Slug: {params.slug}</p>
    </div>
  )
}
```

**Verwendung:**
- `/blog/mein-erster-post` → params.slug = "mein-erster-post"
- `/blog/typescript-guide` → params.slug = "typescript-guide"

### Catch-all Routes

Mit `[...param]` kannst du mehrere Segmente erfassen.

```
app/
└── shop/
    └── [...categories]/
        └── page.tsx           → /shop/* (beliebig viele Segmente)
```

**Beispiel: `app/shop/[...categories]/page.tsx`**

```typescript
interface PageProps {
  params: {
    categories: string[]
  }
}

export default function ShopPage({ params }: PageProps) {
  return (
    <div>
      <h1>Shop</h1>
      <p>Kategorien: {params.categories.join(' / ')}</p>
    </div>
  )
}
```

**Verwendung:**
- `/shop/elektronik` → categories = ["elektronik"]
- `/shop/elektronik/laptops` → categories = ["elektronik", "laptops"]
- `/shop/elektronik/laptops/gaming` → categories = ["elektronik", "laptops", "gaming"]

### Optional Catch-all Routes

Mit `[[...param]]` sind die Segmente optional.

```
app/
└── docs/
    └── [[...slug]]/
        └── page.tsx
```

**Matches:**
- `/docs`
- `/docs/einfuehrung`
- `/docs/einfuehrung/installation`

### Route Groups

Mit `(ordnername)` kannst du Ordner gruppieren ohne die URL zu beeinflussen.

```
app/
├── (marketing)/
│   ├── about/
│   │   └── page.tsx          → /about
│   └── kontakt/
│       └── page.tsx          → /kontakt
└── (shop)/
    ├── produkte/
    │   └── page.tsx          → /produkte
    └── warenkorb/
        └── page.tsx          → /warenkorb
```

### Parallel Routes

Mit `@ordner` kannst du mehrere Seiten parallel rendern.

```
app/
└── dashboard/
    ├── @analytics/
    │   └── page.tsx
    ├── @team/
    │   └── page.tsx
    └── layout.tsx
```

**`app/dashboard/layout.tsx`:**

```typescript
export default function DashboardLayout({
  children,
  analytics,
  team,
}: {
  children: React.ReactNode
  analytics: React.ReactNode
  team: React.ReactNode
}) {
  return (
    <div>
      {children}
      <div className="grid grid-cols-2">
        <div>{analytics}</div>
        <div>{team}</div>
      </div>
    </div>
  )
}
```

---

## Server Components vs Client Components

### Server Components (Standard)

Standardmäßig sind alle Components in Next.js **Server Components**.

**Vorteile:**
- Kein JavaScript zum Client gesendet
- Direkter Zugriff auf Backend-Ressourcen
- Bessere Performance
- Automatisches Code-Splitting

**Beispiel:**

```typescript
// app/components/ServerComponent.tsx
// Kein "use client" = Server Component

async function getData() {
  const res = await fetch('https://api.example.com/data')
  return res.json()
}

export default async function ServerComponent() {
  const data = await getData()
  
  return (
    <div>
      <h2>Server Component</h2>
      <p>Daten: {JSON.stringify(data)}</p>
    </div>
  )
}
```

**Wann verwenden:**
- Daten von einer API fetchen
- Backend-Ressourcen nutzen
- Sensible Informationen (API Keys, Tokens)
- Große Dependencies, die nicht zum Client sollen

### Client Components

Client Components laufen im Browser und können Interaktivität haben.

**Aktivierung mit `"use client"`:**

```typescript
// app/components/Counter.tsx
'use client'

import { useState } from 'react'

export default function Counter() {
  const [count, setCount] = useState(0)

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
    </div>
  )
}
```

**Wann verwenden:**
- Interaktive UI (onClick, onChange, etc.)
- State Management (useState, useReducer)
- Effects (useEffect)
- Browser-APIs (localStorage, window, etc.)
- Event Listener
- Custom Hooks

### Kombinieren von Server und Client Components

```typescript
// app/page.tsx (Server Component)
import ClientCounter from './components/ClientCounter'
import { Suspense } from 'react'

async function getServerData() {
  // Läuft auf dem Server
  const res = await fetch('https://api.example.com/data')
  return res.json()
}

export default async function HomePage() {
  const data = await getServerData()

  return (
    <div>
      <h1>Meine Seite</h1>
      
      {/* Server-gerenderter Inhalt */}
      <p>Server Daten: {data.message}</p>
      
      {/* Client Component für Interaktivität */}
      <ClientCounter />
      
      {/* Server Component mit Suspense */}
      <Suspense fallback={<p>Laden...</p>}>
        <AsyncComponent />
      </Suspense>
    </div>
  )
}
```

### Best Practices

**DO ✅**
```typescript
// Client Component nur für interaktive Teile
'use client'

import { useState } from 'react'

export default function InteractiveButton() {
  const [clicked, setClicked] = useState(false)
  
  return (
    <button onClick={() => setClicked(true)}>
      {clicked ? 'Geklickt!' : 'Klick mich'}
    </button>
  )
}
```

**DON'T ❌**
```typescript
// Gesamte Seite als Client Component (unnötig)
'use client'

export default function Page() {
  // Nur ein Button ist interaktiv, aber die ganze Seite ist Client-side
  return (
    <div>
      <h1>Statischer Inhalt</h1>
      <button onClick={() => alert('Click')}>Button</button>
    </div>
  )
}
```

---

## Data Fetching

### Server-side Data Fetching

#### In Server Components (Empfohlen)

```typescript
// app/posts/page.tsx
interface Post {
  id: number
  title: string
  body: string
}

async function getPosts(): Promise<Post[]> {
  const res = await fetch('https://jsonplaceholder.typicode.com/posts')
  
  if (!res.ok) {
    throw new Error('Failed to fetch posts')
  }
  
  return res.json()
}

export default async function PostsPage() {
  const posts = await getPosts()

  return (
    <div>
      <h1>Blog Posts</h1>
      <ul>
        {posts.map(post => (
          <li key={post.id}>
            <h2>{post.title}</h2>
            <p>{post.body}</p>
          </li>
        ))}
      </ul>
    </div>
  )
}
```

#### Mit Caching

```typescript
// Cached für 60 Sekunden
async function getData() {
  const res = await fetch('https://api.example.com/data', {
    next: { revalidate: 60 } // ISR - Incremental Static Regeneration
  })
  return res.json()
}

// Kein Caching (immer fresh)
async function getLiveData() {
  const res = await fetch('https://api.example.com/live', {
    cache: 'no-store'
  })
  return res.json()
}

// Standard: Cached forever
async function getStaticData() {
  const res = await fetch('https://api.example.com/static')
  return res.json()
}
```

### Parallele Daten laden

```typescript
// app/dashboard/page.tsx
async function getUser() {
  const res = await fetch('https://api.example.com/user')
  return res.json()
}

async function getPosts() {
  const res = await fetch('https://api.example.com/posts')
  return res.json()
}

export default async function Dashboard() {
  // Parallel fetching mit Promise.all
  const [user, posts] = await Promise.all([
    getUser(),
    getPosts()
  ])

  return (
    <div>
      <h1>Dashboard für {user.name}</h1>
      <div>
        <h2>Posts</h2>
        {posts.map((post: any) => (
          <div key={post.id}>{post.title}</div>
        ))}
      </div>
    </div>
  )
}
```

### Client-side Data Fetching

```typescript
// app/components/ClientData.tsx
'use client'

import { useState, useEffect } from 'react'

interface User {
  id: number
  name: string
  email: string
}

export default function ClientData() {
  const [users, setUsers] = useState<User[]>([])
  const [loading, setLoading] = useState(true)
  const [error, setError] = useState<string | null>(null)

  useEffect(() => {
    async function fetchUsers() {
      try {
        const res = await fetch('https://jsonplaceholder.typicode.com/users')
        if (!res.ok) throw new Error('Fehler beim Laden')
        const data = await res.json()
        setUsers(data)
      } catch (err) {
        setError(err instanceof Error ? err.message : 'Ein Fehler ist aufgetreten')
      } finally {
        setLoading(false)
      }
    }

    fetchUsers()
  }, [])

  if (loading) return <p>Lädt...</p>
  if (error) return <p>Fehler: {error}</p>

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>
          {user.name} - {user.email}
        </li>
      ))}
    </ul>
  )
}
```

### Mit SWR (Empfohlen für Client-side)

```typescript
// Erst installieren: npm install swr

'use client'

import useSWR from 'swr'

const fetcher = (url: string) => fetch(url).then(r => r.json())

export default function Profile() {
  const { data, error, isLoading } = useSWR(
    'https://jsonplaceholder.typicode.com/users/1',
    fetcher
  )

  if (error) return <div>Fehler beim Laden</div>
  if (isLoading) return <div>Lädt...</div>

  return (
    <div>
      <h1>{data.name}</h1>
      <p>{data.email}</p>
    </div>
  )
}
```

---

## API Routes

Next.js ermöglicht es, API-Endpunkte direkt im Projekt zu erstellen.

### Basis API Route

```typescript
// app/api/hello/route.ts
import { NextResponse } from 'next/server'

export async function GET() {
  return NextResponse.json({ 
    message: 'Hallo von der API!' 
  })
}
```

**Zugriff:** `http://localhost:3000/api/hello`

### Alle HTTP-Methoden

```typescript
// app/api/posts/route.ts
import { NextRequest, NextResponse } from 'next/server'

// GET - Daten abrufen
export async function GET(request: NextRequest) {
  const posts = [
    { id: 1, title: 'Erster Post' },
    { id: 2, title: 'Zweiter Post' }
  ]
  
  return NextResponse.json(posts)
}

// POST - Daten erstellen
export async function POST(request: NextRequest) {
  const body = await request.json()
  
  // Hier würde man normalerweise in einer DB speichern
  const newPost = {
    id: Date.now(),
    ...body
  }
  
  return NextResponse.json(newPost, { status: 201 })
}

// PUT - Daten aktualisieren
export async function PUT(request: NextRequest) {
  const body = await request.json()
  
  return NextResponse.json({ 
    message: 'Updated',
    data: body
  })
}

// DELETE - Daten löschen
export async function DELETE(request: NextRequest) {
  return NextResponse.json({ 
    message: 'Deleted' 
  })
}
```

### Dynamische API Routes

```typescript
// app/api/posts/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server'

interface Params {
  params: {
    id: string
  }
}

export async function GET(
  request: NextRequest,
  { params }: Params
) {
  const { id } = params
  
  // Simulierte Datenbankabfrage
  const post = {
    id: parseInt(id),
    title: `Post ${id}`,
    content: 'Lorem ipsum...'
  }
  
  return NextResponse.json(post)
}

export async function DELETE(
  request: NextRequest,
  { params }: Params
) {
  const { id } = params
  
  // Hier würde man aus der DB löschen
  
  return NextResponse.json({ 
    message: `Post ${id} gelöscht` 
  })
}
```

### Query Parameters

```typescript
// app/api/search/route.ts
import { NextRequest, NextResponse } from 'next/server'

export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams
  const query = searchParams.get('q')
  const limit = searchParams.get('limit') || '10'
  
  // Simulierte Suche
  const results = {
    query,
    limit: parseInt(limit),
    results: ['Ergebnis 1', 'Ergebnis 2']
  }
  
  return NextResponse.json(results)
}
```

**Verwendung:** `GET /api/search?q=next.js&limit=5`

### Request Headers

```typescript
// app/api/protected/route.ts
import { NextRequest, NextResponse } from 'next/server'

export async function GET(request: NextRequest) {
  const authHeader = request.headers.get('authorization')
  
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return NextResponse.json(
      { error: 'Unauthorized' },
      { status: 401 }
    )
  }
  
  const token = authHeader.split(' ')[1]
  
  // Token validieren
  if (token !== 'geheimes-token') {
    return NextResponse.json(
      { error: 'Invalid token' },
      { status: 403 }
    )
  }
  
  return NextResponse.json({ 
    message: 'Geschützte Daten',
    data: { userId: 1, name: 'Max' }
  })
}
```

### Fehlerbehandlung

```typescript
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server'

export async function GET(request: NextRequest) {
  try {
    // Simulierte Datenbankabfrage
    const users = await fetchUsersFromDB()
    
    return NextResponse.json(users)
  } catch (error) {
    console.error('Fehler beim Abrufen der Benutzer:', error)
    
    return NextResponse.json(
      { 
        error: 'Interner Serverfehler',
        message: error instanceof Error ? error.message : 'Unbekannter Fehler'
      },
      { status: 500 }
    )
  }
}

async function fetchUsersFromDB() {
  // Simulierte DB-Abfrage
  return [
    { id: 1, name: 'Max' },
    { id: 2, name: 'Anna' }
  ]
}
```

### CORS Headers

```typescript
// app/api/public/route.ts
import { NextRequest, NextResponse } from 'next/server'

export async function GET(request: NextRequest) {
  const data = { message: 'Öffentliche API' }
  
  return NextResponse.json(data, {
    headers: {
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
      'Access-Control-Allow-Headers': 'Content-Type, Authorization',
    },
  })
}

export async function OPTIONS(request: NextRequest) {
  return new NextResponse(null, {
    status: 200,
    headers: {
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Methods': 'GET, POST, PUT, DELETE, OPTIONS',
      'Access-Control-Allow-Headers': 'Content-Type, Authorization',
    },
  })
}
```

---

## Styling

### CSS Modules

CSS Modules sind standardmäßig in Next.js verfügbar.

```typescript
// app/components/Button.module.css
.button {
  background-color: #0070f3;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.button:hover {
  background-color: #0051cc;
}

.primary {
  background-color: #0070f3;
}

.secondary {
  background-color: #6c757d;
}
```

```typescript
// app/components/Button.tsx
import styles from './Button.module.css'

interface ButtonProps {
  children: React.ReactNode
  variant?: 'primary' | 'secondary'
  onClick?: () => void
}

export default function Button({ 
  children, 
  variant = 'primary',
  onClick 
}: ButtonProps) {
  return (
    <button 
      className={`${styles.button} ${styles[variant]}`}
      onClick={onClick}
    >
      {children}
    </button>
  )
}
```

### Tailwind CSS

Tailwind ist bei der Installation bereits integrierbar.

```typescript
// app/components/Card.tsx
export default function Card({ 
  title, 
  children 
}: { 
  title: string
  children: React.ReactNode 
}) {
  return (
    <div className="bg-white rounded-lg shadow-md p-6 hover:shadow-lg transition-shadow">
      <h2 className="text-2xl font-bold mb-4 text-gray-800">
        {title}
      </h2>
      <div className="text-gray-600">
        {children}
      </div>
    </div>
  )
}
```

**Tailwind mit dynamischen Klassen:**

```typescript
'use client'

import { useState } from 'react'

export default function Alert() {
  const [type, setType] = useState<'info' | 'success' | 'error'>('info')
  
  const colors = {
    info: 'bg-blue-100 text-blue-800 border-blue-300',
    success: 'bg-green-100 text-green-800 border-green-300',
    error: 'bg-red-100 text-red-800 border-red-300',
  }
  
  return (
    <div className={`p-4 border-l-4 ${colors[type]}`}>
      <p>Dies ist eine {type} Nachricht!</p>
    </div>
  )
}
```

### Global Styles

```css
/* app/globals.css */
@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
  --primary-color: #0070f3;
  --secondary-color: #6c757d;
  --background: #ffffff;
  --foreground: #000000;
}

* {
  box-sizing: border-box;
  padding: 0;
  margin: 0;
}

body {
  color: var(--foreground);
  background: var(--background);
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Roboto', 'Oxygen';
}

a {
  color: var(--primary-color);
  text-decoration: none;
}

a:hover {
  text-decoration: underline;
}
```

### Styled JSX (Built-in)

```typescript
export default function StyledComponent() {
  return (
    <div className="container">
      <h1>Styled JSX</h1>
      <p>Dies ist ein Beispiel mit Styled JSX.</p>
      
      <style jsx>{`
        .container {
          padding: 20px;
          background-color: #f0f0f0;
          border-radius: 8px;
        }
        
        h1 {
          color: #0070f3;
          font-size: 2rem;
        }
        
        p {
          color: #666;
          line-height: 1.6;
        }
      `}</style>
    </div>
  )
}
```

### CSS-in-JS Libraries

**Emotion Example:**

```bash
npm install @emotion/react @emotion/styled
```

```typescript
/** @jsxImportSource @emotion/react */
import { css } from '@emotion/react'
import styled from '@emotion/styled'

const Button = styled.button`
  background-color: #0070f3;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  
  &:hover {
    background-color: #0051cc;
  }
`

export default function EmotionExample() {
  return (
    <div>
      <Button>Emotion Button</Button>
      
      <div css={css`
        padding: 20px;
        background-color: #f0f0f0;
        margin-top: 10px;
      `}>
        Inline Emotion Styles
      </div>
    </div>
  )
}
```

---

## Layouts und Templates

### Root Layout (Erforderlich)

```typescript
// app/layout.tsx
import type { Metadata } from 'next'
import './globals.css'
import Header from './components/Header'
import Footer from './components/Footer'

export const metadata: Metadata = {
  title: {
    default: 'Meine App',
    template: '%s | Meine App'
  },
  description: 'Eine großartige Next.js Anwendung',
}

export default function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="de">
      <body>
        <Header />
        <main className="min-h-screen">
          {children}
        </main>
        <Footer />
      </body>
    </html>
  )
}
```

### Nested Layouts

```typescript
// app/dashboard/layout.tsx
import Sidebar from '../components/Sidebar'

export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode
}) {
  return (
    <div className="flex">
      <Sidebar />
      <div className="flex-1 p-8">
        {children}
      </div>
    </div>
  )
}
```

**Struktur:**
```
app/
├── layout.tsx              # Root Layout für alle
└── dashboard/
    ├── layout.tsx          # Dashboard Layout
    ├── page.tsx            # /dashboard
    └── einstellungen/
        └── page.tsx        # /dashboard/einstellungen
```

Die Seite `/dashboard/einstellungen` wird von **beiden** Layouts umschlossen:
```
RootLayout
  └── DashboardLayout
      └── EinstellungenPage
```

### Templates

Templates sind ähnlich wie Layouts, erstellen aber bei jeder Navigation eine neue Instanz.

```typescript
// app/template.tsx
'use client'

import { useEffect } from 'react'

export default function Template({ 
  children 
}: { 
  children: React.ReactNode 
}) {
  useEffect(() => {
    console.log('Template wurde gemountet')
  }, [])
  
  return (
    <div className="template-container">
      {children}
    </div>
  )
}
```

**Layout vs Template:**
- **Layout**: Bleibt zwischen Navigationen erhalten (State wird beibehalten)
- **Template**: Wird bei jeder Navigation neu erstellt (State wird zurückgesetzt)

---

## Navigation

### Link Component

Der `<Link>`-Component ist die primäre Methode für Navigation in Next.js.

```typescript
// app/components/Navigation.tsx
import Link from 'next/link'

export default function Navigation() {
  return (
    <nav>
      <Link href="/">Home</Link>
      <Link href="/about">Über uns</Link>
      <Link href="/blog">Blog</Link>
      <Link href="/kontakt">Kontakt</Link>
    </nav>
  )
}
```

### Link mit Styling

```typescript
import Link from 'next/link'

export default function StyledNav() {
  return (
    <nav className="flex gap-4 p-4 bg-gray-800">
      <Link 
        href="/" 
        className="text-white hover:text-blue-400 transition-colors"
      >
        Home
      </Link>
      <Link 
        href="/blog"
        className="text-white hover:text-blue-400 transition-colors"
      >
        Blog
      </Link>
    </nav>
  )
}
```

### Aktive Links

```typescript
'use client'

import Link from 'next/link'
import { usePathname } from 'next/navigation'

export default function ActiveNav() {
  const pathname = usePathname()
  
  const links = [
    { href: '/', label: 'Home' },
    { href: '/about', label: 'Über uns' },
    { href: '/blog', label: 'Blog' },
    { href: '/kontakt', label: 'Kontakt' },
  ]
  
  return (
    <nav className="flex gap-4">
      {links.map(link => (
        <Link
          key={link.href}
          href={link.href}
          className={
            pathname === link.href
              ? 'text-blue-600 font-bold'
              : 'text-gray-600 hover:text-blue-600'
          }
        >
          {link.label}
        </Link>
      ))}
    </nav>
  )
}
```

### Programmatische Navigation

```typescript
'use client'

import { useRouter } from 'next/navigation'

export default function LoginForm() {
  const router = useRouter()
  
  async function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault()
    
    // Login-Logik
    const success = await login()
    
    if (success) {
      router.push('/dashboard')
      // oder: router.replace('/dashboard') - ersetzt History-Eintrag
    }
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <button type="submit">Login</button>
    </form>
  )
}

async function login() {
  return true // Simuliert erfolgreichen Login
}
```

### Navigation mit Parametern

```typescript
import Link from 'next/link'

export default function BlogList() {
  const posts = [
    { id: 1, slug: 'erster-post', title: 'Erster Post' },
    { id: 2, slug: 'zweiter-post', title: 'Zweiter Post' },
  ]
  
  return (
    <div>
      {posts.map(post => (
        <Link 
          key={post.id}
          href={`/blog/${post.slug}`}
        >
          {post.title}
        </Link>
      ))}
    </div>
  )
}
```

### Prefetching

Next.js prefetcht automatisch Links im Viewport.

```typescript
import Link from 'next/link'

export default function Navigation() {
  return (
    <>
      {/* Automatisches Prefetching (Standard) */}
      <Link href="/about">Über uns</Link>
      
      {/* Prefetching deaktivieren */}
      <Link href="/blog" prefetch={false}>
        Blog
      </Link>
    </>
  )
}
```

### Scroll-Verhalten

```typescript
import Link from 'next/link'

export default function ScrollExamples() {
  return (
    <>
      {/* Standard: Scrollt nach oben */}
      <Link href="/page">Normale Navigation</Link>
      
      {/* Behält Scroll-Position bei */}
      <Link href="/page" scroll={false}>
        Navigation ohne Scroll
      </Link>
      
      {/* Scrollt zu einem Element */}
      <Link href="/page#section">
        Zu Section scrollen
      </Link>
    </>
  )
}
```

---

## Metadata und SEO

### Statische Metadata

```typescript
// app/page.tsx
import type { Metadata } from 'next'

export const metadata: Metadata = {
  title: 'Startseite',
  description: 'Willkommen auf unserer Website',
  keywords: ['Next.js', 'React', 'TypeScript'],
  authors: [{ name: 'Max Mustermann' }],
  openGraph: {
    title: 'Meine App',
    description: 'Eine tolle Next.js App',
    images: ['/og-image.jpg'],
  },
  twitter: {
    card: 'summary_large_image',
    title: 'Meine App',
    description: 'Eine tolle Next.js App',
    images: ['/twitter-image.jpg'],
  },
}

export default function HomePage() {
  return <h1>Startseite</h1>
}
```

### Dynamische Metadata

```typescript
// app/blog/[slug]/page.tsx
import type { Metadata } from 'next'

interface PageProps {
  params: {
    slug: string
  }
}

async function getPost(slug: string) {
  // Simulierte Datenbankabfrage
  return {
    title: `Blog Post: ${slug}`,
    content: 'Lorem ipsum...',
    author: 'Max Mustermann',
    publishedAt: '2024-01-15',
  }
}

export async function generateMetadata(
  { params }: PageProps
): Promise<Metadata> {
  const post = await getPost(params.slug)
  
  return {
    title: post.title,
    description: post.content.substring(0, 160),
    authors: [{ name: post.author }],
    openGraph: {
      title: post.title,
      type: 'article',
      publishedTime: post.publishedAt,
    },
  }
}

export default async function BlogPost({ params }: PageProps) {
  const post = await getPost(params.slug)
  
  return (
    <article>
      <h1>{post.title}</h1>
      <p>Von {post.author}</p>
      <div>{post.content}</div>
    </article>
  )
}
```

### JSON-LD für Structured Data

```typescript
// app/page.tsx
export default function HomePage() {
  const jsonLd = {
    '@context': 'https://schema.org',
    '@type': 'Organization',
    name: 'Meine Firma',
    url: 'https://example.com',
    logo: 'https://example.com/logo.png',
    contactPoint: {
      '@type': 'ContactPoint',
      telephone: '+49-123-456789',
      contactType: 'Kundenservice',
    },
  }
  
  return (
    <>
      <script
        type="application/ld+json"
        dangerouslySetInnerHTML={{ __html: JSON.stringify(jsonLd) }}
      />
      <h1>Willkommen</h1>
    </>
  )
}
```

### Sitemap generieren

```typescript
// app/sitemap.ts
import { MetadataRoute } from 'next'

export default function sitemap(): MetadataRoute.Sitemap {
  return [
    {
      url: 'https://example.com',
      lastModified: new Date(),
      changeFrequency: 'daily',
      priority: 1,
    },
    {
      url: 'https://example.com/about',
      lastModified: new Date(),
      changeFrequency: 'monthly',
      priority: 0.8,
    },
    {
      url: 'https://example.com/blog',
      lastModified: new Date(),
      changeFrequency: 'weekly',
      priority: 0.5,
    },
  ]
}
```

### Robots.txt

```typescript
// app/robots.ts
import { MetadataRoute } from 'next'

export default function robots(): MetadataRoute.Robots {
  return {
    rules: {
      userAgent: '*',
      allow: '/',
      disallow: '/private/',
    },
    sitemap: 'https://example.com/sitemap.xml',
  }
}
```

---

## Loading und Error Handling

### Loading States

```typescript
// app/blog/loading.tsx
export default function Loading() {
  return (
    <div className="flex justify-center items-center min-h-screen">
      <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-blue-600" />
      <p className="ml-4">Lädt Blog-Posts...</p>
    </div>
  )
}
```

**Automatisch verwendet während:**
- Daten werden gefetcht
- Seite wird geladen
- Navigation zu neuer Route

### Suspense für granulares Loading

```typescript
// app/dashboard/page.tsx
import { Suspense } from 'react'

async function SlowComponent() {
  await new Promise(resolve => setTimeout(resolve, 3000))
  return <div>Langsame Daten geladen!</div>
}

async function FastComponent() {
  await new Promise(resolve => setTimeout(resolve, 500))
  return <div>Schnelle Daten geladen!</div>
}

export default function Dashboard() {
  return (
    <div>
      <h1>Dashboard</h1>
      
      {/* Lädt schnell */}
      <Suspense fallback={<p>Lädt schnelle Daten...</p>}>
        <FastComponent />
      </Suspense>
      
      {/* Lädt langsam, blockiert aber nicht FastComponent */}
      <Suspense fallback={<p>Lädt langsame Daten...</p>}>
        <SlowComponent />
      </Suspense>
    </div>
  )
}
```

### Error Handling

```typescript
// app/blog/error.tsx
'use client'

export default function Error({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <div className="flex flex-col items-center justify-center min-h-screen">
      <h2 className="text-2xl font-bold text-red-600 mb-4">
        Etwas ist schiefgelaufen!
      </h2>
      <p className="text-gray-600 mb-4">{error.message}</p>
      <button
        onClick={reset}
        className="px-4 py-2 bg-blue-600 text-white rounded hover:bg-blue-700"
      >
        Erneut versuchen
      </button>
    </div>
  )
}
```

### Global Error Handler

```typescript
// app/global-error.tsx
'use client'

export default function GlobalError({
  error,
  reset,
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <html>
      <body>
        <h2>Globaler Fehler!</h2>
        <p>{error.message}</p>
        <button onClick={reset}>Erneut versuchen</button>
      </body>
    </html>
  )
}
```

### Not Found Page

```typescript
// app/not-found.tsx
import Link from 'next/link'

export default function NotFound() {
  return (
    <div className="flex flex-col items-center justify-center min-h-screen">
      <h1 className="text-6xl font-bold text-gray-800 mb-4">404</h1>
      <h2 className="text-2xl text-gray-600 mb-8">Seite nicht gefunden</h2>
      <Link 
        href="/"
        className="px-6 py-3 bg-blue-600 text-white rounded-lg hover:bg-blue-700"
      >
        Zurück zur Startseite
      </Link>
    </div>
  )
}
```

### Programmatisch Not Found auslösen

```typescript
// app/blog/[slug]/page.tsx
import { notFound } from 'next/navigation'

async function getPost(slug: string) {
  const post = await fetchPost(slug)
  return post
}

export default async function BlogPost({ 
  params 
}: { 
  params: { slug: string } 
}) {
  const post = await getPost(params.slug)
  
  if (!post) {
    notFound() // Löst not-found.tsx aus
  }
  
  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  )
}

async function fetchPost(slug: string) {
  // Simuliert DB-Abfrage
  return null // Post nicht gefunden
}
```

---

## Praktische Projekte

### Projekt 1: Blog mit API

Ein vollständiger Blog mit API-Integration.

**Projektstruktur:**
```
app/
├── layout.tsx
├── page.tsx
├── blog/
│   ├── page.tsx
│   ├── [slug]/
│   │   └── page.tsx
│   └── loading.tsx
├── api/
│   └── posts/
│       ├── route.ts
│       └── [id]/
│           └── route.ts
└── components/
    ├── Header.tsx
    ├── PostCard.tsx
    └── PostList.tsx
```

**1. API Routes erstellen:**

```typescript
// app/api/posts/route.ts
import { NextResponse } from 'next/server'

const posts = [
  {
    id: 1,
    slug: 'next-js-einfuehrung',
    title: 'Next.js Einführung',
    excerpt: 'Lerne die Grundlagen von Next.js',
    content: 'Next.js ist ein React-Framework...',
    author: 'Max Mustermann',
    date: '2024-01-15',
  },
  {
    id: 2,
    slug: 'typescript-basics',
    title: 'TypeScript Basics',
    excerpt: 'TypeScript für Anfänger',
    content: 'TypeScript ist eine typisierte Obermenge...',
    author: 'Anna Schmidt',
    date: '2024-01-20',
  },
]

export async function GET() {
  return NextResponse.json(posts)
}
```

```typescript
// app/api/posts/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server'

const posts = [
  {
    id: 1,
    slug: 'next-js-einfuehrung',
    title: 'Next.js Einführung',
    excerpt: 'Lerne die Grundlagen von Next.js',
    content: 'Next.js ist ein React-Framework für Production...',
    author: 'Max Mustermann',
    date: '2024-01-15',
  },
]

export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  const post = posts.find(p => p.id === parseInt(params.id))
  
  if (!post) {
    return NextResponse.json(
      { error: 'Post nicht gefunden' },
      { status: 404 }
    )
  }
  
  return NextResponse.json(post)
}
```

**2. Components erstellen:**

```typescript
// app/components/PostCard.tsx
import Link from 'next/link'

interface PostCardProps {
  post: {
    id: number
    slug: string
    title: string
    excerpt: string
    author: string
    date: string
  }
}

export default function PostCard({ post }: PostCardProps) {
  return (
    <article className="bg-white rounded-lg shadow-md p-6 hover:shadow-lg transition-shadow">
      <h2 className="text-2xl font-bold mb-2">
        <Link 
          href={`/blog/${post.slug}`}
          className="hover:text-blue-600"
        >
          {post.title}
        </Link>
      </h2>
      <p className="text-gray-600 mb-4">{post.excerpt}</p>
      <div className="flex justify-between text-sm text-gray-500">
        <span>Von {post.author}</span>
        <time>{post.date}</time>
      </div>
    </article>
  )
}
```

**3. Blog-Liste Seite:**

```typescript
// app/blog/page.tsx
import PostCard from '../components/PostCard'

async function getPosts() {
  const res = await fetch('http://localhost:3000/api/posts', {
    cache: 'no-store'
  })
  return res.json()
}

export default async function BlogPage() {
  const posts = await getPosts()
  
  return (
    <div className="container mx-auto px-4 py-8">
      <h1 className="text-4xl font-bold mb-8">Blog</h1>
      <div className="grid gap-6 md:grid-cols-2 lg:grid-cols-3">
        {posts.map((post: any) => (
          <PostCard key={post.id} post={post} />
        ))}
      </div>
    </div>
  )
}
```

**4. Einzelner Blog-Post:**

```typescript
// app/blog/[slug]/page.tsx
import { notFound } from 'next/navigation'

async function getPostBySlug(slug: string) {
  const res = await fetch('http://localhost:3000/api/posts')
  const posts = await res.json()
  return posts.find((p: any) => p.slug === slug)
}

export default async function BlogPostPage({
  params
}: {
  params: { slug: string }
}) {
  const post = await getPostBySlug(params.slug)
  
  if (!post) {
    notFound()
  }
  
  return (
    <article className="container mx-auto px-4 py-8 max-w-3xl">
      <h1 className="text-4xl font-bold mb-4">{post.title}</h1>
      <div className="flex gap-4 text-gray-600 mb-8">
        <span>Von {post.author}</span>
        <time>{post.date}</time>
      </div>
      <div className="prose prose-lg">
        {post.content}
      </div>
    </article>
  )
}
```

### Projekt 2: Todo-App mit Server Actions

Eine interaktive Todo-App mit Next.js Server Actions.

```typescript
// app/actions/todos.ts
'use server'

import { revalidatePath } from 'next/cache'

// Simulierte Datenbank
let todos = [
  { id: 1, text: 'Next.js lernen', completed: false },
  { id: 2, text: 'Blog erstellen', completed: false },
]

export async function getTodos() {
  return todos
}

export async function addTodo(text: string) {
  const newTodo = {
    id: Date.now(),
    text,
    completed: false,
  }
  todos.push(newTodo)
  revalidatePath('/todos')
  return newTodo
}

export async function toggleTodo(id: number) {
  const todo = todos.find(t => t.id === id)
  if (todo) {
    todo.completed = !todo.completed
  }
  revalidatePath('/todos')
}

export async function deleteTodo(id: number) {
  todos = todos.filter(t => t.id !== id)
  revalidatePath('/todos')
}
```

```typescript
// app/todos/page.tsx
import { getTodos } from '../actions/todos'
import TodoList from '../components/TodoList'
import AddTodoForm from '../components/AddTodoForm'

export default async function TodosPage() {
  const todos = await getTodos()
  
  return (
    <div className="container mx-auto px-4 py-8 max-w-2xl">
      <h1 className="text-4xl font-bold mb-8">Meine Todos</h1>
      <AddTodoForm />
      <TodoList todos={todos} />
    </div>
  )
}
```

```typescript
// app/components/AddTodoForm.tsx
'use client'

import { addTodo } from '../actions/todos'
import { useTransition } from 'react'

export default function AddTodoForm() {
  const [isPending, startTransition] = useTransition()
  
  async function handleSubmit(formData: FormData) {
    const text = formData.get('text') as string
    if (!text.trim()) return
    
    startTransition(async () => {
      await addTodo(text)
    })
  }
  
  return (
    <form action={handleSubmit} className="mb-6">
      <div className="flex gap-2">
        <input
          type="text"
          name="text"
          placeholder="Neues Todo..."
          className="flex-1 px-4 py-2 border rounded-lg"
          disabled={isPending}
        />
        <button
          type="submit"
          disabled={isPending}
          className="px-6 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 disabled:bg-gray-400"
        >
          {isPending ? 'Wird hinzugefügt...' : 'Hinzufügen'}
        </button>
      </div>
    </form>
  )
}
```

```typescript
// app/components/TodoList.tsx
'use client'

import { toggleTodo, deleteTodo } from '../actions/todos'
import { useTransition } from 'react'

interface Todo {
  id: number
  text: string
  completed: boolean
}

export default function TodoList({ todos }: { todos: Todo[] }) {
  const [isPending, startTransition] = useTransition()
  
  function handleToggle(id: number) {
    startTransition(async () => {
      await toggleTodo(id)
    })
  }
  
  function handleDelete(id: number) {
    startTransition(async () => {
      await deleteTodo(id)
    })
  }
  
  return (
    <ul className="space-y-2">
      {todos.map(todo => (
        <li
          key={todo.id}
          className="flex items-center gap-3 p-4 bg-white rounded-lg shadow"
        >
          <input
            type="checkbox"
            checked={todo.completed}
            onChange={() => handleToggle(todo.id)}
            className="w-5 h-5"
            disabled={isPending}
          />
          <span
            className={`flex-1 ${
              todo.completed ? 'line-through text-gray-500' : ''
            }`}
          >
            {todo.text}
          </span>
          <button
            onClick={() => handleDelete(todo.id)}
            disabled={isPending}
            className="px-3 py-1 text-red-600 hover:bg-red-50 rounded"
          >
            Löschen
          </button>
        </li>
      ))}
    </ul>
  )
}
```

---

## Zusammenfassung

Du hast nun die Grundlagen von Next.js gelernt:

✅ **Setup**: Projekt erstellen und konfigurieren
✅ **Routing**: File-based Routing, dynamische Routes, Catch-all Routes
✅ **Components**: Server vs Client Components
✅ **Data Fetching**: Server-side und Client-side
✅ **API Routes**: REST APIs direkt im Projekt
✅ **Styling**: CSS Modules, Tailwind, Styled JSX
✅ **Navigation**: Link Component, programmatische Navigation
✅ **Metadata**: SEO-Optimierung
✅ **Error Handling**: Loading States, Error Boundaries

### Nächste Schritte

1. **Datenbanken integrieren**: PostgreSQL, MongoDB, Prisma
2. **Authentication**: NextAuth.js für User-Management
3. **Deployment**: Vercel, AWS, Docker
4. **Advanced Features**: Middleware, Internationalisierung, Image Optimization
5. **Performance**: Caching-Strategien, ISR, Edge Functions

### Hilfreiche Ressourcen

- [Next.js Dokumentation](https://nextjs.org/docs)
- [Next.js Examples](https://github.com/vercel/next.js/tree/canary/examples)
- [Vercel Templates](https://vercel.com/templates/next.js)
- [Next.js Blog](https://nextjs.org/blog)

Viel Erfolg mit Next.js! 🚀
