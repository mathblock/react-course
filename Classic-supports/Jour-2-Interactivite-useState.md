# Jour 2 : Interactivité et État Local - useState

## 📋 Objectifs de la journée

À la fin de cette journée, vous serez capable de :
- ✅ Comprendre ce qu'est l'état (state) et pourquoi il existe
- ✅ Utiliser le hook useState pour gérer l'état local
- ✅ Gérer les événements utilisateur (click, change, submit)
- ✅ Créer des formulaires contrôlés avec React
- ✅ Implémenter un panier fonctionnel avec filtrage et recherche

---

## 🌅 MATIN (3h30) : useState en profondeur

### 1. Comprendre l'État (State) (45 min)

#### Qu'est-ce que l'État ?

L'**état** (state) est la **mémoire d'un composant**. C'est une donnée qui peut changer au fil du temps et qui déclenche un nouveau rendu du composant quand elle change.

**Analogie** : 
- **Props** = Courrier que vous recevez (lecture seule)
- **State** = Votre carnet personnel (vous pouvez l'éditer)

#### Pourquoi avons-nous besoin d'état ?

Sans état, React ne sait pas quand re-rendre un composant. Les variables JavaScript normales ne déclenchent pas de re-rendu.

```typescript
// ❌ Ceci ne fonctionne PAS
function Counter() {
  let count = 0;
  
  const increment = () => {
    count = count + 1;
    console.log(count); // S'affiche dans la console
    // Mais l'interface ne se met PAS à jour !
  };
  
  return (
    <div>
      <p>Compteur : {count}</p>
      <button onClick={increment}>+1</button>
    </div>
  );
}
```

```typescript
// ✅ Avec useState, ça fonctionne !
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  const increment = () => {
    setCount(count + 1);
    // React re-rend le composant avec la nouvelle valeur
  };
  
  return (
    <div>
      <p>Compteur : {count}</p>
      <button onClick={increment}>+1</button>
    </div>
  );
}
```

#### Syntaxe de useState

```typescript
const [valeur, setValeur] = useState(valeurInitiale);
```

- `valeur` : La valeur actuelle de l'état
- `setValeur` : Fonction pour mettre à jour l'état
- `valeurInitiale` : Valeur au premier rendu

**Exemples** :
```typescript
const [count, setCount] = useState(0);              // Nombre
const [name, setName] = useState("Marie");          // String
const [isOpen, setIsOpen] = useState(false);        // Boolean
const [items, setItems] = useState([]);             // Tableau
const [user, setUser] = useState({ name: "Jean" }); // Objet
```

#### Les 3 règles critiques de l'état

**Règle 1 : Ne JAMAIS muter l'état directement**

```typescript
// ❌ INTERDIT - Mutation directe
const [items, setItems] = useState([1, 2, 3]);
items.push(4);        // ❌ Ne déclenche PAS de re-rendu
setItems(items);      // ❌ React ne voit pas le changement

// ✅ CORRECT - Créer un nouveau tableau
setItems([...items, 4]);  // ✅ Nouveau tableau, React détecte le changement
```

**Règle 2 : setState est asynchrone**

```typescript
const [count, setCount] = useState(0);

const handleClick = () => {
  setCount(count + 1);
  console.log(count);  // ⚠️ Affiche encore 0 (ancienne valeur)
  // Le nouvel état sera disponible au prochain rendu
};
```

**Règle 3 : L'état est une constante jusqu'au prochain rendu**

```typescript
const [count, setCount] = useState(0);

const handleClick = () => {
  setCount(count + 1);  // count = 0 ici
  setCount(count + 1);  // count = 0 ici aussi !
  // Résultat : count devient 1, pas 2 !
};

// ✅ Solution : Fonction de mise à jour
const handleClick = () => {
  setCount(prev => prev + 1);  // prev est la vraie valeur courante
  setCount(prev => prev + 1);  // Maintenant count devient 2 !
};
```

### 2. useState avec différents types (30 min)

#### État de type nombre

```typescript
function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Compteur : {count}</p>
      <button onClick={() => setCount(count + 1)}>+1</button>
      <button onClick={() => setCount(count - 1)}>-1</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}
```

#### État de type boolean (toggle)

```typescript
function Toggle() {
  const [isOn, setIsOn] = useState(false);
  
  return (
    <div>
      <p>Statut : {isOn ? "Allumé ✅" : "Éteint ❌"}</p>
      <button onClick={() => setIsOn(!isOn)}>
        Toggle
      </button>
      <button onClick={() => setIsOn(true)}>Allumer</button>
      <button onClick={() => setIsOn(false)}>Éteindre</button>
    </div>
  );
}
```

#### État de type string (input)

```typescript
function NameInput() {
  const [name, setName] = useState("");
  
  return (
    <div>
      <input 
        type="text"
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Votre nom"
      />
      <p>Bonjour {name || "inconnu"} !</p>
    </div>
  );
}
```

#### État de type tableau

```typescript
function TodoList() {
  const [todos, setTodos] = useState<string[]>([]);
  const [input, setInput] = useState("");
  
  const addTodo = () => {
    if (input.trim()) {
      setTodos([...todos, input]);  // ✅ Spread operator
      setInput("");
    }
  };
  
  const removeTodo = (index: number) => {
    setTodos(todos.filter((_, i) => i !== index));  // ✅ Filter
  };
  
  return (
    <div>
      <input value={input} onChange={(e) => setInput(e.target.value)} />
      <button onClick={addTodo}>Ajouter</button>
      <ul>
        {todos.map((todo, index) => (
          <li key={index}>
            {todo}
            <button onClick={() => removeTodo(index)}>❌</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

#### État de type objet

```typescript
interface User {
  name: string;
  age: number;
  email: string;
}

function UserForm() {
  const [user, setUser] = useState<User>({
    name: "",
    age: 0,
    email: ""
  });
  
  const updateName = (name: string) => {
    setUser({ ...user, name });  // ✅ Spread + override
  };
  
  const updateAge = (age: number) => {
    setUser({ ...user, age });
  };
  
  return (
    <div>
      <input 
        value={user.name}
        onChange={(e) => updateName(e.target.value)}
      />
      <input 
        type="number"
        value={user.age}
        onChange={(e) => updateAge(Number(e.target.value))}
      />
    </div>
  );
}
```

### 3. Gestion des événements (45 min)

#### Événements de base

```typescript
function EventExamples() {
  // Click
  const handleClick = () => {
    console.log("Cliqué !");
  };
  
  // Double click
  const handleDoubleClick = () => {
    console.log("Double cliqué !");
  };
  
  // Mouse enter/leave
  const handleMouseEnter = () => {
    console.log("Souris entrée");
  };
  
  return (
    <div>
      <button onClick={handleClick}>Cliquer</button>
      <button onDoubleClick={handleDoubleClick}>Double cliquer</button>
      <div onMouseEnter={handleMouseEnter}>Survolez-moi</div>
    </div>
  );
}
```

#### Événements de formulaire

```typescript
function FormEvents() {
  const [value, setValue] = useState("");
  
  // Change (input, textarea, select)
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    setValue(e.target.value);
  };
  
  // Submit
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();  // ⚠️ Important : empêche le rechargement
    console.log("Formulaire soumis avec :", value);
  };
  
  // Focus / Blur
  const handleFocus = () => console.log("Focus");
  const handleBlur = () => console.log("Blur");
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        value={value}
        onChange={handleChange}
        onFocus={handleFocus}
        onBlur={handleBlur}
      />
      <button type="submit">Envoyer</button>
    </form>
  );
}
```

#### Passer des arguments aux handlers

```typescript
function ItemList() {
  const handleDelete = (id: string) => {
    console.log("Supprimer l'item", id);
  };
  
  const items = ["A", "B", "C"];
  
  return (
    <div>
      {items.map(item => (
        <button key={item} onClick={() => handleDelete(item)}>
          Supprimer {item}
        </button>
      ))}
    </div>
  );
}
```

#### Erreurs courantes avec les événements

```typescript
// ❌ Appel de fonction immédiat
<button onClick={handleClick()}>Cliquer</button>
// S'exécute au rendu, pas au clic !

