# Jour 4 : Production et Finitions - useEffect, API & Déploiement

## 📋 Objectifs de la journée

À la fin de cette journée, vous serez capable de :
- ✅ Comprendre et utiliser useEffect correctement
- ✅ Faire des appels API et gérer les états asynchrones
- ✅ Créer des custom hooks réutilisables
- ✅ Construire des formulaires avec validation complète
- ✅ Optimiser les performances avec React.memo, useMemo, useCallback
- ✅ Déployer votre application en production

---

## 🌅 MATIN (3h30) : useEffect et Données Asynchrones

### 1. Comprendre useEffect (45 min)

#### Qu'est-ce qu'un effet de bord (side effect) ?

Un **effet de bord** est une opération qui interagit avec le monde extérieur au composant :
- 🌐 Appels API
- 💾 localStorage / sessionStorage
- 📡 Abonnements (WebSocket, événements)
- 🖥️ Manipulation du DOM
- ⏰ Timers (setTimeout, setInterval)

#### La synchronisation, pas le lifecycle

**❌ Ancienne façon de penser** : "componentDidMount", "componentWillUnmount"
**✅ Nouvelle façon de penser** : "Synchroniser mon composant avec le système externe"

```typescript
import { useEffect } from 'react';

function Component() {
  useEffect(() => {
    // Code qui s'exécute après le rendu
    console.log('Effet exécuté');
    
    // Fonction de cleanup (optionnelle)
    return () => {
      console.log('Nettoyage');
    };
  }, [/* dépendances */]);
  
  return <div>Mon composant</div>;
}
```

#### Les 3 formes de useEffect

**1. Sans dépendances - S'exécute après CHAQUE rendu**
```typescript
useEffect(() => {
  console.log('Après chaque rendu');
});
```

**2. Tableau vide - S'exécute UNE SEULE FOIS au montage**
```typescript
useEffect(() => {
  console.log('Seulement au montage');
}, []);
```

**3. Avec dépendances - S'exécute quand les dépendances changent**
```typescript
useEffect(() => {
  console.log(`Count a changé : ${count}`);
}, [count]);
```

#### Le tableau de dépendances : Les règles d'or

**Règle 1 : Inclure TOUTES les valeurs utilisées**

```typescript
// ❌ Incorrect - userId manquant dans les dépendances
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, []);  // ❌ userId devrait être dans les dépendances !
}

// ✅ Correct
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);  // ✅
}
```

**Règle 2 : Ne jamais désactiver eslint-plugin-react-hooks**

Cette règle ESLint vous aide à éviter les bugs. Ne la désactivez JAMAIS.

```typescript
// ❌ DANGER !
useEffect(() => {
  fetchData(userId);
  // eslint-disable-next-line react-hooks/exhaustive-deps
}, []);  // ❌ Vous créez un bug !
```

**Règle 3 : Les objets et tableaux créent des problèmes**

```typescript
// ❌ Boucle infinie !
function Component() {
  const [data, setData] = useState(null);
  
  const options = { id: 123 };  // ⚠️ Nouvel objet à chaque rendu !
  
  useEffect(() => {
    fetchData(options).then(setData);
  }, [options]);  // ❌ options change à chaque fois = boucle infinie
}

// ✅ Solution 1 : Extraire les valeurs primitives
function Component() {
  const [data, setData] = useState(null);
  const optionId = 123;
  
  useEffect(() => {
    fetchData({ id: optionId }).then(setData);
  }, [optionId]);  // ✅ Nombre primitif
}

// ✅ Solution 2 : useMemo
function Component() {
  const [data, setData] = useState(null);
  
  const options = useMemo(() => ({ id: 123 }), []);
  
  useEffect(() => {
    fetchData(options).then(setData);
  }, [options]);  // ✅
}
```

#### Fonction de cleanup

Le cleanup s'exécute :
- Avant que l'effet ne s'exécute à nouveau
- Quand le composant se démonte

