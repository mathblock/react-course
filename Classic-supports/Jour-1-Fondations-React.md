# Jour 1 : Fondations React - Composants et Props

## 📋 Objectifs de la journée

À la fin de cette journée, vous serez capable de :
- ✅ Comprendre la philosophie React et le Virtual DOM
- ✅ Créer des composants fonctionnels avec TypeScript
- ✅ Utiliser JSX pour construire des interfaces
- ✅ Passer et utiliser des props
- ✅ Rendre des listes avec `.map()`
- ✅ Construire une page menu statique pour un foodtruck

---

## 🌅 MATIN (3h30) : Premiers pas avec React

### 1. Introduction à React (45 min)

#### Qu'est-ce que React ?

React est une **bibliothèque JavaScript** pour construire des interfaces utilisateur. Créée par Facebook en 2013, c'est aujourd'hui la bibliothèque front-end la plus populaire.

**Pourquoi React ?**
- 🧩 **Composants réutilisables** : Construisez une fois, utilisez partout
- ⚡ **Rapide** : Virtual DOM pour des mises à jour optimisées
- 📱 **Déclaratif** : Décrivez ce que vous voulez voir, React s'occupe du reste
- 🎯 **Populaire** : Énorme écosystème et communauté

#### Philosophie React : Déclaratif vs Impératif

**Approche Impérative (Vanilla JS)** :
```javascript
// On dit COMMENT faire étape par étape
const button = document.createElement('button');
button.textContent = 'Cliquez-moi';
button.addEventListener('click', () => {
  button.textContent = 'Cliqué !';
});
document.body.appendChild(button);
```

**Approche Déclarative (React)** :
```typescript
// On dit CE QUE l'on veut afficher
function Button() {
  const [clicked, setClicked] = useState(false);
  return <button onClick={() => setClicked(true)}>
    {clicked ? 'Cliqué !' : 'Cliquez-moi'}
  </button>;
}
```

#### Le Virtual DOM

React maintient une copie virtuelle du DOM en mémoire. Quand vos données changent :
1. React met à jour le Virtual DOM
2. Compare avec la version précédente (diffing)
3. Ne met à jour que les éléments réellement modifiés dans le vrai DOM

**Résultat** : Performances optimales automatiquement ! 🚀

### 2. Configuration de l'environnement (15 min)

#### Installation avec Vite (recommandé 2025)

```bash
# Créer un nouveau projet React avec TypeScript
npm create vite@latest my-foodtruck -- --template react-ts

# Se déplacer dans le dossier
cd my-foodtruck

# Installer les dépendances
npm install

# Lancer le serveur de développement
npm run dev
```

**⚠️ Note importante** : Nous utilisons **Vite** et NON Create React App (CRA) qui est obsolète depuis 2024.

#### Structure du projet

```
my-foodtruck/
├── src/
│   ├── App.tsx          # Composant principal
│   ├── main.tsx         # Point d'entrée
│   ├── index.css        # Styles globaux
│   └── components/      # Nos composants (à créer)
├── package.json
└── tsconfig.json
```

### 3. JSX : Le langage de React (45 min)

#### Qu'est-ce que JSX ?

JSX (JavaScript XML) permet d'écrire du "HTML" dans JavaScript. C'est du sucre syntaxique qui sera transformé en JavaScript.

```typescript
// JSX
const element = <h1>Bonjour le monde !</h1>;

// Est transformé en :
const element = React.createElement('h1', null, 'Bonjour le monde !');
```

#### Les règles du JSX

**1. Un seul élément parent**
```typescript
// ❌ Incorrect
return (
  <h1>Titre</h1>
  <p>Paragraphe</p>
);

// ✅ Correct avec fragment
return (
  <>
    <h1>Titre</h1>
    <p>Paragraphe</p>
  </>
);

// ✅ Correct avec div
return (
  <div>
    <h1>Titre</h1>
    <p>Paragraphe</p>
  </div>
);
```

**2. Utiliser className au lieu de class**
```typescript
// ❌ Incorrect
<div class="container">

// ✅ Correct
<div className="container">
```

