# TypeScript Basics - Kompletter Walkthrough

## Inhaltsverzeichnis
1. [Einführung](#einführung)
2. [Installation und Setup](#installation-und-setup)
3. [Grundlegende Typen](#grundlegende-typen)
4. [Funktionen](#funktionen)
5. [Interfaces und Type Aliases](#interfaces-und-type-aliases)
6. [Klassen](#klassen)
7. [Generics](#generics)
8. [Union und Intersection Types](#union-und-intersection-types)
9. [Type Guards](#type-guards)
10. [Praktische Übungen](#praktische-übungen)

---

## Einführung

### Was ist TypeScript?
TypeScript ist eine von Microsoft entwickelte, typisierte Obermenge von JavaScript. Es erweitert JavaScript um statische Typen und kompiliert zu reinem JavaScript.

### Warum TypeScript?
- **Frühe Fehlererkennung**: Fehler werden bereits beim Schreiben des Codes erkannt
- **Bessere IDE-Unterstützung**: Autovervollständigung und IntelliSense
- **Verbesserte Wartbarkeit**: Code ist selbstdokumentierend durch Typen
- **Refactoring**: Sicheres Umstrukturieren von Code

---

## Installation und Setup

### Schritt 1: Node.js installieren
Lade Node.js von [nodejs.org](https://nodejs.org) herunter und installiere es.

### Schritt 2: TypeScript global installieren
```bash
npm install -g typescript
```

### Schritt 3: Version überprüfen
```bash
tsc --version
```

### Schritt 4: Erstes TypeScript-Projekt erstellen
```bash
# Projektordner erstellen
mkdir mein-ts-projekt
cd mein-ts-projekt

# package.json initialisieren
npm init -y

# TypeScript lokal installieren
npm install --save-dev typescript

# tsconfig.json erstellen
tsc --init
```

### Schritt 5: tsconfig.json konfigurieren
Öffne `tsconfig.json` und konfiguriere die wichtigsten Optionen:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules"]
}
```

---

## Grundlegende Typen

### Primitive Typen

#### String
```typescript
let name: string = "Max";
let greeting: string = `Hallo, ${name}!`;
```

#### Number
```typescript
let alter: number = 25;
let preis: number = 19.99;
let hex: number = 0xf00d;
let binary: number = 0b1010;
```

#### Boolean
```typescript
let istAktiv: boolean = true;
let istVerheiratet: boolean = false;
```

#### Null und Undefined
```typescript
let nichts: null = null;
let undefiniert: undefined = undefined;
```

### Arrays

```typescript
// Methode 1: Type[]
let zahlen: number[] = [1, 2, 3, 4, 5];
let namen: string[] = ["Anna", "Ben", "Clara"];

// Methode 2: Array<Type>
let farben: Array<string> = ["rot", "grün", "blau"];

// Gemischte Arrays mit Union Types
let mixed: (string | number)[] = [1, "zwei", 3, "vier"];
```

### Tupel
Tupel erlauben Arrays mit fester Länge und bekannten Typen an jeder Position.

```typescript
// Koordinaten [x, y]
let koordinate: [number, number] = [10, 20];

// Person [Name, Alter]
let person: [string, number] = ["Max", 30];

// Fehler: Falsche Reihenfolge
// let fehler: [string, number] = [30, "Max"]; // ❌
```

### Enums
Enums ermöglichen es, benannte Konstanten zu definieren.

```typescript
// Numerisches Enum (startet bei 0)
enum Richtung {
  Oben,    // 0
  Rechts,  // 1
  Unten,   // 2
  Links    // 3
}

let bewegung: Richtung = Richtung.Oben;

// String Enum
enum Status {
  Ausstehend = "AUSSTEHEND",
  Aktiv = "AKTIV",
  Abgeschlossen = "ABGESCHLOSSEN"
}

let aufgabenStatus: Status = Status.Aktiv;
```

### Any und Unknown

```typescript
// Any: Deaktiviert Type-Checking (vermeiden!)
let irgendetwas: any = 4;
irgendetwas = "jetzt ein String";
irgendetwas = true;

// Unknown: Typsicher, aber flexibel
let unbekannt: unknown = 4;
// unbekannt.toFixed(); // ❌ Fehler

// Type Guard erforderlich
if (typeof unbekannt === "number") {
  console.log(unbekannt.toFixed(2)); // ✅ OK
}
```

### Void, Never

```typescript
// Void: Funktion gibt nichts zurück
function loggen(nachricht: string): void {
  console.log(nachricht);
}

// Never: Funktion kehrt nie zurück
function fehler(nachricht: string): never {
  throw new Error(nachricht);
}

function endlosSchleife(): never {
  while (true) {
    // läuft ewig
  }
}
```

---

## Funktionen

### Funktionstypen

```typescript
// Benannte Funktion
function addieren(a: number, b: number): number {
  return a + b;
}

// Anonyme Funktion
const multiplizieren = function(a: number, b: number): number {
  return a * b;
};

// Arrow Function
const subtrahieren = (a: number, b: number): number => {
  return a - b;
};

// Kurzform Arrow Function
const dividieren = (a: number, b: number): number => a / b;
```

### Optionale und Default-Parameter

```typescript
// Optionale Parameter (mit ?)
function begruessung(vorname: string, nachname?: string): string {
  if (nachname) {
    return `Hallo ${vorname} ${nachname}`;
  }
  return `Hallo ${vorname}`;
}

console.log(begruessung("Max")); // "Hallo Max"
console.log(begruessung("Max", "Mustermann")); // "Hallo Max Mustermann"

// Default-Parameter
function erstellen(name: string, aktiv: boolean = true): object {
  return { name, aktiv };
}

console.log(erstellen("Produkt")); // { name: "Produkt", aktiv: true }
console.log(erstellen("Produkt", false)); // { name: "Produkt", aktiv: false }
```

### Rest-Parameter

```typescript
function summe(...zahlen: number[]): number {
  return zahlen.reduce((total, n) => total + n, 0);
}

console.log(summe(1, 2, 3)); // 6
console.log(summe(1, 2, 3, 4, 5)); // 15
```

### Funktionsüberladung

```typescript
// Überladungs-Signaturen
function formatieren(wert: string): string;
function formatieren(wert: number): string;
function formatieren(wert: boolean): string;

// Implementierung
function formatieren(wert: string | number | boolean): string {
  if (typeof wert === "string") {
    return wert.toUpperCase();
  } else if (typeof wert === "number") {
    return wert.toFixed(2);
  } else {
    return wert ? "Ja" : "Nein";
  }
}

console.log(formatieren("hallo")); // "HALLO"
console.log(formatieren(42.5678)); // "42.57"
console.log(formatieren(true)); // "Ja"
```

---

## Interfaces und Type Aliases

### Interfaces

Interfaces definieren die Struktur von Objekten.

```typescript
interface Person {
  vorname: string;
  nachname: string;
  alter: number;
  email?: string; // optional
}

const benutzer: Person = {
  vorname: "Anna",
  nachname: "Schmidt",
  alter: 28
};

// Mit optionalem Property
const admin: Person = {
  vorname: "Max",
  nachname: "Müller",
  alter: 35,
  email: "max@example.com"
};
```

### Readonly Properties

```typescript
interface Buch {
  readonly isbn: string;
  titel: string;
  autor: string;
  seiten: number;
}

const meinBuch: Buch = {
  isbn: "978-3-16-148410-0",
  titel: "TypeScript für Einsteiger",
  autor: "Max Mustermann",
  seiten: 300
};

// meinBuch.isbn = "andere-isbn"; // ❌ Fehler: readonly
meinBuch.titel = "TypeScript für Profis"; // ✅ OK
```

### Interface für Funktionen

```typescript
interface Rechner {
  (a: number, b: number): number;
}

const addiere: Rechner = (x, y) => x + y;
const subtrahiere: Rechner = (x, y) => x - y;

console.log(addiere(5, 3)); // 8
console.log(subtrahiere(5, 3)); // 2
```

### Interface Extending

```typescript
interface Tier {
  name: string;
  alter: number;
}

interface Hund extends Tier {
  rasse: string;
  bellen(): void;
}

const meinHund: Hund = {
  name: "Bello",
  alter: 5,
  rasse: "Golden Retriever",
  bellen() {
    console.log("Wuff! Wuff!");
  }
};
```

### Type Aliases

Type Aliases ermöglichen es, Typen mit einem Namen zu versehen.

```typescript
// Einfacher Type Alias
type ID = string | number;

let benutzerId: ID = 123;
benutzerId = "abc-456";

// Objekt Type
type Produkt = {
  id: number;
  name: string;
  preis: number;
  verfuegbar: boolean;
};

const laptop: Produkt = {
  id: 1,
  name: "ThinkPad",
  preis: 999.99,
  verfuegbar: true
};

// Funktions-Type
type Operation = (a: number, b: number) => number;

const multi: Operation = (x, y) => x * y;
```

### Interface vs Type Alias

**Verwende Interface wenn:**
- Du Objekt-Formen definierst
- Du die Möglichkeit zur Erweiterung brauchst
- Du mit Klassen arbeitest

**Verwende Type Alias wenn:**
- Du Union oder Intersection Types brauchst
- Du primitive Typen aliassen willst
- Du Tupel oder komplexe Typen definierst

```typescript
// Interface kann erweitert werden (Declaration Merging)
interface Fenster {
  titel: string;
}

interface Fenster {
  breite: number;
}

// Zusammengeführt zu: { titel: string, breite: number }

// Type Alias kann nicht erweitert werden (Fehler!)
type Tuer = { farbe: string };
// type Tuer = { groesse: number }; // ❌ Fehler: Duplicate identifier
```

---

## Klassen

### Grundlegende Klassen

```typescript
class Person {
  // Properties
  vorname: string;
  nachname: string;
  alter: number;

  // Constructor
  constructor(vorname: string, nachname: string, alter: number) {
    this.vorname = vorname;
    this.nachname = nachname;
    this.alter = alter;
  }

  // Methode
  vollstaendigerName(): string {
    return `${this.vorname} ${this.nachname}`;
  }

  // Methode
  vorstellen(): void {
    console.log(`Ich bin ${this.vollstaendigerName()} und ${this.alter} Jahre alt.`);
  }
}

const person1 = new Person("Anna", "Schmidt", 28);
console.log(person1.vollstaendigerName()); // "Anna Schmidt"
person1.vorstellen(); // "Ich bin Anna Schmidt und 28 Jahre alt."
```

### Access Modifiers

```typescript
class Bankkonto {
  public inhaber: string;        // Von überall zugänglich
  private kontostand: number;    // Nur innerhalb der Klasse
  protected kontonummer: string; // In Klasse und Subklassen

  constructor(inhaber: string, kontostand: number, kontonummer: string) {
    this.inhaber = inhaber;
    this.kontostand = kontostand;
    this.kontonummer = kontonummer;
  }

  public einzahlen(betrag: number): void {
    if (betrag > 0) {
      this.kontostand += betrag;
      console.log(`${betrag}€ eingezahlt. Neuer Kontostand: ${this.kontostand}€`);
    }
  }

  public auszahlen(betrag: number): boolean {
    if (betrag > 0 && betrag <= this.kontostand) {
      this.kontostand -= betrag;
      console.log(`${betrag}€ ausgezahlt. Neuer Kontostand: ${this.kontostand}€`);
      return true;
    }
    console.log("Unzureichender Kontostand!");
    return false;
  }

  public getKontostand(): number {
    return this.kontostand;
  }
}

const konto = new Bankkonto("Max Mustermann", 1000, "DE89370400440532013000");
console.log(konto.inhaber); // "Max Mustermann"
// console.log(konto.kontostand); // ❌ Fehler: private
konto.einzahlen(500); // ✅ OK
console.log(konto.getKontostand()); // 1500
```

### Kurzschreibweise im Constructor

```typescript
class Mitarbeiter {
  // Properties werden automatisch erstellt und initialisiert
  constructor(
    public name: string,
    private gehalt: number,
    protected abteilung: string
  ) {}

  getInfo(): string {
    return `${this.name} arbeitet in der Abteilung ${this.abteilung}`;
  }
}

const mitarbeiter = new Mitarbeiter("Lisa Müller", 50000, "IT");
console.log(mitarbeiter.name); // "Lisa Müller"
// console.log(mitarbeiter.gehalt); // ❌ Fehler: private
```

### Getters und Setters

```typescript
class Temperatur {
  private _celsius: number = 0;

  get celsius(): number {
    return this._celsius;
  }

  set celsius(wert: number) {
    if (wert < -273.15) {
      throw new Error("Temperatur kann nicht unter -273.15°C sein!");
    }
    this._celsius = wert;
  }

  get fahrenheit(): number {
    return (this._celsius * 9/5) + 32;
  }

  set fahrenheit(wert: number) {
    this.celsius = (wert - 32) * 5/9;
  }
}

const temp = new Temperatur();
temp.celsius = 25;
console.log(temp.celsius); // 25
console.log(temp.fahrenheit); // 77

temp.fahrenheit = 86;
console.log(temp.celsius); // 30
```

### Vererbung

```typescript
class Fahrzeug {
  constructor(
    public marke: string,
    public modell: string,
    protected baujahr: number
  ) {}

  beschreibung(): string {
    return `${this.marke} ${this.modell} (${this.baujahr})`;
  }

  starten(): void {
    console.log(`${this.beschreibung()} startet...`);
  }
}

class Auto extends Fahrzeug {
  constructor(
    marke: string,
    modell: string,
    baujahr: number,
    public tueren: number
  ) {
    super(marke, modell, baujahr); // Eltern-Constructor aufrufen
  }

  // Methode überschreiben
  beschreibung(): string {
    return `${super.beschreibung()} mit ${this.tueren} Türen`;
  }

  hupen(): void {
    console.log("Huuup! Huuup!");
  }
}

const meinAuto = new Auto("VW", "Golf", 2022, 5);
console.log(meinAuto.beschreibung()); // "VW Golf (2022) mit 5 Türen"
meinAuto.starten(); // "VW Golf (2022) mit 5 Türen startet..."
meinAuto.hupen(); // "Huuup! Huuup!"
```

### Abstract Classes

```typescript
abstract class Form {
  constructor(public farbe: string) {}

  abstract flaeche(): number;
  abstract umfang(): number;

  beschreibung(): string {
    return `Eine ${this.farbe} Form mit Fläche ${this.flaeche().toFixed(2)}`;
  }
}

class Kreis extends Form {
  constructor(farbe: string, public radius: number) {
    super(farbe);
  }

  flaeche(): number {
    return Math.PI * this.radius ** 2;
  }

  umfang(): number {
    return 2 * Math.PI * this.radius;
  }
}

class Rechteck extends Form {
  constructor(farbe: string, public breite: number, public hoehe: number) {
    super(farbe);
  }

  flaeche(): number {
    return this.breite * this.hoehe;
  }

  umfang(): number {
    return 2 * (this.breite + this.hoehe);
  }
}

const kreis = new Kreis("rot", 5);
const rechteck = new Rechteck("blau", 10, 20);

console.log(kreis.beschreibung()); // "Eine rot Form mit Fläche 78.54"
console.log(rechteck.beschreibung()); // "Eine blau Form mit Fläche 200.00"
```

### Static Members

```typescript
class Mathematik {
  static PI: number = 3.14159;

  static kreisflaeche(radius: number): number {
    return this.PI * radius ** 2;
  }

  static max(...zahlen: number[]): number {
    return Math.max(...zahlen);
  }
}

// Ohne Instanz verwendbar
console.log(Mathematik.PI); // 3.14159
console.log(Mathematik.kreisflaeche(5)); // 78.54
console.log(Mathematik.max(1, 5, 3, 9, 2)); // 9
```

---

## Generics

Generics ermöglichen es, wiederverwendbare Komponenten zu erstellen, die mit verschiedenen Typen arbeiten können.

### Grundlegende Generics

```typescript
// Ohne Generics (nicht flexibel)
function identitaetNumber(wert: number): number {
  return wert;
}

function identitaetString(wert: string): string {
  return wert;
}

// Mit Generics (flexibel und typsicher)
function identitaet<T>(wert: T): T {
  return wert;
}

const zahl = identitaet<number>(42);
const text = identitaet<string>("Hallo");
const wahrheit = identitaet<boolean>(true);

// Type Inference (TypeScript erkennt den Typ automatisch)
const zahl2 = identitaet(42); // T wird als number inferiert
const text2 = identitaet("Welt"); // T wird als string inferiert
```

### Generic Arrays

```typescript
function erstesElement<T>(arr: T[]): T | undefined {
  return arr[0];
}

const zahlen = [1, 2, 3, 4, 5];
const erste = erstesElement(zahlen); // Type: number | undefined

const namen = ["Anna", "Ben", "Clara"];
const ersteName = erstesElement(namen); // Type: string | undefined

// Generic mit mehreren Type-Parametern
function paar<T, U>(erstes: T, zweites: U): [T, U] {
  return [erstes, zweites];
}

const p1 = paar<string, number>("Alter", 25); // [string, number]
const p2 = paar("Name", true); // Type Inference: [string, boolean]
```

### Generic Interfaces

```typescript
interface Box<T> {
  inhalt: T;
  leer: boolean;
  oeffnen(): T;
  fuellen(item: T): void;
}

class Geschenkbox<T> implements Box<T> {
  leer: boolean = true;

  constructor(public inhalt: T) {
    if (inhalt !== null && inhalt !== undefined) {
      this.leer = false;
    }
  }

  oeffnen(): T {
    this.leer = true;
    return this.inhalt;
  }

  fuellen(item: T): void {
    this.inhalt = item;
    this.leer = false;
  }
}

const spielzeugBox = new Geschenkbox<string>("Teddybär");
console.log(spielzeugBox.oeffnen()); // "Teddybär"

const zahlenBox = new Geschenkbox<number>(42);
console.log(zahlenBox.oeffnen()); // 42
```

### Generic Constraints

```typescript
// Constraint: T muss eine length-Property haben
interface HatLaenge {
  length: number;
}

function logLaenge<T extends HatLaenge>(item: T): void {
  console.log(`Länge: ${item.length}`);
}

logLaenge("Hallo"); // "Länge: 5"
logLaenge([1, 2, 3, 4]); // "Länge: 4"
logLaenge({ length: 10, wert: "test" }); // "Länge: 10"
// logLaenge(123); // ❌ Fehler: number hat keine length-Property

// Constraint mit keyof
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const person = {
  name: "Max",
  alter: 30,
  stadt: "Berlin"
};

const name = getProperty(person, "name"); // Type: string
const alter = getProperty(person, "alter"); // Type: number
// const fehler = getProperty(person, "email"); // ❌ Fehler: "email" existiert nicht
```

### Generic Classes

```typescript
class Stapel<T> {
  private items: T[] = [];

  push(item: T): void {
    this.items.push(item);
  }

  pop(): T | undefined {
    return this.items.pop();
  }

  peek(): T | undefined {
    return this.items[this.items.length - 1];
  }

  istLeer(): boolean {
    return this.items.length === 0;
  }

  groesse(): number {
    return this.items.length;
  }
}

const zahlenStapel = new Stapel<number>();
zahlenStapel.push(1);
zahlenStapel.push(2);
zahlenStapel.push(3);
console.log(zahlenStapel.pop()); // 3

const textStapel = new Stapel<string>();
textStapel.push("Hallo");
textStapel.push("Welt");
console.log(textStapel.peek()); // "Welt"
```

---

## Union und Intersection Types

### Union Types

Union Types erlauben es, dass ein Wert einen von mehreren Typen haben kann.

```typescript
// Einfacher Union Type
let id: string | number;
id = 123;      // ✅ OK
id = "abc123"; // ✅ OK
// id = true;  // ❌ Fehler

// Union Type in Funktionen
function formatieren(wert: string | number): string {
  if (typeof wert === "string") {
    return wert.toUpperCase();
  } else {
    return wert.toFixed(2);
  }
}

console.log(formatieren("hallo")); // "HALLO"
console.log(formatieren(42.678)); // "42.68"

// Union Type mit Literaltypen
type Status = "ausstehend" | "laufend" | "abgeschlossen" | "fehlgeschlagen";

function setzeStatus(status: Status): void {
  console.log(`Status: ${status}`);
}

setzeStatus("laufend"); // ✅ OK
// setzeStatus("unbekannt"); // ❌ Fehler
```

### Array mit Union Types

```typescript
// Array das Strings ODER Numbers enthalten kann
let gemischt: (string | number)[] = [1, "zwei", 3, "vier"];

// Array das NUR Strings ist ODER NUR Numbers ist
let einheitlich: string[] | number[] = ["a", "b", "c"];
einheitlich = [1, 2, 3]; // ✅ OK
// einheitlich = [1, "zwei"]; // ❌ Fehler
```

### Intersection Types

Intersection Types kombinieren mehrere Typen zu einem.

```typescript
interface Person {
  name: string;
  alter: number;
}

interface Mitarbeiter {
  mitarbeiterId: number;
  abteilung: string;
}

// Intersection: Hat alle Properties von Person UND Mitarbeiter
type Angestellter = Person & Mitarbeiter;

const angestellter: Angestellter = {
  name: "Max Mustermann",
  alter: 30,
  mitarbeiterId: 12345,
  abteilung: "IT"
};

// Funktion mit Intersection Type
function druckeAngestelltenInfo(angestellter: Person & Mitarbeiter): void {
  console.log(`${angestellter.name} (${angestellter.alter}) - ` +
              `ID: ${angestellter.mitarbeiterId}, ${angestellter.abteilung}`);
}

druckeAngestelltenInfo(angestellter);
```

### Komplexe Kombinationen

```typescript
type Kontaktmethode = "email" | "telefon" | "post";
type Prioritaet = "niedrig" | "mittel" | "hoch";

interface Nachricht {
  inhalt: string;
  datum: Date;
}

interface EmailNachricht extends Nachricht {
  betreff: string;
  empfaenger: string;
}

interface TelefonNachricht extends Nachricht {
  telefonnummer: string;
  dauer: number;
}

// Union von Intersection Types
type KommunikationsLog = 
  | (EmailNachricht & { methode: "email" })
  | (TelefonNachricht & { methode: "telefon" })
  | (Nachricht & { methode: "post"; adresse: string });

function logKommunikation(log: KommunikationsLog): void {
  console.log(`Methode: ${log.methode}`);
  console.log(`Inhalt: ${log.inhalt}`);
  
  if (log.methode === "email") {
    console.log(`Betreff: ${log.betreff}`);
  } else if (log.methode === "telefon") {
    console.log(`Dauer: ${log.dauer} Minuten`);
  } else {
    console.log(`Adresse: ${log.adresse}`);
  }
}
```

---

## Type Guards

Type Guards helfen dabei, den Typ einer Variable innerhalb eines bestimmten Codeblocks einzugrenzen.

### typeof Type Guards

```typescript
function verarbeiten(wert: string | number): string {
  if (typeof wert === "string") {
    // Hier weiß TypeScript, dass wert ein string ist
    return wert.toUpperCase();
  } else {
    // Hier weiß TypeScript, dass wert eine number ist
    return wert.toFixed(2);
  }
}

console.log(verarbeiten("hallo")); // "HALLO"
console.log(verarbeiten(3.14159)); // "3.14"
```

### instanceof Type Guards

```typescript
class Hund {
  bellen(): void {
    console.log("Wuff!");
  }
}

class Katze {
  miauen(): void {
    console.log("Miau!");
  }
}

function tierGeraeusch(tier: Hund | Katze): void {
  if (tier instanceof Hund) {
    tier.bellen(); // TypeScript weiß: tier ist ein Hund
  } else {
    tier.miauen(); // TypeScript weiß: tier ist eine Katze
  }
}

const hund = new Hund();
const katze = new Katze();

tierGeraeusch(hund); // "Wuff!"
tierGeraeusch(katze); // "Miau!"
```

### in Type Guards

```typescript
interface Vogel {
  fliegen(): void;
  fluegel: number;
}

interface Fisch {
  schwimmen(): void;
  flossen: number;
}

function bewegen(tier: Vogel | Fisch): void {
  if ("fliegen" in tier) {
    // Hier ist tier ein Vogel
    console.log(`Vogel fliegt mit ${tier.fluegel} Flügeln`);
    tier.fliegen();
  } else {
    // Hier ist tier ein Fisch
    console.log(`Fisch schwimmt mit ${tier.flossen} Flossen`);
    tier.schwimmen();
  }
}

const vogel: Vogel = {
  fliegen: () => console.log("Flieg!"),
  fluegel: 2
};

const fisch: Fisch = {
  schwimmen: () => console.log("Schwimm!"),
  flossen: 4
};

bewegen(vogel);
bewegen(fisch);
```

### Custom Type Guards (User-Defined Type Guards)

```typescript
interface Auto {
  fahren(): void;
  tueren: number;
}

interface Boot {
  segeln(): void;
  segel: number;
}

// Custom Type Guard Funktion
function istAuto(fahrzeug: Auto | Boot): fahrzeug is Auto {
  return (fahrzeug as Auto).fahren !== undefined;
}

function fahrzeugBewegen(fahrzeug: Auto | Boot): void {
  if (istAuto(fahrzeug)) {
    // TypeScript weiß: fahrzeug ist ein Auto
    console.log(`Auto mit ${fahrzeug.tueren} Türen`);
    fahrzeug.fahren();
  } else {
    // TypeScript weiß: fahrzeug ist ein Boot
    console.log(`Boot mit ${fahrzeug.segel} Segeln`);
    fahrzeug.segeln();
  }
}

const auto: Auto = {
  fahren: () => console.log("Fahr!"),
  tueren: 4
};

const boot: Boot = {
  segeln: () => console.log("Segel!"),
  segel: 2
};

fahrzeugBewegen(auto);
fahrzeugBewegen(boot);
```

### Discriminated Unions

```typescript
interface Quadrat {
  art: "quadrat";
  seitenlaenge: number;
}

interface Rechteck {
  art: "rechteck";
  breite: number;
  hoehe: number;
}

interface Kreis {
  art: "kreis";
  radius: number;
}

type Form = Quadrat | Rechteck | Kreis;

function berechneFlaeche(form: Form): number {
  switch (form.art) {
    case "quadrat":
      return form.seitenlaenge ** 2;
    case "rechteck":
      return form.breite * form.hoehe;
    case "kreis":
      return Math.PI * form.radius ** 2;
    default:
      // Exhaustiveness Check
      const _exhaustiveCheck: never = form;
      return _exhaustiveCheck;
  }
}

const quadrat: Quadrat = { art: "quadrat", seitenlaenge: 5 };
const rechteck: Rechteck = { art: "rechteck", breite: 10, hoehe: 20 };
const kreis: Kreis = { art: "kreis", radius: 7 };

console.log(berechneFlaeche(quadrat)); // 25
console.log(berechneFlaeche(rechteck)); // 200
console.log(berechneFlaeche(kreis)); // 153.94
```

---

## Praktische Übungen

### Übung 1: Todo-Liste (Einfach)

Erstelle eine typisierte Todo-Liste Anwendung.

```typescript
interface Todo {
  id: number;
  titel: string;
  erledigt: boolean;
  erstellt: Date;
}

class TodoListe {
  private todos: Todo[] = [];
  private naechsteId: number = 1;

  hinzufuegen(titel: string): Todo {
    const neuesTodo: Todo = {
      id: this.naechsteId++,
      titel,
      erledigt: false,
      erstellt: new Date()
    };
    this.todos.push(neuesTodo);
    return neuesTodo;
  }

  alleAnzeigen(): Todo[] {
    return this.todos;
  }

  erledigteAnzeigen(): Todo[] {
    return this.todos.filter(todo => todo.erledigt);
  }

  offeneAnzeigen(): Todo[] {
    return this.todos.filter(todo => !todo.erledigt);
  }

  alsErledigtMarkieren(id: number): boolean {
    const todo = this.todos.find(t => t.id === id);
    if (todo) {
      todo.erledigt = true;
      return true;
    }
    return false;
  }

  loeschen(id: number): boolean {
    const index = this.todos.findIndex(t => t.id === id);
    if (index !== -1) {
      this.todos.splice(index, 1);
      return true;
    }
    return false;
  }
}

// Verwendung
const meineTodos = new TodoListe();
meineTodos.hinzufuegen("TypeScript lernen");
meineTodos.hinzufuegen("Projekt erstellen");
meineTodos.hinzufuegen("Dokumentation schreiben");

console.log("Alle Todos:", meineTodos.alleAnzeigen());
meineTodos.alsErledigtMarkieren(1);
console.log("Offene Todos:", meineTodos.offeneAnzeigen());
```

### Übung 2: Generische Datenverwaltung (Mittel)

Erstelle eine generische Datenbank-Klasse.

```typescript
interface Entitaet {
  id: number;
}

class Datenbank<T extends Entitaet> {
  private daten: T[] = [];
  private naechsteId: number = 1;

  erstellen(item: Omit<T, 'id'>): T {
    const neuesItem = { ...item, id: this.naechsteId++ } as T;
    this.daten.push(neuesItem);
    return neuesItem;
  }

  lesen(id: number): T | undefined {
    return this.daten.find(item => item.id === id);
  }

  alleAuslesen(): T[] {
    return [...this.daten];
  }

  aktualisieren(id: number, updates: Partial<Omit<T, 'id'>>): T | undefined {
    const index = this.daten.findIndex(item => item.id === id);
    if (index !== -1) {
      this.daten[index] = { ...this.daten[index], ...updates };
      return this.daten[index];
    }
    return undefined;
  }

  loeschen(id: number): boolean {
    const index = this.daten.findIndex(item => item.id === id);
    if (index !== -1) {
      this.daten.splice(index, 1);
      return true;
    }
    return false;
  }

  filtern(predicate: (item: T) => boolean): T[] {
    return this.daten.filter(predicate);
  }
}

// Verwendungsbeispiel
interface Benutzer extends Entitaet {
  name: string;
  email: string;
  aktiv: boolean;
}

const benutzerDB = new Datenbank<Benutzer>();

const benutzer1 = benutzerDB.erstellen({
  name: "Max Mustermann",
  email: "max@example.com",
  aktiv: true
});

const benutzer2 = benutzerDB.erstellen({
  name: "Anna Schmidt",
  email: "anna@example.com",
  aktiv: false
});

console.log("Alle Benutzer:", benutzerDB.alleAuslesen());
console.log("Aktive Benutzer:", benutzerDB.filtern(b => b.aktiv));

benutzerDB.aktualisieren(2, { aktiv: true });
console.log("Benutzer 2:", benutzerDB.lesen(2));
```

### Übung 3: Formular-Validator (Fortgeschritten)

Erstelle ein typsicheres Formular-Validierungssystem.

```typescript
type ValidationRule<T> = {
  validate: (value: T) => boolean;
  nachricht: string;
};

type FormFeld<T> = {
  wert: T;
  regeln: ValidationRule<T>[];
  fehler: string[];
};

type FormSchema<T> = {
  [K in keyof T]: FormFeld<T[K]>;
};

class FormValidator<T extends Record<string, any>> {
  private schema: FormSchema<T>;

  constructor(initialWerte: T) {
    this.schema = {} as FormSchema<T>;
    for (const key in initialWerte) {
      this.schema[key] = {
        wert: initialWerte[key],
        regeln: [],
        fehler: []
      };
    }
  }

  feldRegel<K extends keyof T>(
    feld: K,
    regel: ValidationRule<T[K]>
  ): this {
    this.schema[feld].regeln.push(regel);
    return this;
  }

  setzeWert<K extends keyof T>(feld: K, wert: T[K]): void {
    this.schema[feld].wert = wert;
  }

  validiereAlles(): boolean {
    let istGueltig = true;

    for (const key in this.schema) {
      const feld = this.schema[key];
      feld.fehler = [];

      for (const regel of feld.regeln) {
        if (!regel.validate(feld.wert)) {
          feld.fehler.push(regel.nachricht);
          istGueltig = false;
        }
      }
    }

    return istGueltig;
  }

  getFehler<K extends keyof T>(feld: K): string[] {
    return this.schema[feld].fehler;
  }

  getAlleFehler(): Record<keyof T, string[]> {
    const fehler = {} as Record<keyof T, string[]>;
    for (const key in this.schema) {
      fehler[key] = this.schema[key].fehler;
    }
    return fehler;
  }

  getWerte(): T {
    const werte = {} as T;
    for (const key in this.schema) {
      werte[key] = this.schema[key].wert;
    }
    return werte;
  }
}

// Verwendungsbeispiel
interface RegistrierungsForm {
  benutzername: string;
  email: string;
  passwort: string;
  alter: number;
}

const form = new FormValidator<RegistrierungsForm>({
  benutzername: "",
  email: "",
  passwort: "",
  alter: 0
});

// Validierungsregeln hinzufügen
form
  .feldRegel("benutzername", {
    validate: (wert) => wert.length >= 3,
    nachricht: "Benutzername muss mindestens 3 Zeichen lang sein"
  })
  .feldRegel("benutzername", {
    validate: (wert) => /^[a-zA-Z0-9_]+$/.test(wert),
    nachricht: "Nur Buchstaben, Zahlen und Unterstriche erlaubt"
  })
  .feldRegel("email", {
    validate: (wert) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(wert),
    nachricht: "Ungültige E-Mail-Adresse"
  })
  .feldRegel("passwort", {
    validate: (wert) => wert.length >= 8,
    nachricht: "Passwort muss mindestens 8 Zeichen lang sein"
  })
  .feldRegel("passwort", {
    validate: (wert) => /[A-Z]/.test(wert) && /[0-9]/.test(wert),
    nachricht: "Passwort muss Großbuchstaben und Zahlen enthalten"
  })
  .feldRegel("alter", {
    validate: (wert) => wert >= 18,
    nachricht: "Sie müssen mindestens 18 Jahre alt sein"
  });

// Formular ausfüllen
form.setzeWert("benutzername", "max");
form.setzeWert("email", "max@example.com");
form.setzeWert("passwort", "Pass123");
form.setzeWert("alter", 25);

// Validieren
if (form.validiereAlles()) {
  console.log("Formular ist gültig!");
  console.log("Werte:", form.getWerte());
} else {
  console.log("Formular hat Fehler:");
  console.log(form.getAlleFehler());
}
```

---

## Zusammenfassung

Du hast nun die Grundlagen von TypeScript gelernt:

✅ **Typsystem**: Primitive Typen, Arrays, Tupel, Enums
✅ **Funktionen**: Parameter, Rückgabewerte, Überladung
✅ **Objekte**: Interfaces, Type Aliases
✅ **Klassen**: Properties, Methoden, Vererbung, Access Modifiers
✅ **Generics**: Flexible, wiederverwendbare Komponenten
✅ **Type Guards**: Typsichere Verzweigungen
✅ **Union & Intersection**: Komplexe Typkombinationen

### Nächste Schritte

1. **Übung macht den Meister**: Implementiere eigene Projekte
2. **TypeScript mit Frameworks**: Lerne Next.js, React, oder Node.js mit TypeScript
3. **Advanced Topics**: Mapped Types, Conditional Types, Template Literal Types
4. **Best Practices**: Lerne über Code-Organisation und Design Patterns

### Hilfreiche Ressourcen

- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [TypeScript Playground](https://www.typescriptlang.org/play)
- [DefinitelyTyped](https://github.com/DefinitelyTyped/DefinitelyTyped) - Type Definitionen für JavaScript-Bibliotheken

Viel Erfolg beim Lernen! 🚀