```typescript
// Exemple avec timer
function Timer() {
  const [seconds, setSeconds] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);
    
    // ⚠️ IMPORTANT : Nettoyer l'intervalle
    return () => {
      clearInterval(interval);
    };
  }, []);
  
  return <div>{seconds} secondes</div>;
}

// Exemple avec écouteur d'événements
function WindowSize() {
  const [size, setSize] = useState({ width: 0, height: 0 });
  
  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };
    
    window.addEventListener('resize', handleResize);
    handleResize();  // Initialiser
    
    // ⚠️ IMPORTANT : Retirer l'écouteur
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []);
  
  return <div>{size.width} x {size.height}</div>;
}
```

### 2. Appels API avec useEffect (1h)

#### Pattern standard pour fetching de données

```typescript
interface User {
  id: string;
  name: string;
  email: string;
}

function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState<User | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  
  useEffect(() => {
    // Fonction async à l'intérieur de useEffect
    const fetchUser = async () => {
      try {
        setLoading(true);
        setError(null);
        
        const response = await fetch(`/api/users/${userId}`);
        
        if (!response.ok) {
          throw new Error('Erreur lors du chargement');
        }
        
        const data = await response.json();
        setUser(data);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'Erreur inconnue');
      } finally {
        setLoading(false);
      }
    };
    
    fetchUser();
  }, [userId]);
  
  if (loading) return <div>Chargement...</div>;
  if (error) return <div>Erreur : {error}</div>;
  if (!user) return <div>Utilisateur non trouvé</div>;
  
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

#### ⚠️ Erreur commune : async directement dans useEffect

```typescript
// ❌ INTERDIT - useEffect ne peut pas être async
useEffect(async () => {
  const data = await fetchData();
  setData(data);
}, []);

// ✅ CORRECT - Créer une fonction async à l'intérieur
useEffect(() => {
  const loadData = async () => {
    const data = await fetchData();
    setData(data);
  };
  
  loadData();
}, []);

// ✅ CORRECT - IIFE (Immediately Invoked Function Expression)
useEffect(() => {
  (async () => {
    const data = await fetchData();
    setData(data);
  })();
}, []);
```

#### Annuler les requêtes avec AbortController

```typescript
function SearchResults({ query }: { query: string }) {
  const [results, setResults] = useState([]);
  
  useEffect(() => {
    const controller = new AbortController();
    
    const search = async () => {
      try {
        const response = await fetch(`/api/search?q=${query}`, {
          signal: controller.signal  // ← Passer le signal
        });
        const data = await response.json();
        setResults(data);
      } catch (err) {
        if (err.name === 'AbortError') {
          console.log('Requête annulée');
        }
      }
    };
    
    search();
    
    // Annuler la requête si le composant se démonte
    // ou si query change avant la fin de la requête
    return () => {
      controller.abort();
    };
  }, [query]);
  
  return <div>{/* Afficher les résultats */}</div>;
}
```

### 3. Custom Hooks (45 min)

#### Pourquoi créer des custom hooks ?

- ♻️ Réutiliser de la logique entre composants
- 🧹 Garder les composants propres et lisibles
- 🧪 Plus facile à tester

#### Exemple : useFetch

```typescript
// src/hooks/useFetch.ts

import { useState, useEffect } from 'react';

interface UseFetchResult<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
  refetch: () => void;
}

