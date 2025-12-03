# JOUR 4 : Production et Finitions - useEffect, API & Déploiement

## 📊 Vue d'ensemble

**Durée totale** : 7 heures (3h30 matin + 3h30 après-midi)  
**Niveau** : Avancé  
**Prérequis** : Jours 1-3 complétés (composants, useState, Router, Context)

---

## 🎯 Objectifs Pédagogiques

À la fin de cette journée, les étudiants seront capables de :

### Connaissances (Savoir)
- [ ] Comprendre le concept d'effet de bord (side effect)
- [ ] Expliquer le cycle de vie d'un effet
- [ ] Comprendre les dépendances de useEffect
- [ ] Connaître les patterns d'optimisation React
- [ ] Comprendre le build et le déploiement

### Compétences (Savoir-faire)
- [ ] Utiliser useEffect correctement avec dépendances
- [ ] Faire des appels API avec fetch
- [ ] Gérer les états de chargement et d'erreur
- [ ] Créer des custom hooks réutilisables
- [ ] Implémenter des formulaires avec validation
- [ ] Optimiser avec React.memo, useMemo, useCallback
- [ ] Builder et déployer une application React

### Attitudes (Savoir-être)
- [ ] Penser aux effets de bord
- [ ] Anticiper les états asynchrones
- [ ] Optimiser quand nécessaire (pas prématurément)
- [ ] Préparer une app pour la production

---

## ⏰ Planning Détaillé

### 🌅 MATIN (08h00 - 12h00)

#### 08h00 - 08h15 | Accueil & Révisions (15 min)
**Format** : Discussion + Quiz

**Quiz révision Jour 3** :

1. Comment créer une route dynamique ? → `/menu/:id`
2. Quel hook pour naviguer programmatiquement ? → `useNavigate`
3. Comment accéder aux paramètres d'URL ? → `useParams`
4. Quel est le problème du prop drilling ?
5. Comment créer un Context ?

**Annonce du jour** :
"Dernier jour ! Aujourd'hui on rend l'application production-ready :
1. **useEffect** pour la synchronisation
2. **Appels API** pour des données réelles
3. **Formulaire de commande** complet
4. **Optimisations** de performance
5. **Déploiement** en ligne ! 🚀"

---

#### 08h15 - 10h00 | useEffect : Synchronisation et Effets de Bord (1h45)
**Format** : Présentation approfondie + Live coding

##### 08h15 - 08h45 | Qu'est-ce qu'un effet de bord ? (30 min)

**Définition** :
Un **effet de bord** (side effect) est une interaction avec le monde extérieur au composant.

**Exemples d'effets de bord** :
- 🌐 Appels API (fetch, axios)
- 💾 localStorage / sessionStorage
- 📡 WebSocket, SSE
- 🖥️ Manipulation du DOM directe
- ⏰ Timers (setTimeout, setInterval)
- 📊 Logging, analytics
- 🔔 Notifications
- 👂 Event listeners (window, document)

**Non-effets de bord** :
- Calculs mathématiques
- Transformation de données
- Rendu de JSX
- Appels de fonctions pures

**⚠️ Ancienne mentalité (Class components)** :
"componentDidMount, componentWillUnmount, componentDidUpdate..."

**✅ Nouvelle mentalité (Hooks)** :
"Je veux **synchroniser** mon composant avec un système externe."

**Exemple concret** :

```typescript
// ❌ Sans useEffect - Ne fonctionne pas
function WindowSize() {
  const [size, setSize] = useState({ width: 0, height: 0 });
  
  // Ce code s'exécute à chaque rendu (trop!)
  window.addEventListener('resize', () => {
    setSize({ width: window.innerWidth, height: window.innerHeight });
  });
  
  return <div>{size.width} x {size.height}</div>;
}

// ✅ Avec useEffect - Fonctionne
function WindowSize() {
  const [size, setSize] = useState({ width: 0, height: 0 });
  
  useEffect(() => {
    const handleResize = () => {
      setSize({ width: window.innerWidth, height: window.innerHeight });
    };
    
    window.addEventListener('resize', handleResize);
    handleResize();  // Initialiser
    
    // Cleanup
    return () => {
      window.removeEventListener('resize', handleResize);
    };
  }, []);  // [] = seulement au montage
  
  return <div>{size.width} x {size.height}</div>;
}
```

