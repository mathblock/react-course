# Cheat Sheet JavaScript ES6+ & TypeScript pour React

## 📦 JavaScript ES6+ Essentiels

### Variables

```javascript
// let - variable réassignable, scope de bloc
let count = 0;
count = 1;  // ✅

// const - constante, non réassignable
const name = "Marie";
name = "Jean";  // ❌ Erreur !

// const avec objets/tableaux - la référence est constante, pas le contenu
const user = { name: "Marie" };
user.name = "Jean";  // ✅ Modification de propriété OK
user = {};  // ❌ Réassignation interdite
```

### Arrow Functions

```javascript
// Fonction classique
function add(a, b) {
  return a + b;
}

// Arrow function
const add = (a, b) => a + b;

// Avec plusieurs instructions
const multiply = (a, b) => {
  const result = a * b;
  return result;
};

// Sans paramètres
const greet = () => console.log("Hello");

// Un seul paramètre (parenthèses optionnelles)
const double = x => x * 2;

// Retour d'objet (entourer de parenthèses)
const makeUser = name => ({ name, age: 25 });
```

### Template Literals

```javascript
const name = "Marie";
const age = 25;

// Interpolation
const message = `Bonjour ${name}, vous avez ${age} ans`;

// Multi-lignes
const html = `
  <div>
    <h1>${name}</h1>
    <p>Age: ${age}</p>
  </div>
`;

// Expressions
const info = `Dans 5 ans, j'aurai ${age + 5} ans`;
```

### Destructuring

```javascript
// Objets
const user = { name: "Marie", age: 25, city: "Paris" };

const { name, age } = user;
console.log(name);  // "Marie"

// Avec renommage
const { name: userName, age: userAge } = user;

// Valeur par défaut
const { country = "France" } = user;

// Nested
const person = { 
  name: "Jean",
  address: { city: "Lyon", zip: "69000" }
};
const { address: { city } } = person;

// Tableaux
const fruits = ["pomme", "banane", "orange"];
const [first, second] = fruits;
console.log(first);  // "pomme"

// Skip des éléments
const [, , third] = fruits;
console.log(third);  // "orange"

// Rest operator
const [head, ...tail] = fruits;
console.log(tail);  // ["banane", "orange"]
```

### Spread Operator

```javascript
// Copie de tableau
const arr1 = [1, 2, 3];
const arr2 = [...arr1, 4, 5];  // [1, 2, 3, 4, 5]

// Fusion de tableaux
const merged = [...arr1, ...arr2];

// Copie d'objet
const user = { name: "Marie", age: 25 };
const userCopy = { ...user };

// Fusion d'objets
const address = { city: "Paris", country: "France" };
const fullUser = { ...user, ...address };

// Override de propriétés
const updated = { ...user, age: 26 };  // age devient 26

// Spread dans les paramètres de fonction
function sum(...numbers) {
  return numbers.reduce((a, b) => a + b, 0);
}
sum(1, 2, 3, 4);  // 10
```

### Méthodes de Tableaux

```javascript
const numbers = [1, 2, 3, 4, 5];

// map - Transformer chaque élément
const doubled = numbers.map(n => n * 2);
// [2, 4, 6, 8, 10]

// filter - Garder certains éléments
const evens = numbers.filter(n => n % 2 === 0);
// [2, 4]

// find - Trouver un élément
const found = numbers.find(n => n > 3);
// 4

// findIndex - Trouver l'index
const index = numbers.findIndex(n => n > 3);
// 3

// some - Au moins un élément vérifie la condition
const hasEven = numbers.some(n => n % 2 === 0);
// true

// every - Tous les éléments vérifient la condition
const allPositive = numbers.every(n => n > 0);
// true

// reduce - Réduire à une seule valeur
const sum = numbers.reduce((acc, n) => acc + n, 0);
// 15

// forEach - Itérer (pas de retour)
numbers.forEach(n => console.log(n));

// includes - Vérifier la présence
const hasThree = numbers.includes(3);
// true
```

### Promises & Async/Await

```javascript
// Promise
const fetchData = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve({ data: "Hello" });
    }, 1000);
  });
};

// Utilisation avec .then()
fetchData()
  .then(result => console.log(result))
  .catch(error => console.error(error));