export function useFetch<T>(url: string): UseFetchResult<T> {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
  const [refetchIndex, setRefetchIndex] = useState(0);
  
  useEffect(() => {
    const controller = new AbortController();
    
    const fetchData = async () => {
      try {
        setLoading(true);
        setError(null);
        
        const response = await fetch(url, {
          signal: controller.signal
        });
        
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const json = await response.json();
        setData(json);
      } catch (err) {
        if (err.name !== 'AbortError') {
          setError(err instanceof Error ? err.message : 'Erreur inconnue');
        }
      } finally {
        setLoading(false);
      }
    };
    
    fetchData();
    
    return () => {
      controller.abort();
    };
  }, [url, refetchIndex]);
  
  const refetch = () => {
    setRefetchIndex(prev => prev + 1);
  };
  
  return { data, loading, error, refetch };
}
```

Utilisation :
```typescript
function MenuPage() {
  const { data: menuItems, loading, error } = useFetch<MenuItem[]>('/api/menu');
  
  if (loading) return <div>Chargement...</div>;
  if (error) return <div>Erreur : {error}</div>;
  
  return (
    <div>
      {menuItems?.map(item => (
        <MenuCard key={item.id} item={item} />
      ))}
    </div>
  );
}
```

#### Exemple : useLocalStorage

```typescript
// src/hooks/useLocalStorage.ts

import { useState, useEffect } from 'react';

export function useLocalStorage<T>(
  key: string,
  initialValue: T
): [T, (value: T) => void] {
  // Récupérer la valeur depuis localStorage
  const [storedValue, setStoredValue] = useState<T>(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });
  
  // Sauvegarder dans localStorage quand la valeur change
  useEffect(() => {
    try {
      window.localStorage.setItem(key, JSON.stringify(storedValue));
    } catch (error) {
      console.error(error);
    }
  }, [key, storedValue]);
  
  return [storedValue, setStoredValue];
}
```

Utilisation :
```typescript
function Settings() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  
  return (
    <div>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Thème : {theme}
      </button>
    </div>
  );
}
```

#### Exemple : useDebounce

```typescript
// src/hooks/useDebounce.ts

import { useState, useEffect } from 'react';

export function useDebounce<T>(value: T, delay: number = 500): T {
  const [debouncedValue, setDebouncedValue] = useState<T>(value);
  
  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);
  
  return debouncedValue;
}
```

Utilisation :
```typescript
function SearchBar() {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 500);
  
  useEffect(() => {
    if (debouncedSearchTerm) {
      // Faire la recherche seulement après 500ms d'inactivité
      searchAPI(debouncedSearchTerm);
    }
  }, [debouncedSearchTerm]);
  
  return (
    <input
      value={searchTerm}
      onChange={(e) => setSearchTerm(e.target.value)}
      placeholder="Rechercher..."
    />
  );
}
```

### 4. États de chargement et gestion d'erreurs (30 min)

#### Pattern : Loading, Error, Success

```typescript
type LoadingState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: string };

function DataComponent() {
  const [state, setState] = useState<LoadingState<MenuItem[]>>({ status: 'idle' });
  
  useEffect(() => {
    setState({ status: 'loading' });
    
    fetch('/api/menu')
      .then(res => res.json())
      .then(data => setState({ status: 'success', data }))
      .catch(err => setState({ status: 'error', error: err.message }));
  }, []);
  
  switch (state.status) {
    case 'idle':
    case 'loading':
      return <LoadingSpinner />;
    
    case 'error':
      return <ErrorMessage message={state.error} />;
    
    case 'success':
      return <MenuList items={state.data} />;
  }
}
```

#### Composants de feedback

```typescript
// src/components/LoadingSpinner.tsx

function LoadingSpinner() {
  return (
    <div className="loading-spinner">
      <div className="spinner"></div>
      <p>Chargement...</p>
    </div>
  );
}

// src/components/ErrorMessage.tsx

interface ErrorMessageProps {
  message: string;
  onRetry?: () => void;
}

function ErrorMessage({ message, onRetry }: ErrorMessageProps) {
  return (
    <div className="error-message">
      <p>❌ {message}</p>
      {onRetry && (
        <button onClick={onRetry}>Réessayer</button>
      )}
    </div>
  );
}

// src/components/EmptyState.tsx