##### 08h45 - 09h15 | Syntaxe et dépendances de useEffect (30 min)

**3 formes de useEffect** :

```typescript
// 1. Sans dépendances - S'exécute après CHAQUE rendu
useEffect(() => {
  console.log('Après chaque rendu');
});

// 2. Tableau vide - S'exécute UNE FOIS au montage
useEffect(() => {
  console.log('Une fois au montage');
}, []);

// 3. Avec dépendances - S'exécute quand les dépendances changent
useEffect(() => {
  console.log(`Count a changé : ${count}`);
}, [count]);
```

**Le tableau de dépendances : RÈGLES D'OR** :

**Règle 1 : Inclure TOUTES les valeurs utilisées**

```typescript
// ❌ FAUX - userId manque
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, []);  // ❌ userId devrait être ici !
  
  // Bug : Si userId change, on ne refetch pas
}

// ✅ CORRECT
function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]);  // ✅ userId inclus
}
```

**Règle 2 : Ne JAMAIS désactiver eslint-plugin-react-hooks**

```typescript
// ❌ DANGER ABSOLU
useEffect(() => {
  doSomething(value);
  // eslint-disable-next-line react-hooks/exhaustive-deps
}, []);  // ❌ Vous créez un bug !
```

**Règle 3 : Attention aux objets/tableaux**

```typescript
// ❌ Boucle infinie !
function Component() {
  const [data, setData] = useState(null);
  
  const options = { id: 123 };  // ⚠️ Nouvel objet à chaque rendu
  
  useEffect(() => {
    fetchData(options).then(setData);
  }, [options]);  // ❌ options change à chaque fois = boucle
}

// ✅ Solution 1 : Extraire les primitives
function Component() {
  const [data, setData] = useState(null);
  const optionId = 123;  // Nombre primitif
  
  useEffect(() => {
    fetchData({ id: optionId }).then(setData);
  }, [optionId]);  // ✅
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

**Fonction de cleanup** :

```typescript
useEffect(() => {
  // Setup
  const timer = setInterval(() => {
    console.log('Tick');
  }, 1000);
  
  // Cleanup (s'exécute avant le prochain effet et au démontage)
  return () => {
    clearInterval(timer);
  };
}, []);
```

**Quand cleanup ?**
- ⏰ Timers (setTimeout, setInterval)
- 👂 Event listeners (addEventListener)
- 📡 Abonnements (WebSocket, SSE)
- 🔄 Requêtes en cours (AbortController)

##### 09h15 - 10h00 | Exercices useEffect (45 min)

**🎯 Exercice 1 : Timer (15 min)**

```typescript
// Consigne : Créer un timer qui compte les secondes
// - Démarre à 0
// - S'incrémente toutes les secondes
// - Se nettoie au démontage

function Timer() {
  // TODO: useState pour les secondes
  
  // TODO: useEffect pour l'intervalle
  
  return <div>Temps écoulé : {/* secondes */} secondes</div>;
}
```

**Solution** :
```typescript
function Timer() {
  const [seconds, setSeconds] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);
    
    return () => {
      clearInterval(interval);
    };
  }, []);
  
  return <div>Temps écoulé : {seconds} secondes</div>;
}
```

**🎯 Exercice 2 : Document title (10 min)**

```typescript
// Consigne : Changer le titre de la page selon une prop
interface PageTitleProps {
  title: string;
}

function PageTitle({ title }: PageTitleProps) {
  // TODO: useEffect pour changer document.title
  
  return null;
}

