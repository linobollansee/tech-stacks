# Fortgeschrittenes TypeScript - Kompletter Walkthrough

## Inhaltsverzeichnis
1. [Utility Types](#utility-types)
2. [Advanced Types](#advanced-types)
3. [Conditional Types](#conditional-types)
4. [Mapped Types](#mapped-types)
5. [Template Literal Types](#template-literal-types)
6. [Type Guards & Type Predicates](#type-guards--type-predicates)
7. [Discriminated Unions](#discriminated-unions)
8. [Generics Deep Dive](#generics-deep-dive)
9. [Type Inference & infer Keyword](#type-inference--infer-keyword)
10. [Decorators](#decorators)
11. [Advanced Patterns](#advanced-patterns)
12. [Performance & Optimization](#performance--optimization)
13. [Type-Level Programming](#type-level-programming)
14. [Best Practices](#best-practices)
15. [Praktische Projekte](#praktische-projekte)

---

## Utility Types

### Built-in Utility Types

TypeScript bietet viele eingebaute Utility Types für häufige Transformationen.

#### Partial<T>

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

// Alle Properties optional
type PartialUser = Partial<User>;
// { id?: number; name?: string; email?: string; age?: number; }

function updateUser(id: number, updates: Partial<User>) {
  // Nur geänderte Felder übergeben
}

updateUser(1, { name: 'Max' });  // ✅
updateUser(2, { email: 'max@example.com', age: 25 });  // ✅
```

#### Required<T>

```typescript
interface Config {
  host?: string;
  port?: number;
  ssl?: boolean;
}

// Alle Properties erforderlich
type RequiredConfig = Required<Config>;
// { host: string; port: number; ssl: boolean; }

function validateConfig(config: Required<Config>) {
  // Alle Felder müssen vorhanden sein
}
```

#### Readonly<T>

```typescript
interface User {
  id: number;
  name: string;
}

const user: Readonly<User> = {
  id: 1,
  name: 'Max'
};

user.id = 2;  // ❌ Error: Cannot assign to 'id'
user.name = 'Anna';  // ❌ Error: Cannot assign to 'name'
```

#### Record<K, T>

```typescript
// Objekt mit bestimmten Keys und Value-Types
type Role = 'admin' | 'user' | 'guest';

type Permissions = Record<Role, string[]>;
// { admin: string[]; user: string[]; guest: string[]; }

const permissions: Permissions = {
  admin: ['read', 'write', 'delete'],
  user: ['read', 'write'],
  guest: ['read']
};

// Mit Union Type Keys
type UserRecord = Record<string, User>;
const users: UserRecord = {
  'user1': { id: 1, name: 'Max', email: 'max@example.com', age: 25 },
  'user2': { id: 2, name: 'Anna', email: 'anna@example.com', age: 30 }
};
```

#### Pick<T, K>

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
  createdAt: Date;
}

// Nur bestimmte Properties auswählen
type UserPreview = Pick<User, 'id' | 'name' | 'email'>;
// { id: number; name: string; email: string; }

type LoginData = Pick<User, 'email' | 'password'>;
// { email: string; password: string; }
```

#### Omit<T, K>

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

// Bestimmte Properties ausschließen
type UserWithoutPassword = Omit<User, 'password'>;
// { id: number; name: string; email: string; }

type CreateUserDto = Omit<User, 'id'>;
// { name: string; email: string; password: string; }
```

#### Exclude<T, U> & Extract<T, U>

```typescript
type T1 = 'a' | 'b' | 'c' | 'd';

// Exclude: Entfernt Types aus Union
type T2 = Exclude<T1, 'a' | 'c'>;  // 'b' | 'd'

// Extract: Behält nur bestimmte Types
type T3 = Extract<T1, 'a' | 'c'>;  // 'a' | 'c'

// Praktisches Beispiel
type AllEvents = 'click' | 'scroll' | 'mousemove' | 'keydown' | 'keyup';
type MouseEvents = Extract<AllEvents, `mouse${string}`>;  // 'mousemove'
type KeyEvents = Extract<AllEvents, `key${string}`>;  // 'keydown' | 'keyup'
```

#### NonNullable<T>

```typescript
type T1 = string | number | null | undefined;
type T2 = NonNullable<T1>;  // string | number

function processValue(value: string | null | undefined) {
  // Nach Check ist Type NonNullable
  if (value) {
    const nonNull: NonNullable<typeof value> = value;
    console.log(nonNull.toUpperCase());
  }
}
```

#### ReturnType<T> & Parameters<T>

```typescript
function getUser(id: number) {
  return { id, name: 'Max', email: 'max@example.com' };
}

// Return Type extrahieren
type User = ReturnType<typeof getUser>;
// { id: number; name: string; email: string; }

// Parameter Types extrahieren
type GetUserParams = Parameters<typeof getUser>;
// [id: number]

function createUser(name: string, email: string, age?: number) {
  return { name, email, age };
}

type CreateUserParams = Parameters<typeof createUser>;
// [name: string, email: string, age?: number | undefined]
```

#### Awaited<T>

```typescript
// Extrahiert Type aus Promise
type T1 = Awaited<Promise<string>>;  // string
type T2 = Awaited<Promise<Promise<number>>>;  // number

async function getUser(): Promise<User> {
  return { id: 1, name: 'Max', email: 'max@example.com', age: 25 };
}

type UserType = Awaited<ReturnType<typeof getUser>>;
// User
```

---

## Advanced Types

### Union Types

```typescript
type Status = 'pending' | 'success' | 'error';
type ID = string | number;

function handleStatus(status: Status) {
  switch (status) {
    case 'pending':
      console.log('Loading...');
      break;
    case 'success':
      console.log('Done!');
      break;
    case 'error':
      console.log('Failed!');
      break;
  }
}
```

### Intersection Types

```typescript
interface HasId {
  id: number;
}

interface HasTimestamps {
  createdAt: Date;
  updatedAt: Date;
}

interface HasName {
  name: string;
}

// Kombiniert alle Properties
type User = HasId & HasName & HasTimestamps;
// { id: number; name: string; createdAt: Date; updatedAt: Date; }

const user: User = {
  id: 1,
  name: 'Max',
  createdAt: new Date(),
  updatedAt: new Date()
};
```

### Index Signatures

```typescript
interface StringMap {
  [key: string]: string;
}

const translations: StringMap = {
  hello: 'Hallo',
  goodbye: 'Auf Wiedersehen'
};

// Mit bestimmten Keys und dynamischen Keys
interface User {
  id: number;
  name: string;
  [key: string]: any;  // Zusätzliche dynamische Properties
}

const user: User = {
  id: 1,
  name: 'Max',
  age: 25,  // ✅
  email: 'max@example.com'  // ✅
};
```

### Tuple Types

```typescript
// Basic Tuple
type Point = [number, number];
const point: Point = [10, 20];

// Named Tuples
type Point3D = [x: number, y: number, z: number];
const point3d: Point3D = [10, 20, 30];

// Optional Elements
type OptionalTuple = [string, number?];
const t1: OptionalTuple = ['hello'];  // ✅
const t2: OptionalTuple = ['hello', 42];  // ✅

// Rest Elements
type StringNumberBooleans = [string, number, ...boolean[]];
const t3: StringNumberBooleans = ['hello', 42, true, false, true];

// Readonly Tuples
type ReadonlyPoint = readonly [number, number];
const point2: ReadonlyPoint = [10, 20];
point2[0] = 15;  // ❌ Error: Cannot assign to '0'
```

---

## Conditional Types

### Basis Syntax

```typescript
// T extends U ? X : Y
type IsString<T> = T extends string ? true : false;

type T1 = IsString<string>;  // true
type T2 = IsString<number>;  // false

// Mit Union Types (Distributive)
type T3 = IsString<string | number>;  // boolean (true | false)
```

### Praktische Beispiele

```typescript
// Extract Array Element Type
type ArrayElement<T> = T extends (infer U)[] ? U : never;

type T1 = ArrayElement<string[]>;  // string
type T2 = ArrayElement<number[]>;  // number
type T3 = ArrayElement<string>;    // never

// Flatten Type
type Flatten<T> = T extends any[] ? T[number] : T;

type T4 = Flatten<string[]>;  // string
type T5 = Flatten<number>;    // number

// Exclude null and undefined
type NonNullish<T> = T extends null | undefined ? never : T;

type T6 = NonNullish<string | null>;  // string
type T7 = NonNullish<number | undefined>;  // number

// Function Type Check
type IsFunction<T> = T extends (...args: any[]) => any ? true : false;

type T8 = IsFunction<() => void>;  // true
type T9 = IsFunction<string>;  // false
```

### Nested Conditionals

```typescript
type TypeName<T> = 
  T extends string ? 'string' :
  T extends number ? 'number' :
  T extends boolean ? 'boolean' :
  T extends undefined ? 'undefined' :
  T extends Function ? 'function' :
  'object';

type T1 = TypeName<string>;  // 'string'
type T2 = TypeName<42>;  // 'number'
type T3 = TypeName<() => void>;  // 'function'
```

### Distributive Conditional Types

```typescript
type ToArray<T> = T extends any ? T[] : never;

// Union Types werden distribuiert
type T1 = ToArray<string | number>;
// ToArray<string> | ToArray<number>
// string[] | number[]

// Non-Distributive mit []
type ToArrayNonDist<T> = [T] extends [any] ? T[] : never;

type T2 = ToArrayNonDist<string | number>;
// (string | number)[]
```

---

## Mapped Types

### Basis Syntax

```typescript
type Mapped<T> = {
  [K in keyof T]: T[K];
};

interface User {
  id: number;
  name: string;
}

type MappedUser = Mapped<User>;
// { id: number; name: string; }
```

### Modifiers

```typescript
// Optional
type Optional<T> = {
  [K in keyof T]?: T[K];
};

// Readonly
type ReadonlyType<T> = {
  readonly [K in keyof T]: T[K];
};

// Remove Optional
type Concrete<T> = {
  [K in keyof T]-?: T[K];
};

// Remove Readonly
type Mutable<T> = {
  -readonly [K in keyof T]: T[K];
};
```

### Key Remapping

```typescript
// Keys transformieren
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface User {
  name: string;
  age: number;
}

type UserGetters = Getters<User>;
// {
//   getName: () => string;
//   getAge: () => number;
// }

// Keys filtern
type RemoveIdFields<T> = {
  [K in keyof T as K extends `${string}Id` ? never : K]: T[K];
};

interface Data {
  id: number;
  userId: number;
  name: string;
  postId: number;
}

type Filtered = RemoveIdFields<Data>;
// { name: string; }
```

### Praktische Mapped Types

```typescript
// Nullable Type
type Nullable<T> = {
  [K in keyof T]: T[K] | null;
};

// Deep Partial
type DeepPartial<T> = {
  [K in keyof T]?: T[K] extends object ? DeepPartial<T[K]> : T[K];
};

interface User {
  id: number;
  profile: {
    name: string;
    address: {
      street: string;
      city: string;
    };
  };
}

type PartialUser = DeepPartial<User>;
// {
//   id?: number;
//   profile?: {
//     name?: string;
//     address?: {
//       street?: string;
//       city?: string;
//     };
//   };
// }

// Promisify
type Promisify<T> = {
  [K in keyof T]: T[K] extends (...args: any[]) => any
    ? (...args: Parameters<T[K]>) => Promise<ReturnType<T[K]>>
    : T[K];
};

interface API {
  getUser: (id: number) => User;
  deleteUser: (id: number) => boolean;
}

type AsyncAPI = Promisify<API>;
// {
//   getUser: (id: number) => Promise<User>;
//   deleteUser: (id: number) => Promise<boolean>;
// }
```

---

## Template Literal Types

### Basis

```typescript
type World = 'world';
type Greeting = `hello ${World}`;  // 'hello world'

// Mit Union Types
type Size = 'small' | 'medium' | 'large';
type Color = 'red' | 'green' | 'blue';

type SizeColor = `${Size}-${Color}`;
// 'small-red' | 'small-green' | 'small-blue' | 'medium-red' | ...
```

### Intrinsic String Manipulation

```typescript
// Uppercase
type Upper = Uppercase<'hello'>;  // 'HELLO'

// Lowercase
type Lower = Lowercase<'HELLO'>;  // 'hello'

// Capitalize
type Cap = Capitalize<'hello'>;  // 'Hello'

// Uncapitalize
type Uncap = Uncapitalize<'Hello'>;  // 'hello'

// Kombiniert
type EventName<T extends string> = `on${Capitalize<T>}`;
type ClickEvent = EventName<'click'>;  // 'onClick'
type KeyPressEvent = EventName<'keyPress'>;  // 'onKeyPress'
```

### Praktische Beispiele

```typescript
// CSS Properties
type CSSProperty = 'color' | 'background' | 'font-size';
type CSSVariable = `--${CSSProperty}`;
// '--color' | '--background' | '--font-size'

// HTTP Methods
type HTTPMethod = 'GET' | 'POST' | 'PUT' | 'DELETE';
type Endpoint = '/users' | '/posts';
type Route = `${HTTPMethod} ${Endpoint}`;
// 'GET /users' | 'GET /posts' | 'POST /users' | ...

// Event Handlers
type EventType = 'click' | 'scroll' | 'keypress';
type EventHandler = `on${Capitalize<EventType>}`;
// 'onClick' | 'onScroll' | 'onKeypress'

interface Events {
  onClick: (e: MouseEvent) => void;
  onScroll: (e: ScrollEvent) => void;
  onKeypress: (e: KeyboardEvent) => void;
}

// Path Builder
type Path = 'home' | 'about' | 'contact';
type Route2 = `/${Path}`;
// '/home' | '/about' | '/contact'
```

### Advanced Pattern

```typescript
// Type-safe Route Builder
type HTTPMethodPath<
  Method extends string,
  Path extends string
> = `${Method} ${Path}`;

type API = {
  'GET /users': { response: User[] };
  'GET /users/:id': { params: { id: string }; response: User };
  'POST /users': { body: CreateUserDto; response: User };
  'DELETE /users/:id': { params: { id: string }; response: void };
};

function request<T extends keyof API>(
  route: T,
  options?: API[T] extends { body: infer B } ? { body: B } : {}
): Promise<API[T] extends { response: infer R } ? R : never> {
  // Implementation
  return null as any;
}

// Type-safe usage
const users = await request('GET /users');  // User[]
const user = await request('GET /users/:id');  // User
const newUser = await request('POST /users', {
  body: { name: 'Max', email: 'max@example.com', password: 'secret' }
});  // User
```

---

## Type Guards & Type Predicates

### typeof Type Guards

```typescript
function processValue(value: string | number) {
  if (typeof value === 'string') {
    // TypeScript weiß: value ist string
    console.log(value.toUpperCase());
  } else {
    // TypeScript weiß: value ist number
    console.log(value.toFixed(2));
  }
}
```

### instanceof Type Guards

```typescript
class Dog {
  bark() {
    console.log('Woof!');
  }
}

class Cat {
  meow() {
    console.log('Meow!');
  }
}

function makeSound(animal: Dog | Cat) {
  if (animal instanceof Dog) {
    animal.bark();
  } else {
    animal.meow();
  }
}
```

### User-Defined Type Guards

```typescript
interface User {
  id: number;
  name: string;
}

interface Admin {
  id: number;
  name: string;
  permissions: string[];
}

// Type Predicate: value is Type
function isAdmin(user: User | Admin): user is Admin {
  return 'permissions' in user;
}

function processUser(user: User | Admin) {
  if (isAdmin(user)) {
    // TypeScript weiß: user ist Admin
    console.log(user.permissions);
  } else {
    // TypeScript weiß: user ist User
    console.log(user.name);
  }
}
```

### in Operator

```typescript
interface Bird {
  fly: () => void;
  layEggs: () => void;
}

interface Fish {
  swim: () => void;
  layEggs: () => void;
}

function move(animal: Bird | Fish) {
  if ('fly' in animal) {
    animal.fly();
  } else {
    animal.swim();
  }
}
```

### Discriminated Unions (Type Guards)

```typescript
interface Circle {
  kind: 'circle';
  radius: number;
}

interface Square {
  kind: 'square';
  size: number;
}

interface Rectangle {
  kind: 'rectangle';
  width: number;
  height: number;
}

type Shape = Circle | Square | Rectangle;

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case 'circle':
      return Math.PI * shape.radius ** 2;
    case 'square':
      return shape.size ** 2;
    case 'rectangle':
      return shape.width * shape.height;
  }
}
```

### Assertion Functions

```typescript
function assert(condition: any, msg?: string): asserts condition {
  if (!condition) {
    throw new Error(msg || 'Assertion failed');
  }
}

function processValue(value: string | null) {
  assert(value !== null, 'Value cannot be null');
  // Nach assert weiß TypeScript: value ist string
  console.log(value.toUpperCase());
}

// Mit Type Predicate
function assertIsString(value: any): asserts value is string {
  if (typeof value !== 'string') {
    throw new Error('Value is not a string');
  }
}

function processUnknown(value: unknown) {
  assertIsString(value);
  // Nach assert weiß TypeScript: value ist string
  console.log(value.toUpperCase());
}
```

---

## Discriminated Unions

### Basis Pattern

```typescript
interface Success<T> {
  success: true;
  data: T;
}

interface Failure {
  success: false;
  error: string;
}

type Result<T> = Success<T> | Failure;

function handleResult(result: Result<User>) {
  if (result.success) {
    // TypeScript weiß: result ist Success<User>
    console.log(result.data.name);
  } else {
    // TypeScript weiß: result ist Failure
    console.log(result.error);
  }
}
```

### Action Pattern (Redux-style)

```typescript
interface AddTodoAction {
  type: 'ADD_TODO';
  payload: {
    text: string;
  };
}

interface RemoveTodoAction {
  type: 'REMOVE_TODO';
  payload: {
    id: number;
  };
}

interface ToggleTodoAction {
  type: 'TOGGLE_TODO';
  payload: {
    id: number;
  };
}

type TodoAction = AddTodoAction | RemoveTodoAction | ToggleTodoAction;

function reducer(state: Todo[], action: TodoAction): Todo[] {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, { id: Date.now(), text: action.payload.text, done: false }];
    case 'REMOVE_TODO':
      return state.filter(todo => todo.id !== action.payload.id);
    case 'TOGGLE_TODO':
      return state.map(todo =>
        todo.id === action.payload.id ? { ...todo, done: !todo.done } : todo
      );
  }
}
```

### Exhaustive Checks

```typescript
type Status = 'pending' | 'success' | 'error';

function assertNever(value: never): never {
  throw new Error(`Unhandled value: ${value}`);
}

function handleStatus(status: Status) {
  switch (status) {
    case 'pending':
      return 'Loading...';
    case 'success':
      return 'Done!';
    case 'error':
      return 'Failed!';
    default:
      // Falls ein neuer Status hinzugefügt wird, gibt es hier einen Compile Error
      return assertNever(status);
  }
}
```

---

## Generics Deep Dive

### Multiple Type Parameters

```typescript
function merge<T, U>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 };
}

const result = merge({ name: 'Max' }, { age: 25 });
// { name: string; age: number; }
```

### Generic Constraints

```typescript
interface HasLength {
  length: number;
}

function getLength<T extends HasLength>(value: T): number {
  return value.length;
}

getLength('hello');  // ✅ string has length
getLength([1, 2, 3]);  // ✅ array has length
getLength({ length: 10 });  // ✅ object has length
getLength(42);  // ❌ Error: number has no length

// Mit keyof
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { id: 1, name: 'Max', age: 25 };
const name = getProperty(user, 'name');  // string
const age = getProperty(user, 'age');  // number
getProperty(user, 'invalid');  // ❌ Error
```

### Default Type Parameters

```typescript
interface APIResponse<T = any> {
  data: T;
  status: number;
}

const response1: APIResponse = { data: 'anything', status: 200 };
const response2: APIResponse<User> = { data: user, status: 200 };

// Mit Constraints
interface Container<T extends object = {}> {
  value: T;
}

const c1: Container = { value: {} };
const c2: Container<User> = { value: user };
```

### Generic Classes

```typescript
class GenericNumber<T extends number | string> {
  zeroValue: T;
  add: (x: T, y: T) => T;

  constructor(zeroValue: T, add: (x: T, y: T) => T) {
    this.zeroValue = zeroValue;
    this.add = add;
  }
}

const numCalculator = new GenericNumber<number>(
  0,
  (x, y) => x + y
);

const stringCalculator = new GenericNumber<string>(
  '',
  (x, y) => x + y
);
```

### Generic Utility Implementation

```typescript
// Custom Partial Implementation
type MyPartial<T> = {
  [K in keyof T]?: T[K];
};

// Custom Pick Implementation
type MyPick<T, K extends keyof T> = {
  [P in K]: T[P];
};

// Custom Omit Implementation
type MyOmit<T, K extends keyof T> = MyPick<T, Exclude<keyof T, K>>;

// Custom Record Implementation
type MyRecord<K extends keyof any, T> = {
  [P in K]: T;
};
```

---

## Type Inference & infer Keyword

### Return Type Inference

```typescript
function createUser(name: string, age: number) {
  return { name, age, id: Math.random() };
}

// TypeScript inferiert Return Type
type User = ReturnType<typeof createUser>;
// { name: string; age: number; id: number; }
```

### infer Keyword

```typescript
// Array Element Type extrahieren
type ArrayElement<T> = T extends (infer E)[] ? E : never;

type T1 = ArrayElement<string[]>;  // string
type T2 = ArrayElement<number[]>;  // number

// Promise Value Type extrahieren
type UnwrapPromise<T> = T extends Promise<infer U> ? U : T;

type T3 = UnwrapPromise<Promise<string>>;  // string
type T4 = UnwrapPromise<number>;  // number

// Function Return Type
type GetReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function getUser() {
  return { id: 1, name: 'Max' };
}

type UserType = GetReturnType<typeof getUser>;
// { id: number; name: string; }

// Function Parameters
type GetParameters<T> = T extends (...args: infer P) => any ? P : never;

function createUser(name: string, email: string, age?: number) {}

type Params = GetParameters<typeof createUser>;
// [name: string, email: string, age?: number | undefined]
```

### Advanced infer Patterns

```typescript
// Tuple First Element
type First<T extends any[]> = T extends [infer F, ...any[]] ? F : never;

type T1 = First<[string, number, boolean]>;  // string

// Tuple Last Element
type Last<T extends any[]> = T extends [...any[], infer L] ? L : never;

type T2 = Last<[string, number, boolean]>;  // boolean

// Constructor Parameters
type ConstructorParams<T> = T extends new (...args: infer P) => any ? P : never;

class User {
  constructor(public name: string, public age: number) {}
}

type UserConstructorParams = ConstructorParams<typeof User>;
// [name: string, age: number]

// Instance Type
type InstanceType2<T> = T extends new (...args: any[]) => infer R ? R : never;

type UserInstance = InstanceType2<typeof User>;
// User
```

---

## Decorators

### Class Decorators

```typescript
function sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

@sealed
class User {
  constructor(public name: string) {}
}

// Class Decorator mit Factory
function component(name: string) {
  return function (constructor: Function) {
    console.log(`Component ${name} registered`);
    (constructor as any).componentName = name;
  };
}

@component('UserComponent')
class UserComponent {}
```

### Method Decorators

```typescript
function log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;

  descriptor.value = function (...args: any[]) {
    console.log(`Calling ${propertyKey} with`, args);
    const result = originalMethod.apply(this, args);
    console.log(`Result:`, result);
    return result;
  };

  return descriptor;
}

class Calculator {
  @log
  add(a: number, b: number) {
    return a + b;
  }
}

const calc = new Calculator();
calc.add(2, 3);
// Calling add with [2, 3]
// Result: 5
```

### Property Decorators

```typescript
function required(target: any, propertyKey: string) {
  let value: any;

  const getter = () => value;
  const setter = (newValue: any) => {
    if (!newValue) {
      throw new Error(`${propertyKey} is required`);
    }
    value = newValue;
  };

  Object.defineProperty(target, propertyKey, {
    get: getter,
    set: setter,
    enumerable: true,
    configurable: true
  });
}

class User {
  @required
  name: string;

  constructor(name: string) {
    this.name = name;
  }
}

const user = new User('');  // Error: name is required
```

### Parameter Decorators

```typescript
function validate(target: any, propertyKey: string, parameterIndex: number) {
  const existingParams: number[] = Reflect.getOwnMetadata('validate', target, propertyKey) || [];
  existingParams.push(parameterIndex);
  Reflect.defineMetadata('validate', existingParams, target, propertyKey);
}

class UserService {
  createUser(@validate name: string, @validate email: string) {
    // Validation logic
    return { name, email };
  }
}
```

### Decorator Composition

```typescript
function first() {
  console.log('first(): factory');
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log('first(): called');
  };
}

function second() {
  console.log('second(): factory');
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log('second(): called');
  };
}

class Example {
  @first()
  @second()
  method() {}
}

// Output:
// first(): factory
// second(): factory
// second(): called
// first(): called
```

---

## Advanced Patterns

### Builder Pattern

```typescript
class User {
  constructor(
    public name: string,
    public email: string,
    public age?: number,
    public address?: string
  ) {}
}

class UserBuilder {
  private name: string = '';
  private email: string = '';
  private age?: number;
  private address?: string;

  setName(name: string): this {
    this.name = name;
    return this;
  }

  setEmail(email: string): this {
    this.email = email;
    return this;
  }

  setAge(age: number): this {
    this.age = age;
    return this;
  }

  setAddress(address: string): this {
    this.address = address;
    return this;
  }

  build(): User {
    return new User(this.name, this.email, this.age, this.address);
  }
}

const user = new UserBuilder()
  .setName('Max')
  .setEmail('max@example.com')
  .setAge(25)
  .build();
```

### Fluent API

```typescript
class QueryBuilder<T> {
  private conditions: string[] = [];
  private sortField?: keyof T;
  private limitValue?: number;

  where(field: keyof T, operator: string, value: any): this {
    this.conditions.push(`${String(field)} ${operator} ${value}`);
    return this;
  }

  orderBy(field: keyof T): this {
    this.sortField = field;
    return this;
  }

  limit(value: number): this {
    this.limitValue = value;
    return this;
  }

  build(): string {
    let query = 'SELECT * FROM table';
    
    if (this.conditions.length > 0) {
      query += ' WHERE ' + this.conditions.join(' AND ');
    }
    
    if (this.sortField) {
      query += ` ORDER BY ${String(this.sortField)}`;
    }
    
    if (this.limitValue) {
      query += ` LIMIT ${this.limitValue}`;
    }
    
    return query;
  }
}

interface User {
  id: number;
  name: string;
  age: number;
}

const query = new QueryBuilder<User>()
  .where('age', '>', 18)
  .where('name', 'LIKE', '%Max%')
  .orderBy('age')
  .limit(10)
  .build();
```

### Type-Safe Event Emitter

```typescript
type EventMap = {
  'user:created': { id: number; name: string };
  'user:updated': { id: number; changes: Partial<User> };
  'user:deleted': { id: number };
};

class TypedEventEmitter<T extends Record<string, any>> {
  private listeners: {
    [K in keyof T]?: Array<(data: T[K]) => void>;
  } = {};

  on<K extends keyof T>(event: K, listener: (data: T[K]) => void): void {
    if (!this.listeners[event]) {
      this.listeners[event] = [];
    }
    this.listeners[event]!.push(listener);
  }

  emit<K extends keyof T>(event: K, data: T[K]): void {
    const listeners = this.listeners[event];
    if (listeners) {
      listeners.forEach(listener => listener(data));
    }
  }

  off<K extends keyof T>(event: K, listener: (data: T[K]) => void): void {
    const listeners = this.listeners[event];
    if (listeners) {
      const index = listeners.indexOf(listener);
      if (index !== -1) {
        listeners.splice(index, 1);
      }
    }
  }
}

const emitter = new TypedEventEmitter<EventMap>();

emitter.on('user:created', (data) => {
  // data ist type-safe: { id: number; name: string }
  console.log(`User ${data.name} created with ID ${data.id}`);
});

emitter.emit('user:created', { id: 1, name: 'Max' });
```

### Repository Pattern

```typescript
interface Repository<T> {
  find(id: number): Promise<T | null>;
  findAll(): Promise<T[]>;
  create(data: Omit<T, 'id'>): Promise<T>;
  update(id: number, data: Partial<T>): Promise<T>;
  delete(id: number): Promise<void>;
}

class UserRepository implements Repository<User> {
  private users: User[] = [];

  async find(id: number): Promise<User | null> {
    return this.users.find(u => u.id === id) || null;
  }

  async findAll(): Promise<User[]> {
    return this.users;
  }

  async create(data: Omit<User, 'id'>): Promise<User> {
    const user: User = {
      id: this.users.length + 1,
      ...data
    };
    this.users.push(user);
    return user;
  }

  async update(id: number, data: Partial<User>): Promise<User> {
    const index = this.users.findIndex(u => u.id === id);
    if (index === -1) {
      throw new Error('User not found');
    }
    this.users[index] = { ...this.users[index], ...data };
    return this.users[index];
  }

  async delete(id: number): Promise<void> {
    const index = this.users.findIndex(u => u.id === id);
    if (index !== -1) {
      this.users.splice(index, 1);
    }
  }
}
```

---

## Performance & Optimization

### Type Instantiation Depth

```typescript
// ❌ Zu tiefe Rekursion
type DeepArray<T, N extends number> = N extends 0
  ? T
  : DeepArray<T[], Subtract<N, 1>>;

// ✅ Besser: Limit setzen
type SafeDeepArray<T, N extends number> = N extends 0
  ? T
  : N extends 50
  ? any
  : SafeDeepArray<T[], Subtract<N, 1>>;
```

### Avoid Expensive Computations

```typescript
// ❌ Teuer bei großen Unions
type AllCombinations<T extends string> = T extends any
  ? `${T}-${AllCombinations<Exclude<T, T>>}`
  : never;

// ✅ Besser: Nur wenn nötig
type LimitedCombinations<T extends string> = T extends 'a' | 'b' | 'c'
  ? `${T}-something`
  : never;
```

### Use as const

```typescript
// ❌ Ohne as const
const colors = ['red', 'green', 'blue'];
// Type: string[]

// ✅ Mit as const
const colors2 = ['red', 'green', 'blue'] as const;
// Type: readonly ['red', 'green', 'blue']

type Color = typeof colors2[number];
// 'red' | 'green' | 'blue'
```

---

## Type-Level Programming

### Arithmetic

```typescript
// Increment
type Inc<N extends number> = [never, 0, 1, 2, 3, 4, 5][N & 7];

type T1 = Inc<0>;  // 1
type T2 = Inc<3>;  // 4

// Tuple Length
type Length<T extends any[]> = T['length'];

type T3 = Length<[1, 2, 3]>;  // 3
```

### Tuple Operations

```typescript
// Push
type Push<T extends any[], V> = [...T, V];

type T1 = Push<[1, 2], 3>;  // [1, 2, 3]

// Pop
type Pop<T extends any[]> = T extends [...infer Rest, any] ? Rest : [];

type T2 = Pop<[1, 2, 3]>;  // [1, 2]

// Shift
type Shift<T extends any[]> = T extends [any, ...infer Rest] ? Rest : [];

type T3 = Shift<[1, 2, 3]>;  // [2, 3]

// Unshift
type Unshift<T extends any[], V> = [V, ...T];

type T4 = Unshift<[2, 3], 1>;  // [1, 2, 3]
```

### String Operations

```typescript
// Split String
type Split<S extends string, D extends string> =
  S extends `${infer T}${D}${infer U}` ? [T, ...Split<U, D>] : [S];

type T1 = Split<'a-b-c', '-'>;  // ['a', 'b', 'c']

// Join
type Join<T extends string[], D extends string> =
  T extends [infer F extends string, ...infer R extends string[]]
    ? R extends []
      ? F
      : `${F}${D}${Join<R, D>}`
    : '';

type T2 = Join<['a', 'b', 'c'], '-'>;  // 'a-b-c'

// Replace
type Replace<
  S extends string,
  From extends string,
  To extends string
> = S extends `${infer L}${From}${infer R}`
  ? `${L}${To}${R}`
  : S;

type T3 = Replace<'hello world', 'world', 'TypeScript'>;
// 'hello TypeScript'
```

---

## Best Practices

### 1. Prefer Types over Interfaces für Unions

```typescript
// ✅ Type
type Status = 'pending' | 'success' | 'error';

// ❌ Interface kann keine Union sein
interface Status {
  // ...
}
```

### 2. Use unknown instead of any

```typescript
// ❌ any
function process(data: any) {
  return data.value;  // Keine Type-Sicherheit
}

// ✅ unknown
function process2(data: unknown) {
  if (typeof data === 'object' && data !== null && 'value' in data) {
    return (data as { value: any }).value;
  }
}
```

### 3. Exhaustive Type Checking

```typescript
function assertNever(x: never): never {
  throw new Error('Unexpected value: ' + x);
}

type Status = 'pending' | 'success' | 'error';

function handleStatus(status: Status) {
  switch (status) {
    case 'pending':
      return 'Loading...';
    case 'success':
      return 'Done!';
    case 'error':
      return 'Failed!';
    default:
      return assertNever(status);  // Compile Error wenn Status erweitert wird
  }
}
```

### 4. Use Const Assertions

```typescript
// ✅ Const Assertion
const config = {
  apiUrl: 'https://api.example.com',
  timeout: 5000
} as const;

// config.apiUrl ist jetzt 'https://api.example.com' statt string
```

### 5. Avoid Type Assertions

```typescript
// ❌ Type Assertion
const value = getData() as string;

// ✅ Type Guard
function isString(value: unknown): value is string {
  return typeof value === 'string';
}

const value2 = getData();
if (isString(value2)) {
  // value2 ist jetzt string
}
```

---

## Praktische Projekte

### Projekt 1: Type-Safe Form Builder

```typescript
type FormField<T> = {
  value: T;
  required: boolean;
  validators: Array<(value: T) => string | null>;
};

type FormSchema = Record<string, FormField<any>>;

type FormValues<T extends FormSchema> = {
  [K in keyof T]: T[K]['value'];
};

type FormErrors<T extends FormSchema> = {
  [K in keyof T]?: string;
};

class FormBuilder<T extends FormSchema> {
  private schema: T;
  private values: Partial<FormValues<T>> = {};

  constructor(schema: T) {
    this.schema = schema;
  }

  setValue<K extends keyof T>(field: K, value: T[K]['value']): this {
    this.values[field] = value;
    return this;
  }

  getValue<K extends keyof T>(field: K): T[K]['value'] | undefined {
    return this.values[field];
  }

  validate(): FormErrors<T> {
    const errors: FormErrors<T> = {};

    for (const field in this.schema) {
      const fieldSchema = this.schema[field];
      const value = this.values[field];

      if (fieldSchema.required && !value) {
        errors[field] = 'This field is required';
        continue;
      }

      if (value) {
        for (const validator of fieldSchema.validators) {
          const error = validator(value);
          if (error) {
            errors[field] = error;
            break;
          }
        }
      }
    }

    return errors;
  }

  getValues(): Partial<FormValues<T>> {
    return this.values;
  }
}

// Usage
const userFormSchema = {
  name: {
    value: '' as string,
    required: true,
    validators: [
      (value: string) => value.length < 2 ? 'Name too short' : null
    ]
  },
  email: {
    value: '' as string,
    required: true,
    validators: [
      (value: string) => !value.includes('@') ? 'Invalid email' : null
    ]
  },
  age: {
    value: 0 as number,
    required: false,
    validators: [
      (value: number) => value < 18 ? 'Must be 18+' : null
    ]
  }
};

const form = new FormBuilder(userFormSchema)
  .setValue('name', 'Max')
  .setValue('email', 'max@example.com')
  .setValue('age', 25);

const errors = form.validate();
const values = form.getValues();
```

### Projekt 2: Type-Safe API Client

```typescript
type HTTPMethod = 'GET' | 'POST' | 'PUT' | 'DELETE' | 'PATCH';

type APIRoute = {
  method: HTTPMethod;
  path: string;
  params?: Record<string, string>;
  query?: Record<string, string | number | boolean>;
  body?: any;
  response: any;
};

type APISchema = Record<string, APIRoute>;

class APIClient<T extends APISchema> {
  constructor(private baseUrl: string) {}

  async request<K extends keyof T>(
    routeName: K,
    options: {
      params?: T[K]['params'];
      query?: T[K]['query'];
      body?: T[K]['body'];
    } = {}
  ): Promise<T[K]['response']> {
    const route = this.routes[routeName];
    
    let url = `${this.baseUrl}${route.path}`;

    // Replace params
    if (options.params) {
      for (const [key, value] of Object.entries(options.params)) {
        url = url.replace(`:${key}`, value);
      }
    }

    // Add query
    if (options.query) {
      const queryString = new URLSearchParams(
        options.query as any
      ).toString();
      url += `?${queryString}`;
    }

    const response = await fetch(url, {
      method: route.method,
      headers: {
        'Content-Type': 'application/json'
      },
      body: options.body ? JSON.stringify(options.body) : undefined
    });

    return response.json();
  }

  private routes: T = {} as T;
}

// API Definition
type MyAPI = {
  getUsers: {
    method: 'GET';
    path: '/users';
    query: { page?: number; limit?: number };
    response: User[];
  };
  getUser: {
    method: 'GET';
    path: '/users/:id';
    params: { id: string };
    response: User;
  };
  createUser: {
    method: 'POST';
    path: '/users';
    body: { name: string; email: string };
    response: User;
  };
  updateUser: {
    method: 'PUT';
    path: '/users/:id';
    params: { id: string };
    body: Partial<User>;
    response: User;
  };
};

const api = new APIClient<MyAPI>('https://api.example.com');

// Type-safe usage
const users = await api.request('getUsers', {
  query: { page: 1, limit: 10 }
});

const user = await api.request('getUser', {
  params: { id: '123' }
});

const newUser = await api.request('createUser', {
  body: { name: 'Max', email: 'max@example.com' }
});
```

---

## Zusammenfassung

Du hast nun gelernt:

✅ **Utility Types**: Partial, Required, Readonly, Record, Pick, Omit, Exclude, Extract, etc.
✅ **Advanced Types**: Union, Intersection, Index Signatures, Tuples
✅ **Conditional Types**: T extends U ? X : Y, Distributive Conditionals
✅ **Mapped Types**: Key Transformation, Modifiers, Key Remapping
✅ **Template Literal Types**: String Manipulation, Intrinsic Types
✅ **Type Guards**: typeof, instanceof, User-Defined, Assertion Functions
✅ **Discriminated Unions**: Pattern Matching, Exhaustive Checks
✅ **Generics**: Constraints, Default Parameters, Classes
✅ **infer Keyword**: Type Extraction, Advanced Patterns
✅ **Decorators**: Class, Method, Property, Parameter Decorators
✅ **Advanced Patterns**: Builder, Fluent API, Event Emitter, Repository
✅ **Performance**: Optimization, Type Instantiation Depth
✅ **Type-Level Programming**: Arithmetic, Tuple/String Operations
✅ **Best Practices**: Types vs Interfaces, unknown vs any, Const Assertions
✅ **Praktische Projekte**: Form Builder, API Client

### TypeScript Mastery Checklist

- [ ] Utility Types verstehen und anwenden
- [ ] Conditional Types selbst schreiben
- [ ] Mapped Types für Transformationen nutzen
- [ ] Template Literal Types verwenden
- [ ] Type Guards korrekt implementieren
- [ ] Discriminated Unions einsetzen
- [ ] Generics mit Constraints nutzen
- [ ] infer für Type Extraction verwenden
- [ ] Decorators verstehen
- [ ] Type-Safe APIs bauen

### Hilfreiche Ressourcen

- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/)
- [Type Challenges](https://github.com/type-challenges/type-challenges)
- [Total TypeScript](https://www.totaltypescript.com/)

Viel Erfolg mit fortgeschrittenem TypeScript! 🚀