function EmptyState({ message }: { message: string }) {
  return (
    <div className="empty-state">
      <p>📭 {message}</p>
    </div>
  );
}
```

CSS pour le spinner :
```css
.spinner {
  border: 4px solid rgba(0, 0, 0, 0.1);
  border-left-color: #3498db;
  border-radius: 50%;
  width: 40px;
  height: 40px;
  animation: spin 1s linear infinite;
}

@keyframes spin {
  to { transform: rotate(360deg); }
}
```

---

## 🌆 APRÈS-MIDI (3h30) : Formulaires et Optimisations

### 1. Formulaire de checkout avec validation (1h30)

#### Créer le formulaire

```typescript
// src/pages/CheckoutPage.tsx

import { useState } from 'react';
import { useNavigate } from 'react-router-dom';
import { useCart } from '../context/CartContext';

interface CheckoutForm {
  firstName: string;
  lastName: string;
  email: string;
  phone: string;
  deliveryType: 'delivery' | 'pickup';
  address: string;
  city: string;
  postalCode: string;
  notes: string;
}

interface FormErrors {
  [key: string]: string;
}

function CheckoutPage() {
  const { cart, total, clearCart } = useCart();
  const navigate = useNavigate();
  
  const [formData, setFormData] = useState<CheckoutForm>({
    firstName: '',
    lastName: '',
    email: '',
    phone: '',
    deliveryType: 'delivery',
    address: '',
    city: '',
    postalCode: '',
    notes: ''
  });
  
  const [errors, setErrors] = useState<FormErrors>({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  
  // Validation
  const validateForm = (): boolean => {
    const newErrors: FormErrors = {};
    
    // Prénom requis
    if (!formData.firstName.trim()) {
      newErrors.firstName = 'Le prénom est requis';
    }
    
    // Nom requis
    if (!formData.lastName.trim()) {
      newErrors.lastName = 'Le nom est requis';
    }
    
    // Email valide
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!formData.email.trim()) {
      newErrors.email = 'L\'email est requis';
    } else if (!emailRegex.test(formData.email)) {
      newErrors.email = 'Email invalide';
    }
    
    // Téléphone valide (format français)
    const phoneRegex = /^0[1-9](?:\d{8})$/;
    if (!formData.phone.trim()) {
      newErrors.phone = 'Le téléphone est requis';
    } else if (!phoneRegex.test(formData.phone.replace(/\s/g, ''))) {
      newErrors.phone = 'Téléphone invalide (ex: 0612345678)';
    }
    
    // Adresse si livraison
    if (formData.deliveryType === 'delivery') {
      if (!formData.address.trim()) {
        newErrors.address = 'L\'adresse est requise';
      }
      if (!formData.city.trim()) {
        newErrors.city = 'La ville est requise';
      }
      if (!formData.postalCode.trim()) {
        newErrors.postalCode = 'Le code postal est requis';
      } else if (!/^\d{5}$/.test(formData.postalCode)) {
        newErrors.postalCode = 'Code postal invalide';
      }
    }
    
    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };
  
  // Gérer les changements
  const handleChange = (
    e: React.ChangeEvent<HTMLInputElement | HTMLTextAreaElement | HTMLSelectElement>
  ) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
    
    // Effacer l'erreur du champ
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: '' }));
    }
  };
  
  // Soumettre
  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    
    if (!validateForm()) {
      return;
    }
    
    setIsSubmitting(true);
    
    try {
      // Simuler l'envoi au serveur
      await new Promise(resolve => setTimeout(resolve, 1500));
      
      // Générer un numéro de commande
      const orderNumber = `CMD${Date.now()}`;
      
      // Vider le panier
      clearCart();
      
      // Rediriger vers la confirmation
      navigate('/order-confirmation', {
        state: {
          orderNumber,
          email: formData.email
        }
      });
    } catch (err) {
      alert('Erreur lors de la commande');
    } finally {
      setIsSubmitting(false);
    }
  };
  
  if (cart.length === 0) {
    return (
      <div className="checkout-page">
        <h1>Votre panier est vide</h1>
        <button onClick={() => navigate('/menu')}>
          Voir le menu
        </button>
      </div>
    );
  }
  
  return (
    <div className="checkout-page">
      <h1>Finaliser la commande</h1>
      
      <div className="checkout-container">
        {/* Résumé de la commande */}
        <aside className="order-summary">
          <h2>Récapitulatif</h2>
          
          {cart.map(item => (
            <div key={item.item.id} className="summary-item">
              <span>{item.quantity}x {item.item.name}</span>
              <span>{(item.item.price * item.quantity).toFixed(2)}€</span>
            </div>
          ))}
          
          <div className="summary-total">
            <strong>Total</strong>
            <strong>{total.toFixed(2)}€</strong>
          </div>
        </aside>
        
        {/* Formulaire */}
        <form onSubmit={handleSubmit} className="checkout-form">
          <section>
            <h3>Informations personnelles</h3>
            
            <div className="form-row">
              <div className="form-group">
                <label htmlFor="firstName">Prénom *</label>
                <input
                  type="text"
                  id="firstName"
                  name="firstName"
                  value={formData.firstName}
                  onChange={handleChange}
                  className={errors.firstName ? 'error' : ''}
                />
                {errors.firstName && (
                  <span className="error-message">{errors.firstName}</span>
                )}
              </div>
              
              <div className="form-group">
                <label htmlFor="lastName">Nom *</label>
                <input
                  type="text"
                  id="lastName"
                  name="lastName"
                  value={formData.lastName}
                  onChange={handleChange}
                  className={errors.lastName ? 'error' : ''}
                />
                {errors.lastName && (
                  <span className="error-message">{errors.lastName}</span>
                )}
              </div>
            </div>
            
            <div className="form-group">
              <label htmlFor="email">Email *</label>
              <input
                type="email"
                id="email"
                name="email"
                value={formData.email}
                onChange={handleChange}
                className={errors.email ? 'error' : ''}
              />
              {errors.email && (
                <span className="error-message">{errors.email}</span>
              )}
            </div>
            
            <div className="form-group">
              <label htmlFor="phone">Téléphone *</label>
              <input
                type="tel"
                id="phone"
                name="phone"
                value={formData.phone}
                onChange={handleChange}
                placeholder="0612345678"
                className={errors.phone ? 'error' : ''}
              />
              {errors.phone && (
                <span className="error-message">{errors.phone}</span>
              )}
            </div>
          </section>
          
          <section>
            <h3>Livraison</h3>
            
            <div className="form-group">
              <label>Type de commande *</label>
              <div className="radio-group">
                <label>
                  <input
                    type="radio"
                    name="deliveryType"
                    value="delivery"
                    checked={formData.deliveryType === 'delivery'}
                    onChange={handleChange}
                  />
                  Livraison
                </label>
                <label>
                  <input
                    type="radio"
                    name="deliveryType"
                    value="pickup"
                    checked={formData.deliveryType === 'pickup'}
                    onChange={handleChange}
                  />
                  À emporter
                </label>
              </div>
            </div>
            
            {formData.deliveryType === 'delivery' && (
              <>
                <div className="form-group">
                  <label htmlFor="address">Adresse *</label>
                  <input
                    type="text"
                    id="address"
                    name="address"
                    value={formData.address}
                    onChange={handleChange}
                    className={errors.address ? 'error' : ''}
                  />
                  {errors.address && (
                    <span className="error-message">{errors.address}</span>
                  )}
                </div>
                
                <div className="form-row">
                  <div className="form-group">
                    <label htmlFor="city">Ville *</label>
                    <input
                      type="text"
                      id="city"
                      name="city"
                      value={formData.city}
                      onChange={handleChange}
                      className={errors.city ? 'error' : ''}
                    />
                    {errors.city && (
                      <span className="error-message">{errors.city}</span>
                    )}
                  </div>
                  
                  <div className="form-group">
                    <label htmlFor="postalCode">Code postal *</label>
                    <input
                      type="text"
                      id="postalCode"
                      name="postalCode"
                      value={formData.postalCode}
                      onChange={handleChange}
                      className={errors.postalCode ? 'error' : ''}
                    />
                    {errors.postalCode && (
                      <span className="error-message">{errors.postalCode}</span>
                    )}
                  </div>
                </div>
              </>
            )}
            
            <div className="form-group">
              <label htmlFor="notes">Notes (optionnel)</label>
              <textarea
                id="notes"
                name="notes"
                value={formData.notes}
                onChange={handleChange}
                rows={3}
                placeholder="Allergies, instructions spéciales..."
              />
            </div>
          </section>
          
          <button
            type="submit"
            className="btn-submit"
            disabled={isSubmitting}
          >
            {isSubmitting ? 'Commande en cours...' : 'Commander'}
          </button>
        </form>
      </div>
    </div>
  );
}
```

### 2. Page de confirmation (30 min)

```typescript
// src/pages/OrderConfirmationPage.tsx

