# Construire un assistant pédagogique IA sur mesure (sans code fourni)

## 1. Vision globale

Tu souhaites créer une application web où tes élèves (collège/lycée) peuvent poser des questions sur un chapitre précis – ici la **fonction carré** – et obtenir des réponses conformes à **ta méthode pédagogique**, sans divagation hors sujet.

L’objectif est de construire cette application **toi-même**, en maîtrisant chaque brique, mais sans te noyer dans le code. Ce document décrit les **fondamentaux**, les **technologies** et les **principes d’architecture** pour y parvenir.

---

## 2. Architecture simplifiée
```text
Élève (navigateur)
    ↓
Frontend (HTML / CSS / JavaScript)
    ↓
Backend (Cloudflare Worker)
    ↓
Modèle DeepSeek (Workers AI)
```


- Le **Frontend** est une page web unique : formulaire de connexion, zone de chat, affichage des messages.
- Le **Backend** est un Cloudflare Worker : un petit programme JavaScript/TypeScript qui tourne sur les serveurs de Cloudflare. Il reçoit les questions, les transmet au modèle IA, et renvoie la réponse.
- Le **Modèle IA** est DeepSeek-R1, accessible via Cloudflare Workers AI (pas de clé API externe, juste un binding).
- Le **Stockage** (optionnel) utilise Workers KV pour conserver l’historique des conversations (par élève).

---

## 3. Technologies utilisées et pourquoi elles sont pertinentes

### 3.1 Cloudflare Workers
- **C’est quoi ?** Une plateforme “serverless” qui exécute du code à la périphérie du réseau (edge).
- **Pourquoi ?** Gratuit jusqu’à 100 000 requêtes/jour, déploiement simple, pas de serveur à gérer. Tu écris ton code et Cloudflare l’exécute automatiquement.

### 3.2 Workers AI
- **C’est quoi ?** L’intégration native de modèles d’IA dans Cloudflare. Tu peux appeler des modèles comme DeepSeek, Llama, etc. directement depuis ton Worker.
- **Pourquoi ?** Pas besoin de compte DeepSeek, pas de clé API à sécuriser. Le modèle est hébergé par Cloudflare et facturé à l’usage (mais le plan gratuit couvre largement une utilisation scolaire).

### 3.3 Workers KV
- **C’est quoi ?** Un stockage clé‑valeur distribué, ultra‑rapide.
- **Pourquoi ?** Pour sauvegarder l’historique des conversations (par élève) ou stocker des mots de passe individuels. C’est simple et gratuit.

### 3.4 HTML / CSS / JavaScript
- Le frontend classique, sans framework complexe. Une seule page suffit : un champ de mot de passe, un conteneur de messages, une zone de saisie.

---

## 4. Principes de construction (les briques à assembler)

### 4.1 Définir le comportement de l’IA (le “prompt système”)

Avant d’écrire la moindre ligne, tu dois formaliser **comment** l’IA doit répondre. C’est ce qu’on appelle le “system prompt”. C’est un texte que tu enverras à chaque requête pour guider le modèle.

**Analogies :**
- C’est comme une **fiche de poste** que tu donnes à un assistant : “tu es professeur, tu réponds uniquement selon ce manuel, tu ne donnes pas la réponse brute, tu reformules, tu utilises des exemples…”
- Ou comme un **chef cuisinier** qui reçoit une recette précise (ton cours) et une consigne de service.

**Contenu typique :**
- Présentation : “Tu es un assistant pédagogique pour mon cours sur la fonction carré.”
- Règles : “Tu réponds en français simple. Tu ne donnes jamais la solution complète du premier coup, tu guides l’élève. Si la réponse ne se trouve pas dans le cours ci-dessous, tu dis ‘Je ne trouve pas cette information dans le cours’.”
- Le cours lui‑même (texte complet ou extraits). Pour commencer, tu peux coller ton cours dans le prompt. Plus tard, tu pourras faire du “retrieval” (RAG) pour ne passer que les passages pertinents.

### 4.2 Authentification des élèves

Deux approches possibles :

- **Mot de passe unique** (facile) : tous les élèves utilisent le même mot de passe. Tu le stockes dans une variable d’environnement du Worker.
- **Mots de passe individuels** (plus avancé) : chaque élève a son propre code. Tu les stockes dans Workers KV (clé = identifiant élève, valeur = mot de passe hashé).

Le principe est le même : avant d’accepter une question, le Worker vérifie que le mot de passe reçu correspond à ce qui est attendu.

### 4.3 Communication frontend ↔ backend

Le frontend (page web) envoie une requête HTTP POST au Worker, contenant :
- Le mot de passe
- La question de l’élève
- (Optionnel) un identifiant de session pour l’historique

Le Worker :
1. Vérifie le mot de passe.
2. Construit le message complet en ajoutant le system prompt et la question.
3. Appelle le modèle IA via Workers AI.
4. Sauvegarde la conversation (si demandé) dans KV.
5. Renvoie la réponse au frontend.

**Analogie :** C’est comme un **serveur dans un restaurant**. Le client (frontend) donne la commande (question), le serveur la transmet au chef (modèle IA) avec des instructions spéciales (system prompt), puis rapporte le plat (réponse).

### 4.4 Gestion de l’historique (optionnel)

Tu peux permettre à chaque élève de revoir ses conversations. Pour cela, le frontend génère un `sessionId` unique (stocké dans le `localStorage` du navigateur) et l’envoie à chaque requête. Le Worker utilise ce `sessionId` comme clé dans KV pour sauvegarder la liste des messages.