// Utilisation : <PageTitle title="Mon Super Site" />
```

**Solution** :
```typescript
function PageTitle({ title }: PageTitleProps) {
  useEffect(() => {
    document.title = title;
  }, [title]);
  
  return null;
}
```

**🎯 Exercice 3 : Event listener (20 min)**

```typescript
// Consigne : Afficher les coordonnées de la souris
function MouseTracker() {
  // TODO: useState pour position
  
  // TODO: useEffect pour mousemove listener
  
  return (
    <div>
      Position : x={/* x */}, y={/* y */}
    </div>
  );
}
```

**Montrer les bugs courants** :
1. Oublier cleanup → Memory leak
2. Mauvaises dépendances → Stale values

---

#### ☕ 10h00 - 10h15 | PAUSE (15 min)

---

#### 10h15 - 12h00 | Appels API avec useEffect (1h45)
**Format** : Live coding + Custom hooks

##### 10h15 - 10h50 | Pattern de fetching de données (35 min)

**Pattern complet** :

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
    const fetchUser = async () => {
      try {
        setLoading(true);
        setError(null);
        
        const response = await fetch(`/api/users/${userId}`);
        
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        
        const data = await response.json();
        setUser(data);
      } catch (err) {
        setError(err instanceof Error ? err.message : 'Une erreur est survenue');
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

**⚠️ Erreur TRÈS courante : async directement**

```typescript
// ❌ FAUX - useEffect ne peut pas être async
useEffect(async () => {
  const data = await fetchData();
  setData(data);
}, []);

// ✅ CORRECT - Fonction async à l'intérieur
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

**Gérer l'annulation avec AbortController** :

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
          return;
        }
        console.error(err);
      }
    };
    
    search();
    
    // Cleanup : annuler la requête
    return () => {
      controller.abort();
    };
  }, [query]);
  
  return <div>{/* Afficher results */}</div>;
}
```

**Pourquoi c'est important ?**
Si l'utilisateur tape vite dans la recherche, on ne veut pas que les anciennes requêtes écrasent les nouvelles.

##### 10h50 - 11h30 | Custom Hook : useFetch (40 min)

**Créer `src/hooks/useFetch.ts`** :

```typescript
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
          setError(err instanceof Error ? err.message : 'Une erreur est survenue');
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

**Utilisation** :

```typescript
function MenuPage() {
  const { data: menuItems, loading, error, refetch } = useFetch<MenuItem[]>('/api/menu');
  
  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorMessage message={error} onRetry={refetch} />;
  
  return (
    <div>
      {menuItems?.map(item => (
        <MenuCard key={item.id} item={item} />
      ))}
    </div>
  );
}
```

**Montrer la réutilisabilité** :

```typescript
// Réutiliser pour différentes ressources
const { data: users } = useFetch<User[]>('/api/users');
const { data: orders } = useFetch<Order[]>('/api/orders');
const { data: stats } = useFetch<Stats>('/api/stats');
```

##### 11h30 - 12h00 | Autres custom hooks utiles (30 min)

**useLocalStorage** :

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

**Utilisation** :

```typescript
function Settings() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  const [lang, setLang] = useLocalStorage('lang', 'fr');
  
  return (
    <div>
      <button onClick={() => setTheme(theme === 'light' ? 'dark' : 'light')}>
        Thème : {theme}
      </button>
    </div>
  );
}
```

**useDebounce** :

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

**Utilisation** :

