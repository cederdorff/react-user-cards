# React Øvelser: Lister, Keys & API Data

Disse opgaver understøtter undervisningen om lister, keys og datahentning fra eksterne kilder.

---

## 📋 Opgave 6: Lister & Keys - Forstå key-attributten

**Teori:**
Keys er et vigtigt koncept i React når du renderer lister. Keys hjælper React med at identificere hvilke elementer der har ændret sig, blev tilføjet eller fjernet. Dette gør React mere effektiv og forhindrer bugs.

### Step 6.1: Eksperimenter med keys

1. **Test uden keys:**
   - Fjern midlertidigt `key={user.id}` fra din UserList komponent
   - Åbn Developer Tools i browseren og kig efter warnings
   - Hvad sker der i konsollen?

2. **Test med dårlige keys:**
   ```jsx
   // ❌ Dårligt eksempel - brug ikke array index som key
   {users.map((user, index) => (
     <UserCard user={user} key={index} onDelete={handleDeleteUser} />
   ))}
   ```
   - Prøv at slette den første bruger og se hvad der sker
   - Hvad er problemet med at bruge index som key?

3. **Korrekt brug af keys:**
   ```jsx
   // ✅ Godt eksempel - brug unik id som key
   {users.map(user => (
     <UserCard user={user} key={user.id} onDelete={handleDeleteUser} />
   ))}
   ```

### Step 6.2: Skriv en kommentar om keys

Tilføj en kommentar i din UserList komponent der forklarer:
- Hvad keys er
- Hvorfor de er vigtige
- Hvad der gør en god key

---

## 🌐 Opgave 7: Multiple API Sources - Hent data fra forskellige kilder

**Teori:**
I praksis henter man ofte data fra forskellige API'er. Du lærer at arbejde med forskellige datastrukturer og håndtere asynkron datahentning.

### Step 7.1: Hent posts fra JSONPlaceholder

1. **Opret en ny komponent `PostList.jsx`:**
   ```jsx
   import { useState, useEffect } from 'react';

   function PostList() {
     const [posts, setPosts] = useState([]);
     const [loading, setLoading] = useState(true);
     const [error, setError] = useState(null);

     useEffect(() => {
       async function fetchPosts() {
         try {
           setLoading(true);
           const response = await fetch('https://jsonplaceholder.typicode.com/posts');
           if (!response.ok) {
             throw new Error(`HTTP error! status: ${response.status}`);
           }
           const data = await response.json();
           setPosts(data.slice(0, 10)); // Vis kun første 10 posts
         } catch (err) {
           setError(err.message);
         } finally {
           setLoading(false);
         }
       }
       
       fetchPosts();
     }, []);

     if (loading) return <p>Loading posts...</p>;
     if (error) return <p>Error: {error}</p>;

     return (
       <div>
         <h2>Latest Posts</h2>
         <ul>
           {posts.map(post => (
             <li key={post.id}>
               <h3>{post.title}</h3>
               <p>{post.body.substring(0, 100)}...</p>
             </li>
           ))}
         </ul>
       </div>
     );
   }

   export default PostList;
   ```

2. **Tilføj PostList til din App:**
   - Importér og vis PostList komponenten i App.jsx
   - Placer den under UserList

### Step 7.2: Sammenlign datastrukturer

1. **Undersøg data:**
   - Åbn `https://jsonplaceholder.typicode.com/posts` i browseren
   - Sammenlign med `https://raw.githubusercontent.com/cederdorff/race/master/data/users.json`
   - Hvad er forskellene i datastruktur?

2. **Skriv kommentarer:**
   - Forklar forskellen mellem users og posts data
   - Hvordan påvirker forskellige datastrukturer din kode?

---

## ⚡ Opgave 8: Loading States & Error Handling

**Teori:**
Når du henter data fra API'er skal du håndtere forskellige tilstande: loading, success og error. Det giver en bedre brugeroplevelse.

### Step 8.1: Forbedret loading state

1. **Opret en LoadingSpinner komponent:**
   ```jsx
   function LoadingSpinner({ message = "Loading..." }) {
     return (
       <div className="loading-spinner">
         <div className="spinner"></div>
         <p>{message}</p>
       </div>
     );
   }
   ```

2. **Tilføj CSS for spinner:**
   ```css
   .loading-spinner {
     display: flex;
     flex-direction: column;
     align-items: center;
     padding: 20px;
   }

   .spinner {
     border: 4px solid #f3f3f3;
     border-top: 4px solid #3498db;
     border-radius: 50%;
     width: 40px;
     height: 40px;
     animation: spin 2s linear infinite;
   }

   @keyframes spin {
     0% { transform: rotate(0deg); }
     100% { transform: rotate(360deg); }
   }
   ```

### Step 8.2: Error handling komponent

1. **Opret en ErrorMessage komponent:**
   ```jsx
   function ErrorMessage({ error, onRetry }) {
     return (
       <div className="error-message">
         <h3>Noget gik galt!</h3>
         <p>{error}</p>
         {onRetry && (
           <button onClick={onRetry} className="retry-btn">
             Prøv igen
           </button>
         )}
       </div>
     );
   }
   ```