**3. Utiliser camelCase pour les attributs**
```typescript
// ❌ Incorrect
<button onclick={handleClick}>

// ✅ Correct
<button onClick={handleClick}>
```

**4. Fermer toutes les balises**
```typescript
// ❌ Incorrect
<img src="logo.png">
<input type="text">

// ✅ Correct
<img src="logo.png" />
<input type="text" />
```

**5. Expressions JavaScript entre accolades**
```typescript
const name = "Marie";
const age = 25;

return (
  <div>
    <h1>Bonjour {name} !</h1>
    <p>Tu as {age} ans</p>
    <p>Dans 5 ans tu auras {age + 5} ans</p>
  </div>
);
```

#### Exercice JSX (15 min)

Créez un composant qui affiche :
```typescript
function UserCard() {
  const user = {
    firstName: "Jean",
    lastName: "Dupont",
    age: 30,
    email: "jean.dupont@example.com"
  };

  return (
    // TODO: Afficher toutes les infos de l'utilisateur
    // dans un format de carte
  );
}
```

### 4. Les Composants Fonctionnels (45 min)

#### Qu'est-ce qu'un composant ?

Un composant est une **fonction JavaScript qui retourne du JSX**. C'est comme une brique LEGO : vous les assemblez pour construire votre application.

**Règles de nommage** :
- Toujours commencer par une **majuscule**
- Utiliser le PascalCase : `MonComposant`, pas `monComposant`

#### Votre premier composant

```typescript
// src/components/Welcome.tsx

function Welcome() {
  return (
    <div>
      <h1>Bienvenue chez Food Truck Paradise ! 🌮</h1>
      <p>Découvrez nos délicieux plats</p>
    </div>
  );
}

export default Welcome;
```

Pour l'utiliser dans App.tsx :
```typescript
import Welcome from './components/Welcome';

function App() {
  return (
    <div>
      <Welcome />
    </div>
  );
}
```

#### Les Props : Rendre les composants réutilisables

Les **props** (propriétés) permettent de passer des données d'un composant parent à un composant enfant.

**Analogie** : Les props sont comme les arguments d'une fonction !

```typescript
// src/components/Greeting.tsx

interface GreetingProps {
  name: string;
  age: number;
}

function Greeting({ name, age }: GreetingProps) {
  return (
    <div>
      <h1>Bonjour {name} !</h1>
      <p>Tu as {age} ans</p>
    </div>
  );
}

export default Greeting;
```

Utilisation :
```typescript
<Greeting name="Marie" age={25} />
<Greeting name="Pierre" age={30} />
```

#### Props : Règles importantes

**1. Les props sont en lecture seule**
```typescript
// ❌ INTERDIT - Ne jamais modifier les props
function Component({ value }: { value: number }) {
  value = value + 1; // ❌ Erreur !
  return <div>{value}</div>;
}

// ✅ CORRECT - Les props sont immuables
function Component({ value }: { value: number }) {
  const newValue = value + 1; // ✅ Créer une nouvelle variable
  return <div>{newValue}</div>;
}
```

**2. Props optionnelles avec `?`**
```typescript
interface CardProps {
  title: string;
  subtitle?: string; // Optionnelle
  imageUrl?: string; // Optionnelle
}

function Card({ title, subtitle = "Pas de sous-titre", imageUrl }: CardProps) {
  return (
    <div>
      <h2>{title}</h2>
      <p>{subtitle}</p>
      {imageUrl && <img src={imageUrl} alt={title} />}
    </div>
  );
}
```

**3. Destructuring des props**
```typescript
// ❌ Moins lisible
function Card(props: CardProps) {
  return <div>{props.title}</div>;
}

// ✅ Plus lisible avec destructuring
function Card({ title, subtitle }: CardProps) {
  return <div>{title}</div>;
}
```

#### Exercice : Composants et Props (30 min)

Créez ces 3 composants :

