# SQL & Datenbanken - Kompletter Walkthrough

## Inhaltsverzeichnis
1. [Einführung](#einführung)
2. [Datenbank-Grundlagen](#datenbank-grundlagen)
3. [SQL Basics](#sql-basics)
4. [PostgreSQL Setup](#postgresql-setup)
5. [CRUD-Operationen](#crud-operationen)
6. [Datentypen](#datentypen)
7. [Constraints und Keys](#constraints-und-keys)
8. [Beziehungen (Relationships)](#beziehungen-relationships)
9. [JOINs](#joins)
10. [Aggregatfunktionen](#aggregatfunktionen)
11. [Subqueries](#subqueries)
12. [Indexes und Performance](#indexes-und-performance)
13. [Transaktionen](#transaktionen)
14. [Prisma ORM](#prisma-orm)
15. [Praktische Projekte](#praktische-projekte)

---

## Einführung

### Was sind Datenbanken?

**Datenbanken** sind strukturierte Sammlungen von Daten, die effizient gespeichert, abgerufen und verwaltet werden können.

### Arten von Datenbanken

#### Relationale Datenbanken (SQL)
- **PostgreSQL** (empfohlen)
- MySQL
- SQLite
- Microsoft SQL Server
- Oracle

#### NoSQL-Datenbanken
- MongoDB (Document Store)
- Redis (Key-Value Store)
- Cassandra (Wide Column Store)
- Neo4j (Graph Database)

### SQL vs NoSQL

| Aspekt | SQL (Relational) | NoSQL |
|--------|------------------|-------|
| Struktur | Tabellen, Zeilen, Spalten | Dokumente, Key-Value, etc. |
| Schema | Fest definiert | Flexibel |
| Skalierung | Vertikal (besser Hardware) | Horizontal (mehr Server) |
| ACID | Ja | Teilweise |
| Beziehungen | Sehr gut | Eingeschränkt |
| Use Cases | Finanzen, E-Commerce | Social Media, IoT |

### Wann SQL verwenden?

✅ Strukturierte Daten mit klaren Beziehungen
✅ Datenintegrität ist kritisch
✅ Komplexe Abfragen nötig
✅ Transaktionen erforderlich
✅ Business-Anwendungen

---

## Datenbank-Grundlagen

### Wichtige Konzepte

#### Tabelle (Table)
Eine Sammlung verwandter Daten in Zeilen und Spalten.

```
Tabelle: users
┌────┬────────────┬──────────────────────┬─────┐
│ id │ name       │ email                │ age │
├────┼────────────┼──────────────────────┼─────┤
│ 1  │ Max        │ max@example.com      │ 30  │
│ 2  │ Anna       │ anna@example.com     │ 25  │
│ 3  │ Tom        │ tom@example.com      │ 35  │
└────┴────────────┴──────────────────────┴─────┘
```

#### Zeile (Row/Record)
Ein einzelner Datensatz in einer Tabelle.

#### Spalte (Column/Field)
Ein Attribut oder eine Eigenschaft in der Tabelle.

#### Primary Key
Eindeutiger Bezeichner für jede Zeile (z.B. `id`).

#### Foreign Key
Verknüpfung zu einem Primary Key in einer anderen Tabelle.

### Entity-Relationship-Modell (ER-Modell)

```
┌─────────────┐         ┌─────────────┐
│   users     │         │   posts     │
├─────────────┤         ├─────────────┤
│ id (PK)     │─────────│ id (PK)     │
│ name        │    1:N  │ user_id (FK)│
│ email       │         │ title       │
└─────────────┘         │ content     │
                        └─────────────┘
```

### Normalisierung

Datenbank-Design-Technik zur Minimierung von Redundanz.

#### 1. Normalform (1NF)
- Atomare Werte (keine Listen in Feldern)
- Eindeutige Zeilen

```sql
-- ❌ Nicht 1NF
CREATE TABLE users (
    id INT,
    name TEXT,
    hobbies TEXT  -- "Lesen, Sport, Musik" (Liste!)
);

-- ✅ 1NF
CREATE TABLE users (
    id INT,
    name TEXT
);

CREATE TABLE hobbies (
    user_id INT,
    hobby TEXT
);
```

#### 2. Normalform (2NF)
- Erfüllt 1NF
- Keine partiellen Abhängigkeiten

#### 3. Normalform (3NF)
- Erfüllt 2NF
- Keine transitiven Abhängigkeiten

---

## SQL Basics

### SQL-Syntax-Grundlagen

```sql
-- Kommentare mit --
/* Oder mit
   mehrzeiligen Kommentaren */

-- SQL ist case-insensitive (meistens)
SELECT * FROM users;
select * from users;  -- Gleich wie oben

-- Aber: Keywords GROSSSCHREIBEN ist Convention
SELECT name FROM users;

-- Semikolon am Ende (oft optional, aber Best Practice)
SELECT * FROM users;
```

### SQL-Kategorien

#### DDL (Data Definition Language)
Schema definieren: `CREATE`, `ALTER`, `DROP`

#### DML (Data Manipulation Language)
Daten manipulieren: `SELECT`, `INSERT`, `UPDATE`, `DELETE`

#### DCL (Data Control Language)
Zugriffsrechte: `GRANT`, `REVOKE`

#### TCL (Transaction Control Language)
Transaktionen: `COMMIT`, `ROLLBACK`, `SAVEPOINT`

---

## PostgreSQL Setup

### Installation

#### Windows
```powershell
# Mit Chocolatey
choco install postgresql

# Oder: Installer von https://www.postgresql.org/download/windows/
```

#### macOS
```bash
# Mit Homebrew
brew install postgresql@16
brew services start postgresql@16
```

#### Linux (Ubuntu/Debian)
```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

### Docker (Empfohlen für Entwicklung)

```bash
# PostgreSQL Container starten
docker run --name postgres-dev \
  -e POSTGRES_PASSWORD=mypassword \
  -e POSTGRES_USER=myuser \
  -e POSTGRES_DB=mydb \
  -p 5432:5432 \
  -d postgres:16

# Container stoppen
docker stop postgres-dev

# Container starten
docker start postgres-dev

# Container entfernen
docker rm postgres-dev
```

**docker-compose.yml:**

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:16
    container_name: postgres-dev
    environment:
      POSTGRES_USER: myuser
      POSTGRES_PASSWORD: mypassword
      POSTGRES_DB: mydb
    ports:
      - "5432:5432"
    volumes:
      - postgres-data:/var/lib/postgresql/data

volumes:
  postgres-data:
```

```bash
# Starten
docker-compose up -d

# Stoppen
docker-compose down
```

### PostgreSQL-Client (psql)

```bash
# Verbinden
psql -U myuser -d mydb -h localhost

# Mit Docker
docker exec -it postgres-dev psql -U myuser -d mydb

# Wichtige psql-Befehle
\l              # Datenbanken auflisten
\c dbname       # Datenbank wechseln
\dt             # Tabellen auflisten
\d tablename    # Tabellen-Schema anzeigen
\du             # Benutzer auflisten
\q              # Beenden
\h              # SQL-Hilfe
\?              # psql-Hilfe
```

### GUI-Tools

- **pgAdmin** - Offizielles PostgreSQL-Tool
- **DBeaver** - Universal Database Tool
- **TablePlus** - Modern, sleek (macOS/Windows)
- **DataGrip** - JetBrains (kostenpflichtig)

### VS Code Extensions

- **PostgreSQL** von Chris Kolkman
- **SQLTools** mit PostgreSQL Driver

---

## CRUD-Operationen

### Datenbank erstellen

```sql
-- Datenbank erstellen
CREATE DATABASE shop;

-- Datenbank löschen
DROP DATABASE shop;

-- Wenn vorhanden, löschen und neu erstellen
DROP DATABASE IF EXISTS shop;
CREATE DATABASE shop;

-- Zu Datenbank wechseln (psql)
\c shop
```

### Tabelle erstellen (CREATE)

```sql
-- Einfache Tabelle
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(255),
    age INTEGER
);

-- Mit mehr Constraints
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    age INTEGER CHECK (age >= 0),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Daten einfügen (INSERT)

```sql
-- Einzelne Zeile einfügen
INSERT INTO users (name, email, age)
VALUES ('Max Mustermann', 'max@example.com', 30);

-- Mehrere Zeilen einfügen
INSERT INTO users (name, email, age)
VALUES 
    ('Anna Schmidt', 'anna@example.com', 25),
    ('Tom Müller', 'tom@example.com', 35),
    ('Lisa Weber', 'lisa@example.com', 28);

-- Alle Spalten (in Reihenfolge)
INSERT INTO users
VALUES (DEFAULT, 'Peter Pan', 'peter@example.com', 22, DEFAULT);

-- Mit RETURNING (gibt eingefügte Daten zurück)
INSERT INTO users (name, email, age)
VALUES ('Sarah Klein', 'sarah@example.com', 27)
RETURNING *;

-- Oder nur bestimmte Felder
RETURNING id, name;
```

### Daten abfragen (SELECT)

```sql
-- Alle Spalten, alle Zeilen
SELECT * FROM users;

-- Bestimmte Spalten
SELECT name, email FROM users;

-- Mit WHERE-Bedingung
SELECT * FROM users
WHERE age > 25;

-- Mehrere Bedingungen
SELECT * FROM users
WHERE age > 25 AND name LIKE 'A%';

-- ORDER BY (Sortieren)
SELECT * FROM users
ORDER BY age DESC;  -- DESC = absteigend, ASC = aufsteigend

-- LIMIT (Anzahl begrenzen)
SELECT * FROM users
LIMIT 5;

-- OFFSET (Überspringen)
SELECT * FROM users
OFFSET 5 LIMIT 5;  -- Zeilen 6-10

-- DISTINCT (Duplikate entfernen)
SELECT DISTINCT age FROM users;

-- Alias (AS)
SELECT name AS benutzername, email AS emailadresse
FROM users AS u;
```

### Daten aktualisieren (UPDATE)

```sql
-- Alle Zeilen aktualisieren (VORSICHT!)
UPDATE users
SET age = 30;

-- Mit WHERE-Bedingung
UPDATE users
SET age = 31
WHERE id = 1;

-- Mehrere Spalten
UPDATE users
SET 
    name = 'Max Müller',
    email = 'max.mueller@example.com'
WHERE id = 1;

-- Mit RETURNING
UPDATE users
SET age = age + 1
WHERE id = 1
RETURNING *;
```

### Daten löschen (DELETE)

```sql
-- Alle Zeilen löschen (VORSICHT!)
DELETE FROM users;

-- Mit WHERE-Bedingung
DELETE FROM users
WHERE id = 1;

-- Mehrere Bedingungen
DELETE FROM users
WHERE age < 18 OR email IS NULL;

-- Mit RETURNING
DELETE FROM users
WHERE id = 1
RETURNING *;

-- Tabelle leeren (schneller als DELETE)
TRUNCATE TABLE users;

-- Mit CASCADE (löscht auch abhängige Zeilen)
TRUNCATE TABLE users CASCADE;
```

---

## Datentypen

### Numerische Typen

```sql
CREATE TABLE numbers (
    -- Ganzzahlen
    tiny_int SMALLINT,           -- -32768 bis 32767
    normal_int INTEGER,          -- -2 Milliarden bis 2 Milliarden
    big_int BIGINT,              -- Sehr große Zahlen
    
    -- Auto-Increment
    id SERIAL,                   -- INTEGER mit AUTO_INCREMENT
    big_id BIGSERIAL,            -- BIGINT mit AUTO_INCREMENT
    
    -- Dezimalzahlen
    precise_num NUMERIC(10, 2),  -- Genau: 10 Stellen, 2 Dezimalen
    price DECIMAL(8, 2),         -- Synonym für NUMERIC
    
    -- Fließkommazahlen
    small_float REAL,            -- 6 Dezimalstellen Präzision
    double_float DOUBLE PRECISION -- 15 Dezimalstellen Präzision
);
```

### Text-Typen

```sql
CREATE TABLE texts (
    -- Variable Länge
    short_text VARCHAR(50),      -- Max 50 Zeichen
    long_text VARCHAR(1000),     -- Max 1000 Zeichen
    
    -- Feste Länge
    fixed_text CHAR(10),         -- Genau 10 Zeichen (mit Padding)
    
    -- Unbegrenzt
    unlimited_text TEXT,         -- Beliebig lang
    
    -- Beispiele
    name VARCHAR(100) NOT NULL,
    description TEXT,
    code CHAR(5)
);
```

### Datums- und Zeit-Typen

```sql
CREATE TABLE dates (
    -- Datum
    birth_date DATE,                           -- '2024-01-15'
    
    -- Zeit
    meeting_time TIME,                         -- '14:30:00'
    meeting_time_tz TIME WITH TIME ZONE,       -- '14:30:00+01'
    
    -- Datum und Zeit
    created_at TIMESTAMP,                      -- '2024-01-15 14:30:00'
    updated_at TIMESTAMP WITH TIME ZONE,       -- '2024-01-15 14:30:00+01'
    
    -- Zeitintervall
    duration INTERVAL,                         -- '2 days 3 hours'
    
    -- Default-Werte
    created TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    modified TIMESTAMP DEFAULT NOW()
);

-- Beispiel-Inserts
INSERT INTO dates (birth_date, meeting_time, created_at)
VALUES 
    ('1990-05-15', '14:30:00', '2024-01-15 14:30:00'),
    (CURRENT_DATE, CURRENT_TIME, NOW());
```

### Boolean

```sql
CREATE TABLE flags (
    is_active BOOLEAN DEFAULT TRUE,
    is_verified BOOLEAN DEFAULT FALSE,
    has_permission BOOLEAN
);

-- Werte einfügen
INSERT INTO flags (is_active, is_verified)
VALUES 
    (TRUE, FALSE),
    (true, false),    -- Case-insensitive
    ('yes', 'no'),    -- Auch möglich
    (1, 0);           -- Auch möglich
```

### JSON

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    
    -- JSON (Text-basiert)
    metadata JSON,
    
    -- JSONB (Binary, schneller, indexierbar)
    details JSONB
);

-- JSON einfügen
INSERT INTO products (name, details)
VALUES (
    'Laptop',
    '{"brand": "Dell", "ram": "16GB", "storage": "512GB"}'::JSONB
);

-- JSON abfragen
SELECT 
    name,
    details->>'brand' AS brand,
    details->>'ram' AS ram
FROM products;

-- JSON-Feld aktualisieren
UPDATE products
SET details = details || '{"price": 999}'::JSONB
WHERE id = 1;

-- In JSON-Array suchen
SELECT * FROM products
WHERE details @> '{"brand": "Dell"}';
```

### Arrays

```sql
CREATE TABLE tags_example (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    tags TEXT[]
);

-- Array einfügen
INSERT INTO tags_example (name, tags)
VALUES 
    ('Post 1', ARRAY['javascript', 'typescript', 'react']),
    ('Post 2', '{"nodejs", "express", "mongodb"}');

-- Array abfragen
SELECT * FROM tags_example
WHERE 'javascript' = ANY(tags);

-- Array-Funktionen
SELECT 
    name,
    array_length(tags, 1) AS tag_count,
    tags[1] AS first_tag
FROM tags_example;
```

### Enum (Aufzählungstyp)

```sql
-- Enum erstellen
CREATE TYPE user_role AS ENUM ('admin', 'user', 'guest');

CREATE TABLE users_with_roles (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    role user_role DEFAULT 'user'
);

-- Einfügen
INSERT INTO users_with_roles (name, role)
VALUES 
    ('Admin User', 'admin'),
    ('Normal User', 'user'),
    ('Guest User', 'guest');

-- Enum erweitern
ALTER TYPE user_role ADD VALUE 'moderator';
```

---

## Constraints und Keys

### PRIMARY KEY

```sql
-- Einfacher Primary Key
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

-- Oder nachträglich
CREATE TABLE users (
    id SERIAL,
    name VARCHAR(100),
    PRIMARY KEY (id)
);

-- Composite Primary Key (mehrere Spalten)
CREATE TABLE enrollments (
    student_id INTEGER,
    course_id INTEGER,
    enrolled_at TIMESTAMP DEFAULT NOW(),
    PRIMARY KEY (student_id, course_id)
);
```

### FOREIGN KEY

```sql
-- Benutzer-Tabelle
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

-- Posts-Tabelle mit Foreign Key
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    content TEXT,
    user_id INTEGER REFERENCES users(id)
);

-- Mit ON DELETE und ON UPDATE
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    user_id INTEGER,
    FOREIGN KEY (user_id) REFERENCES users(id)
        ON DELETE CASCADE      -- Löscht Posts wenn User gelöscht wird
        ON UPDATE CASCADE      -- Aktualisiert Posts wenn User-ID ändert
);

-- ON DELETE Optionen:
-- CASCADE      - Abhängige Zeilen löschen
-- SET NULL     - user_id auf NULL setzen
-- SET DEFAULT  - user_id auf Default-Wert setzen
-- RESTRICT     - Löschen verhindern (Standard)
-- NO ACTION    - Wie RESTRICT
```

### UNIQUE

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE,           -- Einzelne Spalte
    username VARCHAR(50) UNIQUE NOT NULL
);

-- Composite Unique (Kombination muss einzigartig sein)
CREATE TABLE user_settings (
    user_id INTEGER,
    setting_key VARCHAR(50),
    setting_value TEXT,
    UNIQUE (user_id, setting_key)
);
```

### NOT NULL

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    price NUMERIC(10, 2) NOT NULL,
    description TEXT  -- Kann NULL sein
);
```

### CHECK

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    price NUMERIC(10, 2) CHECK (price > 0),
    quantity INTEGER CHECK (quantity >= 0),
    email VARCHAR(255) CHECK (email LIKE '%@%'),
    age INTEGER CHECK (age >= 18 AND age <= 100)
);

-- Benannte Constraints
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    age INTEGER,
    CONSTRAINT valid_age CHECK (age >= 0 AND age <= 150)
);
```

### DEFAULT

```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    status VARCHAR(20) DEFAULT 'pending',
    quantity INTEGER DEFAULT 1,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE
);
```

### Constraints nachträglich hinzufügen

```sql
-- Primary Key
ALTER TABLE users
ADD PRIMARY KEY (id);

-- Foreign Key
ALTER TABLE posts
ADD FOREIGN KEY (user_id) REFERENCES users(id);

-- Unique
ALTER TABLE users
ADD UNIQUE (email);

-- Check
ALTER TABLE products
ADD CHECK (price > 0);

-- Constraint löschen
ALTER TABLE users
DROP CONSTRAINT users_email_key;
```

---

## Beziehungen (Relationships)

### One-to-One (1:1)

Ein User hat genau ein Profil.

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL
);

CREATE TABLE profiles (
    id SERIAL PRIMARY KEY,
    user_id INTEGER UNIQUE NOT NULL,  -- UNIQUE macht es 1:1
    bio TEXT,
    avatar_url VARCHAR(255),
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

### One-to-Many (1:N)

Ein User hat viele Posts.

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    content TEXT,
    user_id INTEGER NOT NULL,
    FOREIGN KEY (user_id) REFERENCES users(id) ON DELETE CASCADE
);
```

### Many-to-Many (N:M)

Studenten können viele Kurse belegen, Kurse haben viele Studenten.

```sql
-- Studenten-Tabelle
CREATE TABLE students (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

-- Kurse-Tabelle
CREATE TABLE courses (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200)
);

-- Junction/Bridge/Join-Tabelle
CREATE TABLE enrollments (
    student_id INTEGER,
    course_id INTEGER,
    enrolled_at TIMESTAMP DEFAULT NOW(),
    grade VARCHAR(2),
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(id) ON DELETE CASCADE,
    FOREIGN KEY (course_id) REFERENCES courses(id) ON DELETE CASCADE
);
```

### Self-Referencing (Rekursiv)

Mitarbeiter haben einen Vorgesetzten (auch ein Mitarbeiter).

```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    manager_id INTEGER,
    FOREIGN KEY (manager_id) REFERENCES employees(id)
);

-- Beispiel-Daten
INSERT INTO employees (id, name, manager_id) VALUES
    (1, 'CEO', NULL),           -- Kein Manager
    (2, 'CTO', 1),              -- Manager ist CEO
    (3, 'Dev Lead', 2),         -- Manager ist CTO
    (4, 'Developer 1', 3),      -- Manager ist Dev Lead
    (5, 'Developer 2', 3);      -- Manager ist Dev Lead

-- Alle Mitarbeiter mit ihrem Manager
SELECT 
    e.name AS employee,
    m.name AS manager
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.id;
```

---

## JOINs

### INNER JOIN

Gibt nur Zeilen zurück, die in beiden Tabellen übereinstimmen.

```sql
-- Setup
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    user_id INTEGER REFERENCES users(id)
);

INSERT INTO users (name) VALUES ('Max'), ('Anna'), ('Tom');
INSERT INTO posts (title, user_id) VALUES 
    ('Post 1', 1),
    ('Post 2', 1),
    ('Post 3', 2);

-- INNER JOIN
SELECT 
    users.name,
    posts.title
FROM users
INNER JOIN posts ON users.id = posts.user_id;

-- Ergebnis:
-- Max  | Post 1
-- Max  | Post 2
-- Anna | Post 3
-- (Tom hat keine Posts, wird nicht angezeigt)
```

### LEFT JOIN (LEFT OUTER JOIN)

Alle Zeilen aus der linken Tabelle, auch wenn keine Übereinstimmung rechts.

```sql
SELECT 
    users.name,
    posts.title
FROM users
LEFT JOIN posts ON users.id = posts.user_id;

-- Ergebnis:
-- Max  | Post 1
-- Max  | Post 2
-- Anna | Post 3
-- Tom  | NULL    (Tom hat keine Posts, wird trotzdem angezeigt)
```

### RIGHT JOIN (RIGHT OUTER JOIN)

Alle Zeilen aus der rechten Tabelle.

```sql
SELECT 
    users.name,
    posts.title
FROM users
RIGHT JOIN posts ON users.id = posts.user_id;

-- Gibt alle Posts zurück, auch wenn User nicht existiert
```

### FULL OUTER JOIN

Alle Zeilen aus beiden Tabellen.

```sql
SELECT 
    users.name,
    posts.title
FROM users
FULL OUTER JOIN posts ON users.id = posts.user_id;
```

### CROSS JOIN

Kartesisches Produkt (jede Zeile mit jeder Zeile).

```sql
SELECT 
    users.name,
    posts.title
FROM users
CROSS JOIN posts;

-- Wenn 3 Users und 3 Posts: 9 Ergebnisse (3 x 3)
```

### Multiple JOINs

```sql
CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50)
);

CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200),
    user_id INTEGER REFERENCES users(id),
    category_id INTEGER REFERENCES categories(id)
);

-- Drei Tabellen joinen
SELECT 
    users.name AS author,
    posts.title,
    categories.name AS category
FROM posts
INNER JOIN users ON posts.user_id = users.id
INNER JOIN categories ON posts.category_id = categories.id;
```

### Table Aliases

```sql
-- Kurze Aliases für bessere Lesbarkeit
SELECT 
    u.name,
    p.title,
    c.name
FROM posts p
INNER JOIN users u ON p.user_id = u.id
INNER JOIN categories c ON p.category_id = c.id
WHERE u.name = 'Max';
```

### JOIN mit WHERE

```sql
-- Posts von Usern die "Max" heißen
SELECT 
    u.name,
    p.title
FROM users u
INNER JOIN posts p ON u.id = p.user_id
WHERE u.name = 'Max';

-- Posts, die nach einem bestimmten Datum erstellt wurden
SELECT 
    u.name,
    p.title,
    p.created_at
FROM users u
INNER JOIN posts p ON u.id = p.user_id
WHERE p.created_at > '2024-01-01';
```

---

## Aggregatfunktionen

### COUNT

```sql
-- Alle Zeilen zählen
SELECT COUNT(*) FROM users;

-- Nicht-NULL Werte zählen
SELECT COUNT(email) FROM users;

-- DISTINCT Werte zählen
SELECT COUNT(DISTINCT age) FROM users;

-- Mit GROUP BY
SELECT age, COUNT(*) AS count
FROM users
GROUP BY age;
```

### SUM

```sql
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER,
    amount NUMERIC(10, 2)
);

-- Summe aller Bestellungen
SELECT SUM(amount) AS total FROM orders;

-- Summe pro User
SELECT 
    user_id,
    SUM(amount) AS total_spent
FROM orders
GROUP BY user_id;
```

### AVG

```sql
-- Durchschnittsalter
SELECT AVG(age) FROM users;

-- Durchschnitt gerundet
SELECT ROUND(AVG(age), 2) FROM users;

-- Pro Gruppe
SELECT 
    department,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department;
```

### MIN und MAX

```sql
-- Ältester und jüngster User
SELECT 
    MIN(age) AS youngest,
    MAX(age) AS oldest
FROM users;

-- Günstigster und teuerster Preis
SELECT 
    MIN(price) AS cheapest,
    MAX(price) AS most_expensive
FROM products;
```

### GROUP BY

```sql
-- Anzahl Posts pro User
SELECT 
    user_id,
    COUNT(*) AS post_count
FROM posts
GROUP BY user_id;

-- Mit JOIN für Namen
SELECT 
    u.name,
    COUNT(p.id) AS post_count
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
GROUP BY u.id, u.name
ORDER BY post_count DESC;

-- Mehrere Spalten gruppieren
SELECT 
    category_id,
    user_id,
    COUNT(*) AS count
FROM posts
GROUP BY category_id, user_id;
```

### HAVING

HAVING ist wie WHERE, aber für Aggregate nach GROUP BY.

```sql
-- Users mit mehr als 5 Posts
SELECT 
    user_id,
    COUNT(*) AS post_count
FROM posts
GROUP BY user_id
HAVING COUNT(*) > 5;

-- Durchschnittsgehalt > 50000 pro Department
SELECT 
    department,
    AVG(salary) AS avg_salary
FROM employees
GROUP BY department
HAVING AVG(salary) > 50000;
```

### String-Aggregation

```sql
-- Alle Tags zu einem String
SELECT 
    post_id,
    STRING_AGG(tag, ', ') AS tags
FROM post_tags
GROUP BY post_id;

-- Mit ORDER BY innerhalb
SELECT 
    post_id,
    STRING_AGG(tag, ', ' ORDER BY tag) AS tags
FROM post_tags
GROUP BY post_id;
```

### Komplexes Beispiel

```sql
-- Top 5 Users mit den meisten Posts, durchschnittliche Likes
SELECT 
    u.name,
    COUNT(p.id) AS post_count,
    ROUND(AVG(p.likes), 2) AS avg_likes,
    SUM(p.views) AS total_views,
    MAX(p.created_at) AS last_post_date
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
GROUP BY u.id, u.name
HAVING COUNT(p.id) > 0
ORDER BY post_count DESC, avg_likes DESC
LIMIT 5;
```

---

## Subqueries

### Subquery in WHERE

```sql
-- Users die Posts haben
SELECT * FROM users
WHERE id IN (
    SELECT DISTINCT user_id FROM posts
);

-- Posts vom User mit den meisten Posts
SELECT * FROM posts
WHERE user_id = (
    SELECT user_id
    FROM posts
    GROUP BY user_id
    ORDER BY COUNT(*) DESC
    LIMIT 1
);
```

### Subquery in SELECT

```sql
-- Jeder User mit Anzahl seiner Posts
SELECT 
    u.name,
    (SELECT COUNT(*) FROM posts p WHERE p.user_id = u.id) AS post_count
FROM users u;
```

### Subquery in FROM

```sql
-- Durchschnitt der durchschnittlichen Post-Likes pro User
SELECT AVG(avg_likes) AS global_avg
FROM (
    SELECT 
        user_id,
        AVG(likes) AS avg_likes
    FROM posts
    GROUP BY user_id
) AS user_averages;
```

### Correlated Subquery

```sql
-- Posts die mehr Likes haben als der Durchschnitt des Users
SELECT p.*
FROM posts p
WHERE p.likes > (
    SELECT AVG(likes)
    FROM posts
    WHERE user_id = p.user_id
);
```

### EXISTS

```sql
-- Users die mindestens einen Post haben
SELECT * FROM users u
WHERE EXISTS (
    SELECT 1 FROM posts p WHERE p.user_id = u.id
);

-- NOT EXISTS für Users ohne Posts
SELECT * FROM users u
WHERE NOT EXISTS (
    SELECT 1 FROM posts p WHERE p.user_id = u.id
);
```

---

## Indexes und Performance

### Was sind Indexes?

Indexes sind Datenstrukturen, die Abfragen beschleunigen (wie ein Buch-Index).

**Trade-off:**
- ✅ Schnellere SELECT-Queries
- ❌ Langsamere INSERT/UPDATE/DELETE
- ❌ Mehr Speicherplatz

### Index erstellen

```sql
-- Einfacher Index
CREATE INDEX idx_users_email ON users(email);

-- Unique Index
CREATE UNIQUE INDEX idx_users_username ON users(username);

-- Composite Index (mehrere Spalten)
CREATE INDEX idx_posts_user_created ON posts(user_id, created_at);

-- Partial Index (nur bestimmte Zeilen)
CREATE INDEX idx_active_users ON users(email)
WHERE is_active = true;

-- Expression Index
CREATE INDEX idx_users_lower_email ON users(LOWER(email));
```

### Index löschen

```sql
DROP INDEX idx_users_email;
```

### Wann Indexes verwenden?

```sql
-- ✅ GUTE Kandidaten für Indexes:
-- - Foreign Keys
-- - WHERE-Bedingungen die oft verwendet werden
-- - JOIN-Spalten
-- - ORDER BY-Spalten
-- - Columns mit hoher Kardinalität (viele unique Werte)

CREATE INDEX idx_posts_user_id ON posts(user_id);  -- Foreign Key
CREATE INDEX idx_posts_created ON posts(created_at);  -- Oft für Sortierung

-- ❌ SCHLECHTE Kandidaten:
-- - Kleine Tabellen (< 1000 Zeilen)
-- - Spalten mit wenigen unique Werten (z.B. boolean)
-- - Spalten die selten in Queries verwendet werden
```

### Query-Performance analysieren

```sql
-- EXPLAIN zeigt Query-Plan
EXPLAIN SELECT * FROM users WHERE email = 'max@example.com';

-- EXPLAIN ANALYZE führt Query aus und zeigt echte Zeiten
EXPLAIN ANALYZE
SELECT u.name, COUNT(p.id)
FROM users u
LEFT JOIN posts p ON u.id = p.user_id
GROUP BY u.id, u.name;

-- Ausgabe interpretieren:
-- - Seq Scan = Table Scan (langsam, ohne Index)
-- - Index Scan = Index wird verwendet (schnell)
-- - Cost = Geschätzte Kosten
-- - Execution Time = Tatsächliche Zeit
```

### Performance-Optimierungen

```sql
-- ❌ Langsam: LIKE mit führendem Wildcard
SELECT * FROM users WHERE email LIKE '%@example.com';

-- ✅ Schneller: LIKE ohne führendes Wildcard
SELECT * FROM users WHERE email LIKE 'max@%';

-- ❌ Langsam: Funktion auf indexierter Spalte
SELECT * FROM users WHERE UPPER(email) = 'MAX@EXAMPLE.COM';

-- ✅ Schneller: Expression Index erstellen
CREATE INDEX idx_users_upper_email ON users(UPPER(email));

-- ❌ Langsam: SELECT *
SELECT * FROM users;

-- ✅ Schneller: Nur benötigte Spalten
SELECT id, name, email FROM users;

-- ❌ Langsam: N+1 Query Problem
-- Für jeden User separate Query für Posts

-- ✅ Schneller: JOIN verwenden
SELECT u.name, p.title
FROM users u
LEFT JOIN posts p ON u.id = p.user_id;
```

### Vacuum und Analyze

```sql
-- VACUUM: Aufräumen gelöschter Zeilen
VACUUM users;

-- ANALYZE: Statistiken aktualisieren für Query Planer
ANALYZE users;

-- Beides zusammen
VACUUM ANALYZE users;

-- Für gesamte Datenbank
VACUUM ANALYZE;
```

---

## Transaktionen

### Was sind Transaktionen?

Eine Transaktion ist eine Gruppe von SQL-Statements, die als atomare Einheit ausgeführt werden.

**ACID-Eigenschaften:**
- **Atomicity**: Alles oder nichts
- **Consistency**: Datenbank bleibt konsistent
- **Isolation**: Transaktionen beeinflussen sich nicht
- **Durability**: Änderungen sind dauerhaft

### Basis-Transaktion

```sql
-- Transaktion starten
BEGIN;

-- SQL-Statements
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- Transaktion abschließen
COMMIT;

-- Oder bei Fehler rückgängig machen
ROLLBACK;
```

### Beispiel: Geldüberweisung

```sql
CREATE TABLE accounts (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    balance NUMERIC(10, 2) CHECK (balance >= 0)
);

INSERT INTO accounts (name, balance) VALUES 
    ('Alice', 1000),
    ('Bob', 500);

-- Transaktion: 100€ von Alice zu Bob
BEGIN;

-- Prüfen ob genug Guthaben
SELECT balance FROM accounts WHERE id = 1;

-- Überweisung
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;

-- Wenn alles OK:
COMMIT;

-- Bei Fehler:
-- ROLLBACK;

-- Ergebnis überprüfen
SELECT * FROM accounts;
```

### Savepoints

```sql
BEGIN;

UPDATE products SET price = price * 1.1;  -- Preiserhöhung 10%

SAVEPOINT preis_erhoehung;

UPDATE products SET stock = 0 WHERE id = 5;  -- Fehler!

-- Nur bis Savepoint zurückrollen
ROLLBACK TO SAVEPOINT preis_erhoehung;

-- Preiserhöhung bleibt, Stock-Update wird verworfen

COMMIT;
```

### Isolation Levels

```sql
-- Read Uncommitted (schwächste Isolation)
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

-- Read Committed (PostgreSQL Standard)
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- Repeatable Read
SET TRANSACTION ISOLATION LEVEL REPEATABLE READ;

-- Serializable (stärkste Isolation)
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```

### Locking

```sql
-- Row-Level Lock (nur diese Zeilen)
BEGIN;
SELECT * FROM accounts WHERE id = 1 FOR UPDATE;
-- Andere Transaktionen müssen warten
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;

-- Table-Level Lock
BEGIN;
LOCK TABLE accounts IN ACCESS EXCLUSIVE MODE;
-- Tabelle komplett gesperrt
COMMIT;
```

---

## Prisma ORM

### Was ist Prisma?

**Prisma** ist ein modernes TypeScript ORM (Object-Relational Mapping) für Node.js.

**Vorteile:**
- ✅ Type-Safety mit TypeScript
- ✅ Automatische Migrations
- ✅ Intuitive Query-API
- ✅ Prisma Studio (GUI)

### Installation

```bash
npm install prisma --save-dev
npm install @prisma/client

# Prisma initialisieren
npx prisma init
```

### Prisma Schema

**prisma/schema.prisma:**

```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  posts     Post[]
  profile   Profile?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Profile {
  id     Int    @id @default(autoincrement())
  bio    String?
  userId Int    @unique
  user   User   @relation(fields: [userId], references: [id])
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String?
  published Boolean  @default(false)
  authorId  Int
  author    User     @relation(fields: [authorId], references: [id])
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

**.env:**

```env
DATABASE_URL="postgresql://myuser:mypassword@localhost:5432/mydb"
```

### Migrations

```bash
# Migration erstellen
npx prisma migrate dev --name init

# Migration anwenden (Production)
npx prisma migrate deploy

# Datenbank zurücksetzen
npx prisma migrate reset

# Prisma Studio starten (GUI)
npx prisma studio
```

### Prisma Client generieren

```bash
npx prisma generate
```

### CRUD mit Prisma

```typescript
// lib/prisma.ts
import { PrismaClient } from '@prisma/client'

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined
}

export const prisma = globalForPrisma.prisma ?? new PrismaClient()

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma
```

#### Create (Erstellen)

```typescript
import { prisma } from './lib/prisma'

// Einzelner User
const user = await prisma.user.create({
  data: {
    email: 'max@example.com',
    name: 'Max Mustermann'
  }
})

// Mit Relationen
const userWithPost = await prisma.user.create({
  data: {
    email: 'anna@example.com',
    name: 'Anna Schmidt',
    posts: {
      create: [
        { title: 'Erster Post', content: 'Hallo Welt!' },
        { title: 'Zweiter Post', content: 'TypeScript ist toll!' }
      ]
    }
  },
  include: {
    posts: true
  }
})
```

#### Read (Lesen)

```typescript
// Alle Users
const users = await prisma.user.findMany()

// Mit Filter
const users = await prisma.user.findMany({
  where: {
    email: {
      endsWith: '@example.com'
    }
  }
})

// Einzelner User
const user = await prisma.user.findUnique({
  where: { email: 'max@example.com' }
})

// Mit Relationen
const userWithPosts = await prisma.user.findUnique({
  where: { id: 1 },
  include: {
    posts: true,
    profile: true
  }
})

// Pagination
const users = await prisma.user.findMany({
  skip: 10,
  take: 5,
  orderBy: {
    createdAt: 'desc'
  }
})

// Aggregation
const count = await prisma.user.count()
const avg = await prisma.post.aggregate({
  _avg: {
    authorId: true
  }
})
```

#### Update (Aktualisieren)

```typescript
// Einzelnen User aktualisieren
const user = await prisma.user.update({
  where: { id: 1 },
  data: {
    name: 'Max Müller'
  }
})

// Mehrere Users aktualisieren
const result = await prisma.user.updateMany({
  where: {
    email: {
      endsWith: '@example.com'
    }
  },
  data: {
    name: 'Updated Name'
  }
})

// Upsert (Update oder Create)
const user = await prisma.user.upsert({
  where: { email: 'max@example.com' },
  update: {
    name: 'Max Updated'
  },
  create: {
    email: 'max@example.com',
    name: 'Max Created'
  }
})
```

#### Delete (Löschen)

```typescript
// Einzelnen User löschen
const user = await prisma.user.delete({
  where: { id: 1 }
})

// Mehrere Users löschen
const result = await prisma.user.deleteMany({
  where: {
    email: {
      contains: 'test'
    }
  }
})
```

### Transaktionen mit Prisma

```typescript
// Sequential Transaction
const [user, post] = await prisma.$transaction([
  prisma.user.create({
    data: { email: 'max@example.com', name: 'Max' }
  }),
  prisma.post.create({
    data: { title: 'Post', authorId: 1 }
  })
])

// Interactive Transaction
await prisma.$transaction(async (tx) => {
  const user = await tx.user.create({
    data: { email: 'anna@example.com', name: 'Anna' }
  })
  
  await tx.post.create({
    data: { 
      title: 'Post von Anna', 
      authorId: user.id 
    }
  })
})
```

### Raw SQL mit Prisma

```typescript
// Raw Query
const result = await prisma.$queryRaw`
  SELECT * FROM User WHERE email = ${'max@example.com'}
`

// Execute Raw (für INSERT/UPDATE/DELETE)
const result = await prisma.$executeRaw`
  UPDATE User SET name = ${'New Name'} WHERE id = ${1}
`
```

---

## Praktische Projekte

### Projekt 1: Blog-System

```sql
-- Datenbank-Schema
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE categories (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL,
    slug VARCHAR(50) UNIQUE NOT NULL
);

CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    slug VARCHAR(200) UNIQUE NOT NULL,
    content TEXT NOT NULL,
    excerpt TEXT,
    author_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    category_id INTEGER REFERENCES categories(id),
    published BOOLEAN DEFAULT FALSE,
    views INTEGER DEFAULT 0,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE comments (
    id SERIAL PRIMARY KEY,
    content TEXT NOT NULL,
    post_id INTEGER REFERENCES posts(id) ON DELETE CASCADE,
    user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
    parent_id INTEGER REFERENCES comments(id) ON DELETE CASCADE,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE tags (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL
);

CREATE TABLE post_tags (
    post_id INTEGER REFERENCES posts(id) ON DELETE CASCADE,
    tag_id INTEGER REFERENCES tags(id) ON DELETE CASCADE,
    PRIMARY KEY (post_id, tag_id)
);

-- Indexes für Performance
CREATE INDEX idx_posts_author ON posts(author_id);
CREATE INDEX idx_posts_category ON posts(category_id);
CREATE INDEX idx_posts_published ON posts(published);
CREATE INDEX idx_comments_post ON comments(post_id);

-- Sample Queries
-- Alle veröffentlichten Posts mit Autor und Kategorie
SELECT 
    p.title,
    p.excerpt,
    u.username AS author,
    c.name AS category,
    p.views,
    p.created_at
FROM posts p
JOIN users u ON p.author_id = u.id
LEFT JOIN categories c ON p.category_id = c.id
WHERE p.published = TRUE
ORDER BY p.created_at DESC
LIMIT 10;

-- Post mit allen Kommentaren
SELECT 
    p.title,
    c.content AS comment,
    u.username AS commenter,
    c.created_at
FROM posts p
LEFT JOIN comments c ON p.id = c.post_id
LEFT JOIN users u ON c.user_id = u.id
WHERE p.slug = 'mein-blog-post'
ORDER BY c.created_at DESC;

-- Top 5 Autoren mit den meisten Posts
SELECT 
    u.username,
    COUNT(p.id) AS post_count,
    SUM(p.views) AS total_views
FROM users u
LEFT JOIN posts p ON u.id = p.author_id
GROUP BY u.id, u.username
ORDER BY post_count DESC
LIMIT 5;
```

### Projekt 2: E-Commerce-System

```sql
-- Produkte
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    price NUMERIC(10, 2) NOT NULL CHECK (price >= 0),
    stock INTEGER DEFAULT 0 CHECK (stock >= 0),
    image_url VARCHAR(500),
    created_at TIMESTAMP DEFAULT NOW()
);

-- Kunden
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    phone VARCHAR(20),
    created_at TIMESTAMP DEFAULT NOW()
);

-- Bestellungen
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER REFERENCES customers(id),
    status VARCHAR(20) DEFAULT 'pending',
    total NUMERIC(10, 2) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    CONSTRAINT valid_status CHECK (status IN ('pending', 'paid', 'shipped', 'delivered', 'cancelled'))
);

-- Bestellpositionen
CREATE TABLE order_items (
    id SERIAL PRIMARY KEY,
    order_id INTEGER REFERENCES orders(id) ON DELETE CASCADE,
    product_id INTEGER REFERENCES products(id),
    quantity INTEGER NOT NULL CHECK (quantity > 0),
    price NUMERIC(10, 2) NOT NULL,
    subtotal NUMERIC(10, 2) NOT NULL
);

-- Warenkorb
CREATE TABLE cart (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER REFERENCES customers(id),
    product_id INTEGER REFERENCES products(id),
    quantity INTEGER DEFAULT 1 CHECK (quantity > 0),
    UNIQUE (customer_id, product_id)
);

-- Transactional: Bestellung erstellen
BEGIN;

-- Neue Bestellung erstellen
INSERT INTO orders (customer_id, total)
VALUES (1, 0)
RETURNING id;  -- z.B. order_id = 42

-- Warenkorb-Items in Bestellung übertragen
INSERT INTO order_items (order_id, product_id, quantity, price, subtotal)
SELECT 
    42,  -- order_id
    c.product_id,
    c.quantity,
    p.price,
    c.quantity * p.price
FROM cart c
JOIN products p ON c.product_id = p.id
WHERE c.customer_id = 1;

-- Lagerbestand reduzieren
UPDATE products p
SET stock = stock - c.quantity
FROM cart c
WHERE c.customer_id = 1 AND p.id = c.product_id;

-- Total der Bestellung aktualisieren
UPDATE orders
SET total = (
    SELECT SUM(subtotal)
    FROM order_items
    WHERE order_id = 42
)
WHERE id = 42;

-- Warenkorb leeren
DELETE FROM cart WHERE customer_id = 1;

COMMIT;
```

---

## Zusammenfassung

Du hast nun gelernt:

✅ **Datenbank-Grundlagen**: Tabellen, Zeilen, Spalten, Relationships
✅ **SQL Basics**: SELECT, INSERT, UPDATE, DELETE
✅ **PostgreSQL**: Installation, Setup, psql
✅ **CRUD**: Alle grundlegenden Operationen
✅ **Datentypen**: Numerisch, Text, Datum, JSON, Arrays
✅ **Constraints**: PRIMARY KEY, FOREIGN KEY, UNIQUE, CHECK
✅ **Beziehungen**: 1:1, 1:N, N:M, Self-Referencing
✅ **JOINs**: INNER, LEFT, RIGHT, FULL
✅ **Aggregation**: COUNT, SUM, AVG, GROUP BY, HAVING
✅ **Subqueries**: Verschachtelte Queries
✅ **Performance**: Indexes, EXPLAIN, Optimierung
✅ **Transaktionen**: ACID, BEGIN, COMMIT, ROLLBACK
✅ **Prisma ORM**: Type-safe Database Access

### SQL Cheat Sheet

```sql
-- CRUD
SELECT * FROM table;
INSERT INTO table (col1, col2) VALUES (val1, val2);
UPDATE table SET col1 = val1 WHERE id = 1;
DELETE FROM table WHERE id = 1;

-- JOINs
SELECT * FROM t1 INNER JOIN t2 ON t1.id = t2.t1_id;
SELECT * FROM t1 LEFT JOIN t2 ON t1.id = t2.t1_id;

-- Aggregation
SELECT COUNT(*), AVG(price), SUM(total) FROM orders;
SELECT category, COUNT(*) FROM products GROUP BY category;

-- Performance
CREATE INDEX idx_name ON table(column);
EXPLAIN ANALYZE SELECT * FROM table WHERE column = value;
```

### Nächste Schritte

1. **Advanced SQL**: Window Functions, CTEs, Recursive Queries
2. **Database Design**: Normalisierung, ER-Diagramme
3. **Backup & Recovery**: pg_dump, Replication
4. **Security**: SQL Injection Prevention, Row-Level Security
5. **Prisma Advanced**: Middleware, Extensions, Custom Queries

### Hilfreiche Ressourcen

- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Prisma Documentation](https://www.prisma.io/docs)
- [SQL Tutorial](https://www.w3schools.com/sql/)
- [PostgreSQL Tutorial](https://www.postgresqltutorial.com/)

Viel Erfolg mit SQL und Datenbanken! 🚀