### Step 8.3: Implementer retry funktionalitet

1. **Uddeling din fetchPosts funktion:**
   ```jsx
   const fetchPosts = useCallback(async () => {
     try {
       setLoading(true);
       setError(null);
       const response = await fetch('https://jsonplaceholder.typicode.com/posts');
       if (!response.ok) {
         throw new Error(`HTTP error! status: ${response.status}`);
       }
       const data = await response.json();
       setPosts(data.slice(0, 10));
     } catch (err) {
       setError(err.message);
     } finally {
       setLoading(false);
     }
   }, []);
   ```

2. **Brug ErrorMessage med retry:**
   ```jsx
   if (error) return <ErrorMessage error={error} onRetry={fetchPosts} />;
   ```

---

## 🔄 Opgave 9: Data Synchronization - Opdater data fra API

**Teori:**
I rigtige applikationer skal du ofte synkronisere lokale ændringer med serveren. Du lærer at sende data til API'er og håndtere response.

### Step 9.1: Simuler POST request

1. **Opdater handleSubmit for at "synkronisere" med server:**
   ```jsx
   async function handleSubmit(e) {
     e.preventDefault();
     const form = e.target;
     
     const newUser = {
       name: form.name.value,
       mail: form.mail.value,
       title: form.title.value,
       image: form.image.value
     };

     try {
       // Simuler POST til server (JSONPlaceholder returnerer fake data)
       const response = await fetch('https://jsonplaceholder.typicode.com/users', {
         method: 'POST',
         headers: {
           'Content-Type': 'application/json',
         },
         body: JSON.stringify(newUser),
       });

       if (!response.ok) {
         throw new Error(`HTTP error! status: ${response.status}`);
       }

       const serverUser = await response.json();
       console.log('Server response:', serverUser);

       // Tilføj med lokal id (da JSONPlaceholder ikke gemmer rigtig data)
       const userWithId = {
         ...newUser,
         id: crypto.randomUUID()
       };

       setUsers([...users, userWithId]);
       form.reset();
       
       // Vis success besked
       alert('Bruger tilføjet!');
     } catch (error) {
       console.error('Fejl ved tilføjelse:', error);
       alert('Kunne ikke tilføje bruger: ' + error.message);
     }
   }
   ```

### Step 9.2: Simuler DELETE request