**1. Button**
```typescript
interface ButtonProps {
  text: string;
  variant: 'primary' | 'secondary' | 'danger';
}

// TODO: Implémenter le composant
// Affichez un bouton avec un style différent selon le variant
```

**2. ProfileCard**
```typescript
interface ProfileCardProps {
  name: string;
  role: string;
  avatarUrl: string;
  bio?: string;
}

// TODO: Implémenter une carte de profil stylisée
```

**3. Alert**
```typescript
interface AlertProps {
  type: 'info' | 'warning' | 'error' | 'success';
  message: string;
}

// TODO: Afficher une alerte avec couleur selon le type
```

---

## 🌆 APRÈS-MIDI (3h30) : Construction du menu foodtruck

### 1. Rappel JavaScript ES6 (30 min)

#### Destructuring

```typescript
// Objets
const user = { name: "Marie", age: 25 };
const { name, age } = user;

// Tableaux
const fruits = ["pomme", "banane", "orange"];
const [first, second] = fruits;
```

#### Spread Operator

```typescript
// Copie de tableau
const arr1 = [1, 2, 3];
const arr2 = [...arr1, 4, 5]; // [1, 2, 3, 4, 5]

// Copie d'objet
const user = { name: "Jean", age: 30 };
const updatedUser = { ...user, age: 31 };
```

#### Array.map()

```typescript
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(num => num * 2);
// [2, 4, 6, 8, 10]

const users = [
  { name: "Marie", age: 25 },
  { name: "Jean", age: 30 }
];
const names = users.map(user => user.name);
// ["Marie", "Jean"]
```

### 2. Rendre des listes avec .map() (45 min)

#### Le principe

Pour afficher une liste d'éléments, on utilise `.map()` pour transformer un tableau de données en tableau de JSX.

```typescript
function FruitList() {
  const fruits = ["Pomme", "Banane", "Orange"];
  
  return (
    <ul>
      {fruits.map((fruit, index) => (
        <li key={index}>{fruit}</li>
      ))}
    </ul>
  );
}
```

#### L'importance de la key prop

React a besoin d'une clé **unique et stable** pour identifier chaque élément de la liste.

```typescript
// ❌ Mauvais : utiliser l'index (sauf si liste statique)
{items.map((item, index) => (
  <div key={index}>{item}</div>
))}

// ✅ Bon : utiliser un ID unique
{items.map((item) => (
  <div key={item.id}>{item.name}</div>
))}
```

**Pourquoi c'est important ?**
Sans key stable, React ne peut pas identifier correctement les éléments lors d'une mise à jour, ce qui cause des bugs visuels.

#### Exemple complet

```typescript
interface Product {
  id: string;
  name: string;
  price: number;
}

function ProductList() {
  const products: Product[] = [
    { id: "1", name: "Burger", price: 8.99 },
    { id: "2", name: "Pizza", price: 12.99 },
    { id: "3", name: "Tacos", price: 6.99 }
  ];

  return (
    <div>
      <h2>Notre Menu</h2>
      <div className="product-grid">
        {products.map((product) => (
          <div key={product.id} className="product-card">
            <h3>{product.name}</h3>
            <p>{product.price}€</p>
          </div>
        ))}
      </div>
    </div>
  );
}
```

### 3. Rendu conditionnel (30 min)

Afficher ou masquer des éléments selon une condition.

#### Avec l'opérateur &&

```typescript
function UserGreeting({ isLoggedIn }: { isLoggedIn: boolean }) {
  return (
    <div>
      {isLoggedIn && <p>Bienvenue !</p>}
      {!isLoggedIn && <p>Veuillez vous connecter</p>}
    </div>
  );
}
```

#### Avec l'opérateur ternaire

```typescript
function Status({ isOnline }: { isOnline: boolean }) {
  return (
    <div>
      <span>{isOnline ? "🟢 En ligne" : "🔴 Hors ligne"}</span>
    </div>
  );
}
```

#### Avec une variable