// ✅ Référence à la fonction
<button onClick={handleClick}>Cliquer</button>

// ✅ Ou arrow function
<button onClick={() => handleClick()}>Cliquer</button>
```

### 4. Formulaires contrôlés (30 min)

#### Qu'est-ce qu'un composant contrôlé ?

Un **input contrôlé** est un input dont la valeur est contrôlée par React (via useState).

```typescript
function ControlledInput() {
  const [value, setValue] = useState("");
  
  return (
    <input
      value={value}  // ← React contrôle la valeur
      onChange={(e) => setValue(e.target.value)}
    />
  );
}
```

#### Formulaire complet

```typescript
interface FormData {
  name: string;
  email: string;
  age: number;
  newsletter: boolean;
}

function SignupForm() {
  const [formData, setFormData] = useState<FormData>({
    name: "",
    email: "",
    age: 0,
    newsletter: false
  });
  
  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const { name, value, type, checked } = e.target;
    
    setFormData({
      ...formData,
      [name]: type === 'checkbox' ? checked : value
    });
  };
  
  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    console.log("Données du formulaire :", formData);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        name="name"
        value={formData.name}
        onChange={handleChange}
        placeholder="Nom"
      />
      
      <input
        name="email"
        type="email"
        value={formData.email}
        onChange={handleChange}
        placeholder="Email"
      />
      
      <input
        name="age"
        type="number"
        value={formData.age}
        onChange={handleChange}
      />
      
      <label>
        <input
          name="newsletter"
          type="checkbox"
          checked={formData.newsletter}
          onChange={handleChange}
        />
        S'abonner à la newsletter
      </label>
      
      <button type="submit">S'inscrire</button>
    </form>
  );
}
```

### 5. Mini-challenges (1h30)

#### Challenge 1 : Calculatrice (30 min)

```typescript
// TODO: Créer une calculatrice simple
// - 2 inputs pour les nombres
// - Boutons +, -, *, /
// - Afficher le résultat
```

#### Challenge 2 : Color Picker (30 min)

```typescript
// TODO: Créer un color picker
// - 3 sliders (R, G, B) de 0 à 255
// - Afficher la couleur résultante en background
// - Afficher le code couleur en hexadécimal
```

#### Challenge 3 : Formulaire de newsletter (30 min)

```typescript
// TODO: Créer un formulaire de newsletter
// - Input email avec validation (doit contenir @)
// - Checkbox pour accepter les CGU
// - Bouton désactivé si email invalide ou CGU non acceptées
// - Message de confirmation après soumission
```

---

## 🌆 APRÈS-MIDI (3h30) : Panier et filtrage

### 1. Rappel : Lifting State Up (30 min)

#### Le problème

Quand deux composants ont besoin d'accéder au même état, on doit **remonter l'état** dans leur parent commun.

```typescript
// ❌ État séparé - les composants ne sont pas synchronisés
function ComponentA() {
  const [count, setCount] = useState(0);
  return <div>{count}</div>;
}