// Async/Await
async function getData() {
  try {
    const result = await fetchData();
    console.log(result);
  } catch (error) {
    console.error(error);
  }
}

// Plusieurs appels parallèles
async function getMultiple() {
  const [users, posts] = await Promise.all([
    fetchUsers(),
    fetchPosts()
  ]);
}
```

### Optional Chaining & Nullish Coalescing

```javascript
// Optional chaining (?.)
const user = { name: "Marie", address: { city: "Paris" } };

// Sans optional chaining
const city = user && user.address && user.address.city;

// Avec optional chaining
const city = user?.address?.city;  // "Paris"
const zip = user?.address?.zip;    // undefined

// Nullish coalescing (??)
const value1 = null ?? "default";      // "default"
const value2 = undefined ?? "default"; // "default"
const value3 = 0 ?? "default";         // 0
const value4 = "" ?? "default";        // ""

// Différence avec ||
const value5 = 0 || "default";     // "default"
const value6 = 0 ?? "default";     // 0
```

---

## 🔷 TypeScript Essentiels

### Types de Base

```typescript
// Types primitifs
let name: string = "Marie";
let age: number = 25;
let isActive: boolean = true;
let nothing: null = null;
let notDefined: undefined = undefined;

// Tableaux
let numbers: number[] = [1, 2, 3];
let strings: Array<string> = ["a", "b", "c"];

// Tuples
let tuple: [string, number] = ["Marie", 25];

// Any (à éviter)
let anything: any = "whatever";

// Unknown (préférer à any)
let value: unknown = "hello";
if (typeof value === "string") {
  console.log(value.toUpperCase());  // ✅
}

// Never (pour les erreurs)
function error(message: string): never {
  throw new Error(message);
}

// Void (pas de retour)
function log(message: string): void {
  console.log(message);
}
```

### Interfaces

```typescript
// Interface de base
interface User {
  id: string;
  name: string;
  age: number;
}

// Propriété optionnelle
interface Product {
  id: string;
  name: string;
  description?: string;  // Optionnelle
}

// Propriété en lecture seule
interface Config {
  readonly apiUrl: string;
}

// Index signature
interface Dictionary {
  [key: string]: number;
}

// Extension d'interface
interface Employee extends User {
  department: string;
  salary: number;
}

// Interface pour fonction
interface Calculate {
  (a: number, b: number): number;
}

const add: Calculate = (a, b) => a + b;
```

### Types

```typescript
// Type alias
type ID = string | number;
type Status = "pending" | "success" | "error";

// Union type
let value: string | number;
value = "hello";  // ✅
value = 42;       // ✅

// Intersection type
type HasName = { name: string };
type HasAge = { age: number };
type Person = HasName & HasAge;

const person: Person = {
  name: "Marie",
  age: 25
};

// Type littéral
type Direction = "north" | "south" | "east" | "west";
let direction: Direction = "north";  // ✅
direction = "up";  // ❌ Erreur
```

### Génériques

```typescript
// Fonction générique
function identity<T>(value: T): T {
  return value;
}

const num = identity<number>(42);
const str = identity<string>("hello");

// Interface générique
interface Box<T> {
  value: T;
}

const numberBox: Box<number> = { value: 42 };
const stringBox: Box<string> = { value: "hello" };

// Contrainte générique
function getProperty<T, K extends keyof T>(obj: T, key: K) {
  return obj[key];
}

const user = { name: "Marie", age: 25 };
const name = getProperty(user, "name");  // ✅
const invalid = getProperty(user, "email");  // ❌
```

### Types pour React

```typescript
// Props de composant
interface ButtonProps {
  text: string;
  onClick: () => void;
  variant?: "primary" | "secondary";
  disabled?: boolean;
}

// Props avec children
interface CardProps {
  title: string;
  children: React.ReactNode;
}

// Props avec événements
interface InputProps {
  value: string;
  onChange: (e: React.ChangeEvent<HTMLInputElement>) => void;
  onBlur?: (e: React.FocusEvent<HTMLInputElement>) => void;
}

// Types d'événements courants
type ClickEvent = React.MouseEvent<HTMLButtonElement>;
type ChangeEvent = React.ChangeEvent<HTMLInputElement>;
type SubmitEvent = React.FormEvent<HTMLFormElement>;
type KeyEvent = React.KeyboardEvent<HTMLInputElement>;