import { useLocation, Link } from 'react-router-dom';

function OrderConfirmationPage() {
  const location = useLocation();
  const { orderNumber, email } = location.state || {};
  
  if (!orderNumber) {
    return (
      <div>
        <h1>Commande introuvable</h1>
        <Link to="/menu">Retour au menu</Link>
      </div>
    );
  }
  
  return (
    <div className="confirmation-page">
      <div className="confirmation-card">
        <div className="success-icon">✅</div>
        
        <h1>Commande confirmée !</h1>
        
        <p>Votre commande a bien été enregistrée.</p>
        
        <div className="order-details">
          <p><strong>Numéro de commande :</strong> {orderNumber}</p>
          <p><strong>Email de confirmation :</strong> {email}</p>
        </div>
        
        <p className="info">
          Vous recevrez un email de confirmation à l'adresse indiquée.
        </p>
        
        <div className="actions">
          <Link to="/menu" className="btn-primary">
            Nouvelle commande
          </Link>
        </div>
      </div>
    </div>
  );
}
```

### 3. Optimisations de performance (1h)

#### React.memo - Éviter les re-renders inutiles

```typescript
// ❌ Sans React.memo - re-render à chaque fois que le parent re-render
function ExpensiveComponent({ data }) {
  console.log('Render');
  // Calculs coûteux...
  return <div>{data}</div>;
}