```typescript
function ProductCard({ product }: { product: Product }) {
  let badge = null;
  
  if (product.isNew) {
    badge = <span className="badge">Nouveau</span>;
  }
  
  return (
    <div>
      <h3>{product.name}</h3>
      {badge}
    </div>
  );
}
```

---

## 🎯 ATELIER FIL ROUGE : Menu Foodtruck (2h)

### Objectif

Créer une page menu complète pour un foodtruck avec :
- Header avec logo et navigation
- Liste de plats organisés par catégorie
- Footer avec informations de contact

### Étape 1 : Structure des données (15 min)

Créez `src/data/menuData.ts` :

```typescript
export interface MenuItem {
  id: string;
  name: string;
  description: string;
  price: number;
  category: 'entrees' | 'plats' | 'desserts' | 'boissons';
  imageUrl: string;
  isVegetarian: boolean;
}

export const menuItems: MenuItem[] = [
  {
    id: "1",
    name: "Tacos Poulet",
    description: "Tortilla garnie de poulet mariné, légumes frais et sauce maison",
    price: 8.50,
    category: "plats",
    imageUrl: "https://images.unsplash.com/photo-1565299624946-b28f40a0ae38",
    isVegetarian: false
  },
  {
    id: "2",
    name: "Burger Végétarien",
    description: "Steak végétal, tomates, salade, oignons rouges",
    price: 9.00,
    category: "plats",
    imageUrl: "https://images.unsplash.com/photo-1520072959219-c595dc870360",
    isVegetarian: true
  },
  // Ajoutez au moins 10-15 items au total
];
```

### Étape 2 : Composant Header (20 min)

```typescript
// src/components/Header.tsx

function Header() {
  return (
    <header className="header">
      <div className="container">
        <div className="logo">
          <h1>🌮 Food Truck Paradise</h1>
        </div>
        <nav>
          <a href="#menu">Menu</a>
          <a href="#about">À propos</a>
          <a href="#contact">Contact</a>
        </nav>
      </div>
    </header>
  );
}

export default Header;
```

### Étape 3 : Composant MenuCard (30 min)

```typescript
// src/components/MenuCard.tsx

interface MenuCardProps {
  item: MenuItem;
}

function MenuCard({ item }: MenuCardProps) {
  return (
    <div className="menu-card">
      <img src={item.imageUrl} alt={item.name} />
      <div className="card-content">
        <div className="card-header">
          <h3>{item.name}</h3>
          {item.isVegetarian && <span className="badge">🌱 Végé</span>}
        </div>
        <p className="description">{item.description}</p>
        <div className="card-footer">
          <span className="price">{item.price.toFixed(2)}€</span>
          <button className="btn-primary">Ajouter</button>
        </div>
      </div>
    </div>
  );
}

export default MenuCard;
```

### Étape 4 : Composant Menu principal (30 min)

```typescript
// src/components/Menu.tsx

import { menuItems } from '../data/menuData';
import MenuCard from './MenuCard';

function Menu() {
  return (
    <section className="menu-section">
      <div className="container">
        <h2>Notre Menu</h2>
        
        <div className="category-section">
          <h3>Plats Principaux</h3>
          <div className="menu-grid">
            {menuItems
              .filter(item => item.category === 'plats')
              .map(item => (
                <MenuCard key={item.id} item={item} />
              ))}
          </div>
        </div>

        {/* Répéter pour les autres catégories */}
      </div>
    </section>
  );
}

export default Menu;
```

### Étape 5 : Composant Footer (15 min)

```typescript
// src/components/Footer.tsx

function Footer() {
  return (
    <footer className="footer">
      <div className="container">
        <div className="footer-section">
          <h4>Horaires</h4>
          <p>Lundi - Vendredi : 11h - 22h</p>
          <p>Samedi - Dimanche : 12h - 23h</p>
        </div>
        <div className="footer-section">
          <h4>Contact</h4>
          <p>📞 01 23 45 67 89</p>
          <p>📧 contact@foodtruck.fr</p>
        </div>
        <div className="footer-section">
          <h4>Adresse</h4>
          <p>Place du Marché</p>
          <p>75001 Paris</p>
        </div>
      </div>
    </footer>
  );
}

export default Footer;
```