1. **Opdater handleDeleteUser:**
   ```jsx
   async function handleDeleteUser(id) {
     try {
       // Simuler DELETE til server
       const response = await fetch(`https://jsonplaceholder.typicode.com/users/${id}`, {
         method: 'DELETE',
       });

       if (!response.ok) {
         throw new Error(`HTTP error! status: ${response.status}`);
       }

       // Fjern fra lokal state
       setUsers(users.filter(user => user.id !== id));
       console.log('User deleted from server');
     } catch (error) {
       console.error('Fejl ved sletning:', error);
       alert('Kunne ikke slette bruger: ' + error.message);
     }
   }
   ```

---

## 🏗️ Opgave 10: Custom Hook - Abstraher API logik

**Teori:**
Custom hooks er en måde at genbruge logik mellem komponenter. Du kan abstrahere API calls til et custom hook, som gør din kode mere DRY (Don't Repeat Yourself).

### Step 10.1: Opret useApi hook

1. **Opret `hooks/useApi.js`:**
   ```jsx
   import { useState, useEffect, useCallback } from 'react';

   function useApi(url, options = {}) {
     const [data, setData] = useState(null);
     const [loading, setLoading] = useState(true);
     const [error, setError] = useState(null);

     const fetchData = useCallback(async () => {
       try {
         setLoading(true);
         setError(null);
         
         const response = await fetch(url, options);
         if (!response.ok) {
           throw new Error(`HTTP error! status: ${response.status}`);
         }
         
         const result = await response.json();
         setData(result);
       } catch (err) {
         setError(err.message);
       } finally {
         setLoading(false);
       }
     }, [url, options]);

     useEffect(() => {
       fetchData();
     }, [fetchData]);

     const refetch = useCallback(() => {
       fetchData();
     }, [fetchData]);

     return { data, loading, error, refetch };
   }

   export default useApi;
   ```

### Step 10.2: Brug useApi hook

1. **Refaktorér PostList til at bruge useApi:**
   ```jsx
   import useApi from '../hooks/useApi';

   function PostList() {
     const { data: posts, loading, error, refetch } = useApi('https://jsonplaceholder.typicode.com/posts');

     if (loading) return <LoadingSpinner message="Loading posts..." />;
     if (error) return <ErrorMessage error={error} onRetry={refetch} />;

     return (
       <div>
         <h2>Latest Posts</h2>
         <button onClick={refetch}>Refresh Posts</button>
         <ul>
           {posts?.slice(0, 10).map(post => (
             <li key={post.id}>
               <h3>{post.title}</h3>
               <p>{post.body.substring(0, 100)}...</p>
             </li>
           ))}
         </ul>
       </div>
     );
   }
   ```

---

## 🎯 Opgave 11: Performance - React.memo og useMemo

**Teori:**
Når du arbejder med lister kan performance blive et problem. React tilbyder værktøjer som React.memo og useMemo til at optimere gendering.

### Step 11.1: Memoize UserCard komponenten

1. **Opdater UserCard.jsx:**
   ```jsx
   import React, { useState, useEffect, memo } from "react";

   const UserCard = memo(function UserCard({ user, onDelete }) {
     console.log('UserCard rendered for:', user.name);
     
     // ... resten af din UserCard kode
   });

   export default UserCard;
   ```

### Step 11.2: Brug useMemo for tunge beregninger

1. **Tilføj en tung beregning i UserList:**
   ```jsx
   import { useMemo } from 'react';

   function UserList({ users, onDelete }) {
     // Simuler en tung beregning
     const userStats = useMemo(() => {
       console.log('Calculating user stats...');
       return {
         total: users.length,
         domains: [...new Set(users.map(user => user.mail?.split('@')[1]))].length,
         avgNameLength: users.reduce((sum, user) => sum + user.name.length, 0) / users.length || 0
       };
     }, [users]);

     return (
       <div>
         <div className="user-stats">
           <p>Total users: {userStats.total}</p>
           <p>Unique domains: {userStats.domains}</p>
           <p>Average name length: {userStats.avgNameLength.toFixed(1)}</p>
         </div>
         
         <div className="user-grid">
           {users.map(user => (
             <UserCard 
               key={user.id} 
               user={user} 
               onDelete={onDelete}
             />
           ))}
         </div>
       </div>
     );
   }
   ```

---

## 📊 Opgave 12: Data Transformation - Bearbejd API data

**Teori:**
API data kommer ikke altid i det format du har brug for. Du lærer at transformere og filtrere data, samt kombinere data fra flere kilder.

### Step 12.1: Kombiner users og posts data

1. **Opret en ny komponent `UserPostsView.jsx`:**
   ```jsx
   import { useState, useEffect } from 'react';

   function UserPostsView() {
     const [users, setUsers] = useState([]);
     const [posts, setPosts] = useState([]);
     const [loading, setLoading] = useState(true);

     useEffect(() => {
       async function fetchData() {
         try {
           setLoading(true);
           
           // Hent data parallelt
           const [usersResponse, postsResponse] = await Promise.all([
             fetch('https://jsonplaceholder.typicode.com/users'),
             fetch('https://jsonplaceholder.typicode.com/posts')
           ]);

           const usersData = await usersResponse.json();
           const postsData = await postsResponse.json();

           setUsers(usersData);
           setPosts(postsData);
         } catch (error) {
           console.error('Error fetching data:', error);
         } finally {
           setLoading(false);
         }
       }

       fetchData();
     }, []);

     // Kombiner data - find posts per user
     const usersWithPosts = users.map(user => ({
       ...user,
       posts: posts.filter(post => post.userId === user.id)
     }));

     if (loading) return <div>Loading...</div>;

     return (
       <div>
         <h2>Users with their Posts</h2>
         {usersWithPosts.map(user => (
           <div key={user.id} className="user-posts">
             <h3>{user.name} ({user.posts.length} posts)</h3>
             <p>{user.email}</p>
             {user.posts.slice(0, 2).map(post => (
               <div key={post.id} className="post-preview">
                 <h4>{post.title}</h4>
                 <p>{post.body.substring(0, 100)}...</p>
               </div>
             ))}
           </div>
         ))}
       </div>
     );
   }

   export default UserPostsView;
   ```

---

## 🎨 Bonus Opgaver

### Bonus 1: Infinite Scroll
Implementer infinite scroll for posts listen - hent flere posts når brugeren scroller ned.

### Bonus 2: Real-time updates
Brug `setInterval` til at simulere real-time updates af data hver 30. sekund.

### Bonus 3: Advanced Filtering
Implementer avancerede filtre der arbejder på data fra flere API kilder samtidig.

---

## 📚 Refleksionsspørgsmål

Efter du har gennemført opgaverne, overvej disse spørgsmål:

1. **Keys:** Hvorfor er unique keys vigtige i React lister?
2. **API Integration:** Hvad er fordelene og ulemperne ved at hente data fra flere API kilder?
3. **Error Handling:** Hvordan påvirker god error handling brugeroplevelsen?
4. **Performance:** Hvornår skal du bruge React.memo og useMemo?
5. **Code Organization:** Hvordan hjælper custom hooks med at organisere kode?

---

## 🔗 Ressourcer

- [React Lists and Keys](https://reactjs.org/docs/lists-and-keys.html)
- [JSONPlaceholder API](https://jsonplaceholder.typicode.com/)
- [JavaScript Fetch API](https://javascript.info/fetch)
- [React Performance](https://reactjs.org/docs/optimizing-performance.html)

God arbejdslyst! 🚀