// ✅ Avec React.memo - re-render seulement si data change
const ExpensiveComponent = React.memo(({ data }) => {
  console.log('Render');
  return <div>{data}</div>;
});
```

#### useMemo - Mémoriser des calculs coûteux

```typescript
import { useMemo } from 'react';

function ProductList({ products, filterTerm }) {
  // ❌ Sans useMemo - Recalculé à chaque render
  const filteredProducts = products.filter(p =>
    p.name.includes(filterTerm)
  );
  
  // ✅ Avec useMemo - Recalculé seulement si products ou filterTerm change
  const filteredProducts = useMemo(() => {
    console.log('Calcul du filtre');
    return products.filter(p => p.name.includes(filterTerm));
  }, [products, filterTerm]);
  
  return <div>{/* ... */}</div>;
}
```

#### useCallback - Mémoriser des fonctions

```typescript
import { useCallback } from 'react';

function Parent() {
  const [count, setCount] = useState(0);
  
  // ❌ Sans useCallback - Nouvelle fonction à chaque render
  const handleClick = () => {
    console.log('Clicked');
  };
  
  // ✅ Avec useCallback - Même fonction référence
  const handleClick = useCallback(() => {
    console.log('Clicked');
  }, []);
  
  return <Child onClick={handleClick} />;
}