function ComponentB() {
  const [count, setCount] = useState(0);  // État différent !
  return <div>{count}</div>;
}
```

```typescript
// ✅ État partagé dans le parent
function Parent() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <ComponentA count={count} setCount={setCount} />
      <ComponentB count={count} setCount={setCount} />
    </div>
  );
}

function ComponentA({ count, setCount }: Props) {
  return <div onClick={() => setCount(count + 1)}>{count}</div>;
}

function ComponentB({ count, setCount }: Props) {
  return <div onClick={() => setCount(count + 1)}>{count}</div>;
}
```

#### Principe : "Où placer l'état ?"

**Question à se poser** : Quel composant a besoin d'accéder à cet état ?

1. Si **un seul composant** → État local dans ce composant
2. Si **plusieurs enfants** → État dans le parent commun
3. Si **toute l'application** → On verra Context API demain !

### 2. État dérivé (Derived State) (20 min)

**Derived state** = Valeur calculée à partir de l'état existant. Ne PAS créer un état pour ça !

```typescript
// ❌ État dupliqué inutile
function ProductList({ products }: { products: Product[] }) {
  const [products, setProducts] = useState(initialProducts);
  const [productCount, setProductCount] = useState(products.length);  // ❌
  
  const addProduct = (product: Product) => {
    setProducts([...products, product]);
    setProductCount(productCount + 1);  // ❌ Duplication !
  };
}