```typescript
function SearchBar() {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 500);
  
  useEffect(() => {
    if (debouncedSearchTerm) {
      // Recherche seulement après 500ms d'inactivité
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

**Créer ensemble en live** : useToggle, useClickOutside

---

#### 12h00 - 13h00 | 🍽️ PAUSE DÉJEUNER

---

### 🌆 APRÈS-MIDI (13h00 - 17h00)

#### 13h00 - 14h30 | Formulaire de Checkout avec Validation (1h30)
**Format** : Live coding progressif

##### 13h00 - 14h30 | Formulaire complet (1h30)

**Créer `src/pages/CheckoutPage.tsx`** :

```typescript
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
    
    // Prénom
    if (!formData.firstName.trim()) {
      newErrors.firstName = 'Le prénom est requis';
    }
    
    // Nom
    if (!formData.lastName.trim()) {
      newErrors.lastName = 'Le nom est requis';
    }
    
    // Email
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!formData.email.trim()) {
      newErrors.email = "L'email est requis";
    } else if (!emailRegex.test(formData.email)) {
      newErrors.email = 'Email invalide';
    }
    
    // Téléphone français
    const phoneRegex = /^0[1-9](?:\d{8})$/;
    if (!formData.phone.trim()) {
      newErrors.phone = 'Le téléphone est requis';
    } else if (!phoneRegex.test(formData.phone.replace(/\s/g, ''))) {
      newErrors.phone = 'Téléphone invalide (ex: 0612345678)';
    }
    
    // Adresse si livraison
    if (formData.deliveryType === 'delivery') {
      if (!formData.address.trim()) {
        newErrors.address = "L'adresse est requise";
      }
      if (!formData.city.trim()) {
        newErrors.city = 'La ville est requise';
      }
      if (!formData.postalCode.trim()) {
        newErrors.postalCode = 'Le code postal est requis';
      } else if (!/^\d{5}$/.test(formData.postalCode)) {
        newErrors.postalCode = 'Code postal invalide (5 chiffres)';
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
      setErrors(prev => {
        const newErrors = { ...prev };
        delete newErrors[name];
        return newErrors;
      });
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
          email: formData.email,
          total
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
            {isSubmitting ? 'Commande en cours...' : `Commander (${total.toFixed(2)}€)`}
          </button>
        </form>
      </div>
    </div>
  );
}

export default CheckoutPage;
```

**Page de confirmation** :

```typescript
// src/pages/OrderConfirmationPage.tsx

import { useLocation, Link } from 'react-router-dom';

function OrderConfirmationPage() {
  const location = useLocation();
  const { orderNumber, email, total } = location.state || {};
  
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
        
        <p>Merci pour votre commande</p>
        
        <div className="order-details">
          <p><strong>Numéro de commande :</strong> {orderNumber}</p>
          <p><strong>Email :</strong> {email}</p>
          <p><strong>Total :</strong> {total.toFixed(2)}€</p>
        </div>
        
        <p className="info">
          Vous recevrez un email de confirmation à l'adresse indiquée.
        </p>
        
        <div className="actions">
          <Link to="/menu" className="btn-primary">
            Nouvelle commande
          </Link>
          <Link to="/" className="btn-secondary">
            Retour à l'accueil
          </Link>
        </div>
      </div>
    </div>
  );
}

export default OrderConfirmationPage;
```

---

#### ☕ 14h30 - 14h45 | PAUSE (15 min)

---

#### 14h45 - 15h45 | Optimisations de Performance (1h)
**Format** : Présentation + Démonstrations

##### 14h45 - 15h15 | React.memo, useMemo, useCallback (30 min)

**React.memo** - Éviter les re-renders inutiles :

```typescript
// ❌ Sans React.memo
function ExpensiveComponent({ data }: { data: string }) {
  console.log('Render');
  // Calculs coûteux...
  return <div>{data}</div>;
}

// Le composant re-rend même si data n'a pas changé

// ✅ Avec React.memo
const ExpensiveComponent = React.memo(({ data }: { data: string }) => {
  console.log('Render');
  return <div>{data}</div>;
});