**Attention :** KV a une limite de taille (1 Mo par valeur) ; pour des historiques longs, il faudra paginer ou utiliser un autre stockage.

### 4.5 Déploiement

Cloudflare fournit un outil en ligne de commande (`wrangler`) pour déployer ton Worker. Une fois configuré, la commande `wrangler deploy` publie instantanément ton application à une URL du type `https://mon-assistant.workers.dev`.

---

## 5. Organisation des fichiers (structure type)

```text
mon-assistant/
├── public/
│   ├── index.html      # structure HTML
│   ├── style.css       # apparence (couleurs, mise en page)
│   └── app.js          # code JavaScript du frontend
├── src/
│   └── index.ts        # le Worker (backend)
├── wrangler.toml       # configuration (liens avec KV, AI, variables)
└── package.json        # dépendances (si besoin)
```


- **public/** : tout ce qui est accessible au navigateur.
- **src/** : le code du Worker.
- **wrangler.toml** : c’est ici que tu définis le nom de ton Worker, les bindings (KV, AI) et les variables d’environnement (comme le mot de passe).

---

## 6. Séquence d’exécution (détaillée)

1. **L’élève** ouvre l’URL de l’application. Le navigateur charge `index.html`, `style.css`, `app.js`.
2. Il entre le mot de passe et clique sur “Se connecter”. Le frontend enregistre le mot de passe dans le `localStorage` (ou simplement en mémoire).
3. L’élève tape une question et valide. Le frontend envoie une requête POST à `/api/chat` avec `{ password, message, sessionId }`.
4. Le Worker reçoit la requête. Il vérifie que `password` correspond à la variable d’environnement `CHAT_PASSWORD` (ou à l’entrée KV correspondante).
5. Si le mot de passe est correct, il construit un tableau de messages pour l’IA :
   - `{ role: "system", content: SYSTEM_PROMPT }`
   - `{ role: "user", content: message }`
6. Il appelle Workers AI avec le modèle `@cf/deepseek-ai/deepseek-r1-distill-qwen-32b` (par exemple) et ces messages.
7. Il reçoit la réponse de l’IA.
8. (Optionnel) Il sauvegarde l’échange dans KV : `{ sessionId: [ ...anciens messages, nouveau message utilisateur, réponse IA] }`.
9. Il renvoie la réponse au frontend au format JSON.
10. Le frontend affiche la réponse dans la zone de chat.

---

## 7. Analogies pour mieux comprendre les concepts

- **Workers AI** : un **chef spécialisé** qui connaît beaucoup de choses, mais tu dois lui donner des instructions précises (system prompt) pour qu’il cuisine à ta façon.
- **System prompt** : la **recette de cuisine** que tu donnes au chef. Sans elle, il ferait à sa guise.
- **KV storage** : un **cahier de notes** partagé où chaque élève a sa page (sa clé) sur laquelle tu écris l’historique.
- **Authentification** : la **clé de la salle** ; sans elle, personne ne peut entrer.
- **Frontend / Backend** : comme un **restaurant** : la salle (frontend) où les clients s’assoient, et la cuisine (backend) où sont préparées les réponses.

---

## 8. Considérations importantes

### 8.1 Prompt engineering
La qualité des réponses dépend surtout du **system prompt**. Prends le temps de rédiger :
- Des consignes claires (“ne donne pas la réponse, guide”)
- Ton cours (bien structuré, avec définitions, propriétés, exemples)
- Des exemples de bonnes et mauvaises réponses (optionnel mais puissant)

### 8.2 Coûts
- Cloudflare Workers : 100 000 requêtes/jour gratuites.
- Workers AI : 10 000 requêtes/jour gratuites (pour les modèles comme DeepSeek). Une classe de 30 élèves posant 5 questions par jour = 150 requêtes → bien dans les limites.
- KV : opérations gratuites jusqu’à 1 million de lectures/jour.

### 8.3 Sécurité et vie privée
- Le mot de passe est envoyé en clair (HTTPS). Pour une utilisation scolaire, c’est acceptable.
- Ne stocke pas d’informations sensibles dans KV sans hashage.
- Les conversations peuvent être supprimées automatiquement après un certain temps (via `expirationTtl`).

### 8.4 Extensions possibles
- **Authentification individuelle** : avec des mots de passe stockés dans KV.
- **Streaming** : renvoyer la réponse morceau par morceau pour un effet “l’IA écrit en direct”.
- **Recherche vectorielle (RAG)** : si ton cours est volumineux, tu peux ne passer que les passages pertinents à l’IA.
- **Export des conversations** : permettre à l’élève de télécharger son historique.

---

## 9. Conclusion

En suivant ces principes, tu vas construire une application **100% maîtrisée**, hébergée gratuitement, et qui répondra aux élèves avec ta propre pédagogie.

Les briques sont simples :
- Un frontend basique (HTML/CSS/JS)
- Un Worker Cloudflare qui fait le lien entre le frontend et l’IA
- Le modèle DeepSeek via Workers AI
- (Optionnel) KV pour l’historique

Tu ne réinventes pas la roue, tu assembles des composants modernes et robustes. Le résultat sera professionnel, personnalisable et totalement sous ton contrôle.

**Prochaine étape** : choisis un nom pour ton Worker, crée un compte Cloudflare, installe `wrangler`, et suis la documentation pour créer ton premier Worker. Tu pourras ensuite ajouter les briques une par une, en testant après chaque ajout.
