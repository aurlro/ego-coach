# Architecture de Données

Ce document décrit l'architecture de données de l'application, conçue pour être évolutive et prête pour une future intégration avec une base de données distante comme Supabase.

## 1. Couche d'Abstraction des Données (`data-store.js`)

Toute la logique d'accès aux données est centralisée dans `assets/js/data-store.js`. Ce fichier expose une fonction `createDataStore` qui retourne un objet avec les méthodes suivantes :

-   `getCurrentUser()`: Retourne l'identifiant de l'utilisateur actuellement simulé.
-   `setCurrentUser(newUser)`: Change l'utilisateur actuel.
-   `getData(key, fallback)`: Récupère des données pour l'utilisateur actuel.
-   `saveData(key, data)`: Sauvegarde des données pour l'utilisateur actuel.
-   `clearAllUserData()`: Supprime toutes les données de l'utilisateur actuel.

### Implémentation Actuelle : `localStorage`

Pour l'instant, le `dataStore` utilise le `localStorage` du navigateur. Les données sont stockées sous des clés préfixées par l'identifiant de l'utilisateur, assurant ainsi que les données de chaque utilisateur sont isolées.

Le format de la clé est : `ego-coach-data::{userId}::{dataKey}`

Exemple : `ego-coach-data::Jules::journal`

## 2. Simulation d'Utilisateur

Pour faciliter le développement et tester la logique multi-utilisateurs, une fonctionnalité de simulation a été ajoutée. Elle est accessible en bas de la barre de navigation latérale.

-   **Pour changer d'utilisateur :** Entrez un nom dans le champ de texte et cliquez sur "Go".
-   L'application changera de contexte, et toutes les données (comme le journal) seront propres à cet utilisateur.
-   Les données sont persistées dans le `localStorage`, donc vous pouvez fermer et rouvrir le navigateur, les données de chaque utilisateur seront conservées.

## 3. Guide pour l'Intégration d'une Base de Données (Supabase)

Lorsque le moment sera venu de connecter une véritable base de données, la transition sera grandement simplifiée grâce à la couche d'abstraction.

**Étapes à suivre :**

1.  **Initialiser le client Supabase :** Dans `app.js`, initialisez le client Supabase avec vos clés d'API.

2.  **Modifier `data-store.js` :** Le seul fichier à modifier sera `data-store.js`. Voici comment les fonctions pourraient être adaptées :

    -   **`saveData(key, data)` :**
        -   Au lieu de `localStorage.setItem`, cette fonction fera un appel à Supabase.
        -   Exemple : `supabase.from(key).upsert({ id: userId, data: data })`.
        -   `key` pourrait correspondre au nom de la table (ex: `journal`).

    -   **`getData(key, fallback)` :**
        -   Au lieu de `localStorage.getItem`, cette fonction fera un appel `select` à Supabase.
        -   Exemple : `supabase.from(key).select('data').eq('id', userId)`.

    -   **Gestion de l'authentification :**
        -   `getCurrentUser()` et `setCurrentUser()` devront être remplacés par la gestion de l'authentification de Supabase (`supabase.auth.user()`).
        -   La simulation d'utilisateur devra être retirée au profit d'un vrai système de connexion.

Cette approche garantit que le reste de l'application (`journalModule`, `homeModule`, etc.) n'aura pas besoin d'être modifié, car il dépend uniquement de l'interface du `dataStore` et non de son implémentation.

## 4. Installation du Projet

Ce projet est une application web statique qui ne nécessite pas de processus de build complexe.

**Prérequis :**
-   Un navigateur web moderne (Chrome, Firefox, Edge).
-   Python 3 (pour le serveur de développement).

**Étapes d'installation :**

1.  **Clonez le dépôt :**
    ```bash
    git clone https://github.com/votre-repo/ego-coach.git
    cd ego-coach
    ```

2.  **Lancez un serveur de développement :**
    La manière la plus simple de lancer un serveur local est d'utiliser le module `http.server` de Python.
    ```bash
    python3 -m http.server 8000
    ```
    Si vous avez `npx`, vous pouvez aussi utiliser `serve` :
    ```bash
    npx serve .
    ```

3.  **Ouvrez l'application :**
    Ouvrez votre navigateur et allez à l'adresse `http://localhost:8000`.