// Le composant re-rend seulement si data change
```

**useMemo** - Mémoriser des calculs coûteux :

```typescript
function ProductList({ products, filterTerm }: Props) {
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

**useCallback** - Mémoriser des fonctions :

```typescript
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

// Child avec React.memo ne re-rendra que si onClick change vraiment
const Child = React.memo(({ onClick }: { onClick: () => void }) => {
  console.log('Child render');
  return <button onClick={onClick}>Click</button>;
});
```

##### 15h15 - 15h45 | Quand optimiser ? (30 min)

**⚠️ RÈGLE D'OR : Ne PAS optimiser prématurément !**

```typescript
// ❌ Sur-optimisation inutile
function SimpleComponent({ name }: { name: string }) {
  const greeting = useMemo(() => `Hello ${name}`, [name]);
  const handleClick = useCallback(() => {
    console.log('Clicked');
  }, []);
  
  return (
    <div onClick={handleClick}>
      {greeting}
    </div>
  );
}

// ✅ Simple et lisible (suffisant !)
function SimpleComponent({ name }: { name: string }) {
  const handleClick = () => {
    console.log('Clicked');
  };
  
  return (
    <div onClick={handleClick}>
      Hello {name}
    </div>
  );
}
```

**Quand optimiser ?**

1. **Mesurer d'abord** avec React DevTools Profiler
2. Le composant re-rend **très souvent**
3. Le calcul est **réellement coûteux** (>50ms)
4. Vous avez **identifié un problème** de performance

**Signes qu'une optimisation est nécessaire** :
- ⏱️ UI laggy au scroll/typing
- 🐌 Animations saccadées
- 📊 Profiler montre des re-renders massifs
- 💾 Calculs qui prennent >100ms

**Optimisations générales** :
- Code splitting avec React.lazy
- Virtualization pour longues listes (react-window)
- Debouncing des inputs
- Images optimisées (WebP, lazy loading)

---

#### 15h45 - 16h45 | Build & Déploiement (1h)
**Format** : Live demo + Hands-on

##### 15h45 - 16h15 | Build de production (30 min)

**Créer le build** :

```bash
npm run build
```

**Ce qui se passe** :
1. TypeScript est compilé en JavaScript
2. Code est minifié et optimisé
3. Assets sont hashés pour le cache
4. Dossier `dist/` est créé

**Structure du build** :

```
dist/
├── assets/
│   ├── index-abc123.js     # Bundle JavaScript minifié
│   ├── index-def456.css    # CSS minifié
│   └── logo-789ghi.png     # Assets
├── index.html              # HTML point d'entrée
└── favicon.ico
```

**Prévisualiser le build** :

```bash
npm run preview
```

**Variables d'environnement** :

Créer `.env` :

```
VITE_API_URL=https://api.example.com
VITE_STRIPE_KEY=pk_test_xxxxx
```

Utiliser dans le code :

```typescript
const apiUrl = import.meta.env.VITE_API_URL;
```

**⚠️ Important** : Les variables doivent commencer par `VITE_`

##### 16h15 - 16h45 | Déploiement sur Netlify (30 min)

**Option 1 : Drag & Drop (rapide pour test)**

1. Aller sur [netlify.com](https://netlify.com)
2. Créer un compte
3. Drag & drop le dossier `dist/`
4. Site en ligne ! 🎉

**Option 2 : GitHub (recommandé pour production)**

1. Créer un repo GitHub
2. Pusher le code

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/username/repo.git
git push -u origin main
```

3. Sur Netlify :
   - "Import from Git"
   - Sélectionner le repo
   - Build settings :
     ```
     Build command: npm run build
     Publish directory: dist
     ```
4. Deploy !

**Configuration pour React Router** :

**CRITIQUE** : Sans ça, les routes ne fonctionnent pas après déploiement.

Créer `public/_redirects` :

```
/* /index.html 200
```

**Tester** :
- Aller sur l'URL du site
- Naviguer entre les pages
- Rafraîchir la page → Devrait toujours fonctionner

**Variables d'environnement sur Netlify** :

Site settings → Environment variables → Ajouter :
```
VITE_API_URL = https://api.production.com
```

---

#### 16h45 - 17h30 | Récapitulatif Final & Conclusion (45 min)

##### 16h45 - 17h00 | Quiz final (15 min)

**Questions** :

1. À quoi sert useEffect ?
2. Comment faire un appel API avec useEffect ?
3. Qu'est-ce qu'un custom hook ?
4. Quand utiliser React.memo ?
5. Comment déployer une app React ?

##### 17h00 - 17h15 | Récap des 4 jours (15 min)

**Jour 1** : Composants, Props, JSX  
**Jour 2** : useState, Événements, Immutabilité  
**Jour 3** : React Router, Context API, useReducer  
**Jour 4** : useEffect, API, Optimisations, Déploiement  

**Vous savez maintenant** :
- ✅ Créer des applications React complètes
- ✅ Gérer l'état local et global
- ✅ Naviguer entre plusieurs pages
- ✅ Faire des appels API
- ✅ Optimiser les performances
- ✅ Déployer en production

##### 17h15 - 17h30 | Pour aller plus loin & Conclusion (15 min)

**Prochaines étapes recommandées** :

**Semaine 1-2** :
- Refaire le projet de zéro
- Ajouter des features (auth, favoris, etc.)

**Mois 1** :
- Nouveau projet (Todo app, Weather app)
- Apprendre les tests (Vitest, React Testing Library)

**Mois 2-3** :
- Next.js pour SSR
- TanStack Query pour data fetching
- Bibliothèques UI (shadcn/ui)

**Mois 6+** :
- Patterns avancés
- Performance optimizations
- Contribution open source

**Ressources** :
- [react.dev](https://react.dev) - Documentation officielle
- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app)
- [Patterns.dev](https://patterns.dev) - Design patterns

**Communautés** :
- Reddit: r/reactjs
- Discord: Reactiflux
- Twitter: #ReactJS

**Mot de fin** :
"Félicitations ! 🎉 Vous avez maintenant les bases solides pour construire des applications React professionnelles. La clé maintenant : **PRATIQUER** ! Codez tous les jours, même 30 minutes. Bonne chance dans vos projets !"

---

## 📚 Projet Final (Optionnel)

### Consignes

Créer une variante du site foodtruck avec :

**Obligatoire** :
- [ ] 5+ pages minimum
- [ ] État global avec Context
- [ ] Appels API (mock ou réel)
- [ ] Formulaire avec validation
- [ ] Déployé en ligne

**Bonus** :
- [ ] Tests unitaires
- [ ] Tailwind CSS
- [ ] Animations
- [ ] LocalStorage
- [ ] Mode sombre

---

## 🎯 Critères d'Évaluation

**Niveau 1** ⭐
- [ ] useEffect basique
- [ ] Appel API simple
- [ ] Formulaire contrôlé

**Niveau 2** ⭐⭐
- [ ] Custom hooks
- [ ] Gestion loading/error
- [ ] Validation complète
- [ ] Application déployée

**Niveau 3** ⭐⭐⭐
- [ ] Optimisations (memo, useMemo)
- [ ] Code clean et organisé
- [ ] UX soignée
- [ ] Production-ready

---

## 📝 Notes Formateur

### Points critiques

1. **useEffect est délicat** : Insister sur les dépendances
2. **Async/await dans useEffect** : Montrer les erreurs
3. **Optimisation** : Marteler "mesurer d'abord"
4. **Déploiement** : Vérifier que tout le monde réussit

### Adaptations

**Si en avance** :
- Tests avec Vitest
- Animations avec Framer Motion
- WebSockets

**Si en retard** :
- Simplifier le formulaire
- Skip optimisations
- Déploiement drag&drop uniquement

---

**FIN DE LA FORMATION** 🎉

Merci d'avoir suivi ces 4 jours intensifs !

Votre feedback est précieux : [formulaire de feedback]