// ✅ Calcul à la volée
function ProductList({ products }: { products: Product[] }) {
  const [products, setProducts] = useState(initialProducts);
  const productCount = products.length;  // ✅ Calculé automatiquement
  
  const addProduct = (product: Product) => {
    setProducts([...products, product]);
    // productCount se met à jour automatiquement
  };
}
```

### 3. Atelier Fil Rouge : Panier Fonctionnel (2h30)

#### Étape 1 : Types et État du panier (20 min)

```typescript
// src/types/cart.ts

import { MenuItem } from './menu';

export interface CartItem {
  item: MenuItem;
  quantity: number;
}

export interface Cart {
  items: CartItem[];
}
```

```typescript
// src/App.tsx

import { useState } from 'react';
import { CartItem } from './types/cart';

function App() {
  const [cart, setCart] = useState<CartItem[]>([]);
  
  // Fonction pour ajouter au panier
  const addToCart = (item: MenuItem) => {
    // Vérifier si l'item existe déjà
    const existingItem = cart.find(cartItem => cartItem.item.id === item.id);
    
    if (existingItem) {
      // Augmenter la quantité
      setCart(cart.map(cartItem =>
        cartItem.item.id === item.id
          ? { ...cartItem, quantity: cartItem.quantity + 1 }
          : cartItem
      ));
    } else {
      // Ajouter un nouvel item
      setCart([...cart, { item, quantity: 1 }]);
    }
  };
  
  // Fonction pour retirer du panier
  const removeFromCart = (itemId: string) => {
    setCart(cart.filter(cartItem => cartItem.item.id !== itemId));
  };
  
  // Fonction pour mettre à jour la quantité
  const updateQuantity = (itemId: string, quantity: number) => {
    if (quantity <= 0) {
      removeFromCart(itemId);
    } else {
      setCart(cart.map(cartItem =>
        cartItem.item.id === itemId
          ? { ...cartItem, quantity }
          : cartItem
      ));
    }
  };
  
  // Calcul du total
  const total = cart.reduce(
    (sum, cartItem) => sum + cartItem.item.price * cartItem.quantity,
    0
  );
  
  return (
    <div>
      <Header cartItemsCount={cart.length} />
      <Menu onAddToCart={addToCart} />
      {/* On ajoutera la vue panier plus tard */}
    </div>
  );
}
```

#### Étape 2 : Badge de compteur dans le Header (15 min)

```typescript
// src/components/Header.tsx

interface HeaderProps {
  cartItemsCount: number;
}

function Header({ cartItemsCount }: HeaderProps) {
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
          <button className="cart-button">
            🛒 Panier
            {cartItemsCount > 0 && (
              <span className="cart-badge">{cartItemsCount}</span>
            )}
          </button>
        </nav>
      </div>
    </header>
  );
}
```

CSS pour le badge :
```css
.cart-button {
  position: relative;
  background: #e74c3c;
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 4px;
  cursor: pointer;
}

.cart-badge {
  position: absolute;
  top: -8px;
  right: -8px;
  background: #c0392b;
  color: white;
  border-radius: 50%;
  width: 24px;
  height: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 0.75rem;
  font-weight: bold;
}
```

#### Étape 3 : Bouton "Ajouter au panier" (15 min)

```typescript
// src/components/MenuCard.tsx

interface MenuCardProps {
  item: MenuItem;
  onAddToCart: (item: MenuItem) => void;
}