// useState avec type
const [user, setUser] = useState<User | null>(null);
const [items, setItems] = useState<Item[]>([]);

// useRef avec type
const inputRef = useRef<HTMLInputElement>(null);
const divRef = useRef<HTMLDivElement>(null);
```

### Utility Types

```typescript
interface User {
  id: string;
  name: string;
  email: string;
  age: number;
}

// Partial - Toutes les propriétés optionnelles
type PartialUser = Partial<User>;
// { id?: string; name?: string; email?: string; age?: number; }

// Required - Toutes les propriétés requises
type RequiredUser = Required<PartialUser>;

// Pick - Sélectionner certaines propriétés
type UserPreview = Pick<User, "id" | "name">;
// { id: string; name: string; }

// Omit - Exclure certaines propriétés
type UserWithoutEmail = Omit<User, "email">;
// { id: string; name: string; age: number; }

// Record - Créer un type objet
type Scores = Record<string, number>;
// { [key: string]: number }

// Readonly - Propriétés en lecture seule
type ReadonlyUser = Readonly<User>;
```

---

## 🎯 Patterns Courants pour React

### Typer les props

```typescript
// ❌ Éviter
function Button(props: any) {
  return <button>{props.text}</button>;
}

// ✅ Bon
interface ButtonProps {
  text: string;
  variant?: "primary" | "secondary";
}

function Button({ text, variant = "primary" }: ButtonProps) {
  return <button className={variant}>{text}</button>;
}
```

### Typer useState

```typescript
// Type inféré automatiquement
const [count, setCount] = useState(0);  // number

// Type explicite
const [user, setUser] = useState<User | null>(null);
const [items, setItems] = useState<Item[]>([]);

// Union type
type Status = "idle" | "loading" | "success" | "error";
const [status, setStatus] = useState<Status>("idle");
```

### Typer les événements

```typescript
// Click
const handleClick = (e: React.MouseEvent<HTMLButtonElement>) => {
  console.log(e.currentTarget.value);
};

// Change
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  console.log(e.target.value);
};

// Submit
const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
  e.preventDefault();
};

// Keyboard
const handleKeyPress = (e: React.KeyboardEvent<HTMLInputElement>) => {
  if (e.key === "Enter") {
    // ...
  }
};
```

### Typer useRef

```typescript
// Pour un élément DOM
const inputRef = useRef<HTMLInputElement>(null);

// Utilisation
if (inputRef.current) {
  inputRef.current.focus();
}

// Pour stocker une valeur mutable
const countRef = useRef<number>(0);
countRef.current = 42;
```

### Typer custom hooks

```typescript
interface UseFetchResult<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
}

function useFetch<T>(url: string): UseFetchResult<T> {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  
  // ...
  
  return { data, loading, error };
}

// Utilisation
const { data, loading } = useFetch<User[]>("/api/users");
```

---

## 🚀 Astuces & Best Practices

### Immutabilité

```javascript
// ❌ Mutation
const arr = [1, 2, 3];
arr.push(4);  // Mutation !

const obj = { name: "Marie" };
obj.age = 25;  // Mutation !

// ✅ Immutabilité
const newArr = [...arr, 4];
const newObj = { ...obj, age: 25 };

// Modifier un élément de tableau
const items = [1, 2, 3, 4, 5];
const updated = items.map(item => 
  item === 3 ? 10 : item
);  // [1, 2, 10, 4, 5]

// Supprimer un élément
const removed = items.filter(item => item !== 3);
// [1, 2, 4, 5]
```

### Fonctions pures

```javascript
// ❌ Impure (modifie l'input)
function addItem(arr, item) {
  arr.push(item);
  return arr;
}

// ✅ Pure (retourne un nouveau tableau)
function addItem(arr, item) {
  return [...arr, item];
}
```

### Early return

```javascript
// ❌ Nested if
function processUser(user) {
  if (user) {
    if (user.isActive) {
      if (user.hasPermission) {
        return user.data;
      }
    }
  }
  return null;
}

// ✅ Early return
function processUser(user) {
  if (!user) return null;
  if (!user.isActive) return null;
  if (!user.hasPermission) return null;
  return user.data;
}
```

---

**Cette cheat sheet couvre l'essentiel de JavaScript ES6+ et TypeScript pour React. Gardez-la à portée de main ! 📚**