const Child = React.memo(({ onClick }) => {
  console.log('Child render');
  return <button onClick={onClick}>Click</button>;
});
```

#### ⚠️ Important : N'optimisez pas prématurément !

```typescript
// ❌ Optimisation inutile - Code plus complexe sans bénéfice
function SimpleComponent({ name }) {
  const greeting = useMemo(() => `Hello ${name}`, [name]);
  return <div>{greeting}</div>;
}

// ✅ Simple et lisible
function SimpleComponent({ name }) {
  return <div>Hello {name}</div>;
}
```

**Règle** : Optimisez seulement si :
1. Vous avez mesuré un problème de performance avec React DevTools Profiler
2. Le composant est rendu TRÈS souvent
3. Le calcul est réellement coûteux

### 4. Déploiement (30 min)

#### Build de production

```bash
# Créer le build optimisé
npm run build

# Le dossier dist/ contient les fichiers prêts pour la production
```

#### Déploiement sur Netlify

**Option 1 : Via l'interface web**

1. Créer un compte sur [netlify.com](https://netlify.com)
2. Drag & drop le dossier `dist/`
3. Votre site est en ligne ! 🎉

**Option 2 : Via Git (recommandé)**

1. Pusher votre code sur GitHub
2. Connecter le repo à Netlify
3. Configuration :
   - Build command : `npm run build`
   - Publish directory : `dist`
4. Déploiement automatique à chaque push !

#### Déploiement sur Vercel

```bash
# Installer Vercel CLI
npm install -g vercel

# Se connecter
vercel login

# Déployer
vercel

# Suivre les instructions...
```

#### Configuration pour React Router

**Problème** : Les routes ne fonctionnent pas après déploiement (404).

**Solution** : Créer un fichier de redirection.

Pour Netlify, créer `public/_redirects` :
```
/* /index.html 200
```

Pour Vercel, créer `vercel.json` :
```json
{
  "rewrites": [
    { "source": "/(.*)", "destination": "/" }
  ]
}
```

---

## 📝 Points Clés à Retenir

### useEffect
✅ Penser "synchronisation", pas "lifecycle"
✅ Toujours inclure les dépendances
✅ Cleanup pour les timers, événements, abonnements
✅ Fonction async DANS useEffect, pas directement

### Custom Hooks
✅ Réutiliser la logique
✅ Commencer par "use"
✅ Peuvent utiliser d'autres hooks
✅ Facilite les tests

### Performance
✅ React.memo pour les composants
✅ useMemo pour les calculs
✅ useCallback pour les fonctions
✅ **Ne pas optimiser prématurément !**

### Bonnes pratiques
✅ Gérer loading, error, success
✅ Valider les formulaires
✅ Feedback utilisateur constant
✅ Code propre et lisible

---

## 🎉 Félicitations !

Vous avez terminé cette formation React/TypeScript de 4 jours !

Vous êtes maintenant capable de :
- ✅ Créer des applications React complètes
- ✅ Gérer l'état local et global
- ✅ Créer des interfaces interactives
- ✅ Faire des appels API
- ✅ Router entre plusieurs pages
- ✅ Déployer en production

## 🚀 Pour aller plus loin

### Prochaines étapes recommandées

1. **Testing**
   - React Testing Library
   - Jest
   - Tests unitaires et d'intégration

2. **Next.js**
   - Server-Side Rendering
   - App Router
   - API Routes

3. **Bibliothèques utiles**
   - TanStack Query (ex React Query) pour data fetching
   - Zustand pour state management
   - React Hook Form pour les formulaires
   - Zod pour la validation

4. **Styling**
   - Styled Components
   - Emotion
   - Tailwind CSS avancé

5. **Performance**
   - Code splitting
   - Lazy loading
   - Optimisation des images

### Ressources

- [React Documentation](https://react.dev)
- [TypeScript Handbook](https://www.typescriptlang.org/docs/)
- [React Patterns](https://reactpatterns.com)
- [JavaScript Info](https://javascript.info)

---

**Bon code et bon courage dans vos projets ! 💪**