function MenuCard({ item, onAddToCart }: MenuCardProps) {
  const [isAdding, setIsAdding] = useState(false);
  
  const handleAdd = () => {
    setIsAdding(true);
    onAddToCart(item);
    
    // Animation de feedback
    setTimeout(() => setIsAdding(false), 500);
  };
  
  return (
    <div className="menu-card">
      <img src={item.imageUrl} alt={item.name} />
      <div className="card-content">
        <h3>{item.name}</h3>
        <p>{item.description}</p>
        <div className="card-footer">
          <span className="price">{item.price.toFixed(2)}€</span>
          <button 
            className={`btn-add ${isAdding ? 'adding' : ''}`}
            onClick={handleAdd}
          >
            {isAdding ? '✅ Ajouté' : '➕ Ajouter'}
          </button>
        </div>
      </div>
    </div>
  );
}
```

#### Étape 4 : Filtrage par catégorie (30 min)

```typescript
// src/components/Menu.tsx

function Menu({ onAddToCart }: { onAddToCart: (item: MenuItem) => void }) {
  const [activeCategory, setActiveCategory] = useState<string>('tous');
  
  const categories = [
    { id: 'tous', label: 'Tous' },
    { id: 'entrees', label: 'Entrées' },
    { id: 'plats', label: 'Plats' },
    { id: 'desserts', label: 'Desserts' },
    { id: 'boissons', label: 'Boissons' }
  ];
  
  // Filtrer les items selon la catégorie active
  const filteredItems = activeCategory === 'tous'
    ? menuItems
    : menuItems.filter(item => item.category === activeCategory);
  
  return (
    <section className="menu-section">
      <div className="container">
        <h2>Notre Menu</h2>
        
        {/* Boutons de filtre */}
        <div className="category-filters">
          {categories.map(cat => (
            <button
              key={cat.id}
              className={`filter-btn ${activeCategory === cat.id ? 'active' : ''}`}
              onClick={() => setActiveCategory(cat.id)}
            >
              {cat.label}
            </button>
          ))}
        </div>
        
        {/* Grille de produits */}
        <div className="menu-grid">
          {filteredItems.map(item => (
            <MenuCard key={item.id} item={item} onAddToCart={onAddToCart} />
          ))}
        </div>
        
        {/* Message si aucun résultat */}
        {filteredItems.length === 0 && (
          <p className="no-results">Aucun produit dans cette catégorie</p>
        )}
      </div>
    </section>
  );
}
```

CSS pour les filtres :
```css
.category-filters {
  display: flex;
  gap: 1rem;
  margin: 2rem 0;
  flex-wrap: wrap;
}

.filter-btn {
  background: white;
  border: 2px solid #ddd;
  padding: 0.5rem 1.5rem;
  border-radius: 20px;
  cursor: pointer;
  transition: all 0.3s;
}

.filter-btn:hover {
  border-color: #3498db;
}

.filter-btn.active {
  background: #3498db;
  color: white;
  border-color: #3498db;
}
```

#### Étape 5 : Barre de recherche (30 min)

```typescript
function Menu({ onAddToCart }: { onAddToCart: (item: MenuItem) => void }) {
  const [activeCategory, setActiveCategory] = useState<string>('tous');
  const [searchTerm, setSearchTerm] = useState('');
  
  // Filtrer par catégorie ET par recherche
  const filteredItems = menuItems
    .filter(item => 
      activeCategory === 'tous' || item.category === activeCategory
    )
    .filter(item =>
      item.name.toLowerCase().includes(searchTerm.toLowerCase()) ||
      item.description.toLowerCase().includes(searchTerm.toLowerCase())
    );
  
  return (
    <section className="menu-section">
      <div className="container">
        <h2>Notre Menu</h2>
        
        {/* Barre de recherche */}
        <div className="search-bar">
          <input
            type="text"
            placeholder="Rechercher un plat..."
            value={searchTerm}
            onChange={(e) => setSearchTerm(e.target.value)}
          />
          {searchTerm && (
            <button 
              className="clear-search"
              onClick={() => setSearchTerm('')}
            >
              ❌
            </button>
          )}
        </div>
        
        {/* Filtres de catégorie */}
        <div className="category-filters">
          {/* ... code des boutons ... */}
        </div>
        
        {/* Affichage du nombre de résultats */}
        <p className="results-count">
          {filteredItems.length} produit{filteredItems.length > 1 ? 's' : ''} trouvé{filteredItems.length > 1 ? 's' : ''}
        </p>
        
        {/* Grille de produits */}
        <div className="menu-grid">
          {filteredItems.map(item => (
            <MenuCard key={item.id} item={item} onAddToCart={onAddToCart} />
          ))}
        </div>
      </div>
    </section>
  );
}
```

#### Étape 6 : Vue simple du panier (30 min)

```typescript
// src/components/CartSummary.tsx