### Étape 6 : Assemblage dans App.tsx (10 min)

```typescript
// src/App.tsx

import Header from './components/Header';
import Menu from './components/Menu';
import Footer from './components/Footer';
import './App.css';

function App() {
  return (
    <div className="app">
      <Header />
      <main>
        <Menu />
      </main>
      <Footer />
    </div>
  );
}

export default App;
```

### Styling CSS de base (20 min)

```css
/* src/App.css */

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 20px;
}

/* Header */
.header {
  background: #2c3e50;
  color: white;
  padding: 1rem 0;
}

.logo h1 {
  font-size: 1.5rem;
}

nav {
  display: flex;
  gap: 2rem;
  margin-top: 1rem;
}

nav a {
  color: white;
  text-decoration: none;
}

/* Menu Grid */
.menu-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 2rem;
  margin: 2rem 0;
}

/* Menu Card */
.menu-card {
  border: 1px solid #ddd;
  border-radius: 8px;
  overflow: hidden;
  transition: transform 0.2s;
}

.menu-card:hover {
  transform: translateY(-4px);
  box-shadow: 0 4px 12px rgba(0,0,0,0.1);
}

.menu-card img {
  width: 100%;
  height: 200px;
  object-fit: cover;
}

.card-content {
  padding: 1rem;
}

.badge {
  background: #27ae60;
  color: white;
  padding: 0.25rem 0.5rem;
  border-radius: 4px;
  font-size: 0.75rem;
}

.price {
  font-size: 1.25rem;
  font-weight: bold;
  color: #e74c3c;
}

/* Footer */
.footer {
  background: #34495e;
  color: white;
  padding: 2rem 0;
  margin-top: 4rem;
}

.footer .container {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 2rem;
}
```

---

## 📝 Points Clés à Retenir

### Concepts React
✅ **Composant** = Fonction qui retourne du JSX
✅ **Props** = Données passées du parent à l'enfant (lecture seule)
✅ **JSX** = Syntaxe pour écrire du HTML dans JavaScript
✅ **Key prop** = Identifiant unique pour les éléments de liste

### Règles d'or
⚠️ Toujours commencer les composants par une **MAJUSCULE**
⚠️ Les props sont **immuables** (jamais les modifier)
⚠️ Utiliser `.map()` pour rendre des listes
⚠️ Toujours donner une **key unique** aux éléments de liste

### TypeScript
🔷 Toujours typer les props avec une **interface**
🔷 Utiliser `?` pour les props optionnelles
🔷 Destructurer les props dans les paramètres

---

## 🏠 Exercice à la maison

Améliorez votre menu foodtruck :

1. **Ajoutez plus de plats** (minimum 20 items)
2. **Créez un composant Hero** avec une grande image et un slogan
3. **Ajoutez des icônes** pour les allergènes (gluten, lactose, etc.)
4. **Rendez le design responsive** avec CSS Grid/Flexbox
5. **Ajoutez un badge "NOUVEAU"** pour les plats récents

---

## 📚 Ressources supplémentaires

- [Documentation React Officielle](https://react.dev)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [MDN - Array.map()](https://developer.mozilla.org/fr/docs/Web/JavaScript/Reference/Global_Objects/Array/map)

---

## ❓ Questions fréquentes

**Q : Quelle est la différence entre un composant et un élément ?**
R : Un composant est une fonction, un élément est ce que cette fonction retourne (du JSX).

**Q : Peut-on passer des fonctions en props ?**
R : Oui ! On le fera demain avec les événements.

**Q : Pourquoi TypeScript ?**
R : Pour avoir l'autocomplétion et détecter les erreurs avant l'exécution.

**Q : Pourquoi Vite et pas Create React App ?**
R : CRA est obsolète depuis 2024. Vite est plus rapide et mieux maintenu.

---

**Bravo ! Vous avez terminé le Jour 1 ! 🎉**

Demain, nous ajouterons de l'interactivité avec l'état local et les événements.