interface CartSummaryProps {
  cart: CartItem[];
  onUpdateQuantity: (itemId: string, quantity: number) => void;
  onRemove: (itemId: string) => void;
}

function CartSummary({ cart, onUpdateQuantity, onRemove }: CartSummaryProps) {
  const total = cart.reduce(
    (sum, cartItem) => sum + cartItem.item.price * cartItem.quantity,
    0
  );
  
  if (cart.length === 0) {
    return (
      <div className="cart-empty">
        <p>Votre panier est vide</p>
        <p>🛒 Ajoutez des produits pour commencer</p>
      </div>
    );
  }
  
  return (
    <div className="cart-summary">
      <h3>Votre Panier ({cart.length})</h3>
      
      {cart.map(cartItem => (
        <div key={cartItem.item.id} className="cart-item">
          <img src={cartItem.item.imageUrl} alt={cartItem.item.name} />
          
          <div className="cart-item-info">
            <h4>{cartItem.item.name}</h4>
            <p className="price">{cartItem.item.price.toFixed(2)}€</p>
          </div>
          
          <div className="quantity-controls">
            <button 
              onClick={() => onUpdateQuantity(cartItem.item.id, cartItem.quantity - 1)}
            >
              -
            </button>
            <span>{cartItem.quantity}</span>
            <button 
              onClick={() => onUpdateQuantity(cartItem.item.id, cartItem.quantity + 1)}
            >
              +
            </button>
          </div>
          
          <p className="subtotal">
            {(cartItem.item.price * cartItem.quantity).toFixed(2)}€
          </p>
          
          <button 
            className="btn-remove"
            onClick={() => onRemove(cartItem.item.id)}
          >
            🗑️
          </button>
        </div>
      ))}
      
      <div className="cart-total">
        <h3>Total</h3>
        <h3>{total.toFixed(2)}€</h3>
      </div>
      
      <button className="btn-checkout">
        Commander
      </button>
    </div>
  );
}
```

---

## 📝 Points Clés à Retenir

### useState
✅ L'état déclenche un re-rendu quand il change
✅ Toujours utiliser `setState`, JAMAIS muter directement
✅ `setState` est asynchrone
✅ Utiliser `prev => ...` pour les mises à jour basées sur l'état précédent

### Immutabilité
✅ Tableaux : `[...arr, item]`, `arr.filter()`, `arr.map()`
✅ Objets : `{ ...obj, key: value }`
✅ Jamais `.push()`, `.splice()`, ou mutation directe

### Événements
✅ `onClick={handler}` pas `onClick={handler()}`
✅ `e.preventDefault()` pour les formulaires
✅ Types TypeScript : `React.ChangeEvent`, `React.FormEvent`, etc.

### Pattern
✅ **Lifting state up** : Remonter l'état dans le parent commun
✅ **Derived state** : Calculer plutôt que dupliquer
✅ **Controlled components** : React contrôle la valeur des inputs

---

## 🏠 Exercice à la maison

Améliorez votre application foodtruck :

1. **Ajoutez un filtre végétarien** avec un toggle
2. **Ajoutez un tri** (prix croissant/décroissant, alphabétique)
3. **Persistez le panier** dans localStorage
4. **Ajoutez des animations** lors de l'ajout au panier
5. **Créez un système de favoris**

---

## 📚 Ressources supplémentaires

- [React - useState Hook](https://react.dev/reference/react/useState)
- [React - Gérer les événements](https://react.dev/learn/responding-to-events)
- [Immutabilité en JavaScript](https://developer.mozilla.org/fr/docs/Glossary/Immutable)

---

**Bravo ! Vous avez terminé le Jour 2 ! 🎉**

Demain, nous ajouterons la navigation multi-pages avec React Router et la gestion d'état globale avec Context API.
