# Learning Log: Bases de données SQL, SQLAlchemy et Alembic

**Date**: 27 janvier 2026  
**Sujet**: Introduction aux bases de données relationnelles et à l'écosystème SQLAlchemy

---

## 1. Les bases de données SQL: Le concept fondamental

### Qu'est-ce qu'une base de données relationnelle?

**Analogie**: Imagine une bibliothèque bien organisée avec des fiches cartonnées.

- Chaque **table** = un tiroir de fiches (auteurs, livres, emprunts)
- Chaque **ligne** = une fiche individuelle
- Chaque **colonne** = un champ sur la fiche (nom, prénom, date)
- Les **relations** = des références entre fiches (ce livre → cet auteur)

**Exemple concret**:
```
Table: authors
+----+---------------+
| id | name          |
+----+---------------+
| 1  | Victor Hugo   |
| 2  | Albert Camus  |
+----+---------------+

Table: books
+----+------------------+-----------+
| id | title            | author_id |
+----+------------------+-----------+
| 1  | Les Misérables   | 1         |
| 2  | L'Étranger       | 2         |
+----+------------------+-----------+
```

### SQL: Le langage pour parler avec la base

**SQL** (Structured Query Language) = la langue que comprend la base de données.
```sql
-- Créer une table
CREATE TABLE authors (
    id INTEGER PRIMARY KEY,
    name VARCHAR(100)
);

-- Ajouter des données
INSERT INTO authors (name) VALUES ('Victor Hugo');

-- Lire les données
SELECT * FROM authors WHERE name = 'Victor Hugo';

-- Modifier
UPDATE authors SET name = 'V. Hugo' WHERE id = 1;

-- Supprimer
DELETE FROM authors WHERE id = 1;
```

### Le problème du SQL brut

Quand tu programmes en Python et que tu veux utiliser une base de données:
```python
import sqlite3

# Tu dois écrire du SQL en texte
cursor.execute("INSERT INTO authors (name) VALUES (?)", ("Victor Hugo",))

# Les résultats sont des tuples sans structure
row = cursor.fetchone()
print(row[0], row[1])  # C'est quoi déjà l'index de "name"?
```

**Problèmes**:
- Tu mélanges deux langages (Python + SQL en string)
- Pas d'auto-complétion
- Erreurs détectées seulement à l'exécution
- Code répétitif pour chaque opération
- Difficile de changer de base de données (SQLite → PostgreSQL)

---

## 2. SQLAlchemy: Parler SQL en Python

### Le concept d'ORM (Object-Relational Mapping)

**ORM** = Pont entre le monde des objets Python et le monde des tables SQL.

**Analogie**: C'est comme avoir un traducteur automatique entre toi (qui parle Python) et la base de données (qui parle SQL).

Au lieu d'écrire du SQL, tu manipules des **objets Python** normaux:
```python
# Tu crées un objet Python
author = Author(name="Victor Hugo")

# SQLAlchemy traduit automatiquement en:
# INSERT INTO authors (name) VALUES ('Victor Hugo')
```

### Les trois composants principaux de SQLAlchemy

#### 1. Le moteur (Engine) - La connexion

**Analogie**: Le câble qui relie ton ordinateur à la base de données.
```python
from sqlalchemy import create_engine

engine = create_engine("sqlite:///library.db")
```

- Un seul moteur pour toute l'application
- Gère le pool de connexions (réutilise les connexions)
- S'adapte automatiquement à ta base (SQLite, PostgreSQL, MySQL...)

#### 2. La session - La conversation

**Analogie**: Une session = une conversation téléphonique avec la base.
```python
from sqlalchemy.orm import sessionmaker

SessionLocal = sessionmaker(bind=engine)  # Fabrique de sessions

# Ouvrir une session
db = SessionLocal()
# ... faire des opérations ...
db.close()  # Fermer la session
```

**Pourquoi des sessions?**
- Tu groupes plusieurs opérations ensemble
- Soit TOUT réussit, soit RIEN (transaction atomique)
- Comme une conversation: tu ouvres, tu parles, tu raccroches

#### 3. Les modèles (Models) - Les structures de données

**Analogie**: Les modèles = les plans architecturaux de tes tables.
```python
from sqlalchemy import Column, Integer, String
from sqlalchemy.ext.declarative import declarative_base

Base = declarative_base()

class Author(Base):
    __tablename__ = "authors"  # Nom de la table SQL
    
    id = Column(Integer, primary_key=True)
    name = Column(String(100))
```

**Ce que fait SQLAlchemy**:
- Traduit cette classe Python en table SQL
- Permet de manipuler les lignes comme des objets Python

### Le workflow complet
```python
# 1. Définir le modèle (une fois)
class Author(Base):
    __tablename__ = "authors"
    id = Column(Integer, primary_key=True)
    name = Column(String(100))

# 2. Créer la table (une fois)
Base.metadata.create_all(engine)

# 3. Utiliser (autant de fois que nécessaire)
db = SessionLocal()

# Créer
author = Author(name="Victor Hugo")
db.add(author)
db.commit()  # Sauvegarde

# Lire
authors = db.query(Author).all()
for author in authors:
    print(author.name)  # Accès comme attribut Python!

# Modifier
author.name = "V. Hugo"
db.commit()

# Supprimer
db.delete(author)
db.commit()

db.close()
```

### Les avantages de SQLAlchemy

**1. Indépendance de la base**
```python
# Même code marche avec SQLite, PostgreSQL, MySQL...
# Seul le connection string change
engine = create_engine("sqlite:///dev.db")  # Dev
engine = create_engine("postgresql://...")  # Production
```

**2. Sécurité automatique**
```python
# Protégé automatiquement contre les injections SQL
name = input("Nom: ")  # Même si l'utilisateur tape du SQL malicieux
author = Author(name=name)  # Toujours sécurisé
```

**3. Relations entre tables**
```python
class Author(Base):
    books = relationship("Book")  # Un auteur a plusieurs livres

class Book(Base):
    author_id = Column(Integer, ForeignKey('authors.id'))
    author = relationship("Author")  # Un livre a un auteur

# Utilisation intuitive
author = db.query(Author).first()
for book in author.books:  # Navigation naturelle!
    print(book.title)
```

**4. Code lisible et maintenable**
```python
# Au lieu de:
cursor.execute("""
    SELECT * FROM authors 
    WHERE name LIKE ? 
    ORDER BY name 
    LIMIT 5
""", ("%Hugo%",))

# Tu écris:
authors = db.query(Author)\
    .filter(Author.name.like("%Hugo%"))\
    .order_by(Author.name)\
    .limit(5)\
    .all()
```

---

## 3. Le problème que SQLAlchemy ne résout pas

### Scénario: Ton application évolue

**Version 1** (juillet 2026): Tu déploies ton POS à Cotonou
```python
class Product(Base):
    id = Column(Integer, primary_key=True)
    name = Column(String(100))
    price = Column(Numeric(10, 2))
```

La base contient maintenant **1000 produits, 5000 commandes**.

**Version 2** (septembre 2026): Tu veux ajouter des catégories
```python
class Product(Base):
    id = Column(Integer, primary_key=True)
    name = Column(String(100))
    price = Column(Numeric(10, 2))
    category = Column(String(50))  # NOUVEAU!
```

**Le problème**: Comment modifier la table existante sans perdre les 1000 produits?
```python
# ❌ MAUVAIS: Détruit tout
Base.metadata.drop_all(engine)
Base.metadata.create_all(engine)

# ❌ SQL manuel: Compliqué à tracker
cursor.execute("ALTER TABLE products ADD COLUMN category VARCHAR(50)")
```

**Ce qu'il te faut**:
- Versionner les changements du schéma (comme Git pour le code)
- Appliquer les changements de façon contrôlée
- Pouvoir revenir en arrière si problème

---

## 4. Alembic: Le versioning de ton schéma

### Le concept: Migrations

**Analogie**: Alembic = Git pour ton schéma de base de données.

Une **migration** = un fichier qui décrit **comment passer d'une version à la suivante**.
```
Version 1 → [Migration 2] → Version 2 → [Migration 3] → Version 3
```

Chaque migration contient:
- `upgrade()`: Comment aller vers la nouvelle version
- `downgrade()`: Comment revenir à l'ancienne version

### Comment ça marche

**1. Tu modifies ton modèle SQLAlchemy**
```python
class Product(Base):
    category = Column(String(50))  # Ajout
```

**2. Alembic détecte le changement**
```bash
alembic revision --autogenerate -m "add category"
```

**3. Alembic génère un fichier de migration**
```python
# alembic/versions/abc123_add_category.py
def upgrade():
    op.add_column('products', sa.Column('category', sa.String(50)))

def downgrade():
    op.drop_column('products', 'category')
```

**4. Tu appliques la migration**
```bash
alembic upgrade head  # Applique tous les changements
```

**Résultat**: Ta table `products` a maintenant la colonne `category`, et **toutes les données existantes sont préservées**! ✅

### Le tracking des versions

Alembic crée une table spéciale `alembic_version` qui sait où tu en es:
```
alembic_version
+-----------------+
| version_num     |
+-----------------+
| abc123          |  ← Tu es à la migration "abc123"
+-----------------+
```

Quand tu fais `alembic upgrade head`:
1. Alembic regarde la version actuelle
2. Applique toutes les migrations manquantes dans l'ordre
3. Met à jour `alembic_version`

### Les avantages d'Alembic

**1. Reproductibilité**
```bash
# Développement en Bruxelles
alembic upgrade head

# Production à Cotonou
alembic upgrade head  # Même commande, même résultat!
```

**2. Historique complet**
```
alembic/versions/
├── 001_initial_schema.py
├── 002_add_categories.py
├── 003_add_stock.py
└── 004_create_orders.py
```

Tu as la **timeline complète** de l'évolution de ta base.

**3. Rollback facile**
```bash
alembic downgrade -1  # Revenir à la version précédente
```

**4. Déploiement multi-environnements**

Tu peux avoir plusieurs restaurants avec des versions différentes:
- Restaurant A: version 005
- Restaurant B: version 008
- Restaurant C: version 010

Alembic applique automatiquement les bonnes migrations à chacun.

---

## 5. SQLAlchemy + Alembic: Le duo gagnant

### La division du travail
```
SQLAlchemy: "Comment UTILISER la base au quotidien"
- Créer, lire, modifier, supprimer des données
- Gérer les relations entre tables
- Construire des requêtes complexes

Alembic: "Comment FAIRE ÉVOLUER le schéma dans le temps"
- Versionner les changements de structure
- Appliquer les modifications sans perte de données
- Synchroniser dev/staging/production
```

### Workflow complet

**Phase 1: Développement initial**
```python
# 1. Définir les modèles
class Author(Base):
    id = Column(Integer, primary_key=True)
    name = Column(String(100))

# 2. Créer la première migration
alembic revision --autogenerate -m "initial schema"
alembic upgrade head
```

**Phase 2: Ajout de fonctionnalités**
```python
# 1. Modifier les modèles
class Author(Base):
    id = Column(Integer, primary_key=True)
    name = Column(String(100))
    bio = Column(Text)  # NOUVEAU

# 2. Créer la migration
alembic revision --autogenerate -m "add author bio"

# 3. Tester localement
alembic upgrade head

# 4. Commiter le code + la migration dans Git
git add models.py alembic/versions/002_add_author_bio.py
git commit -m "Add author bio field"

# 5. Déployer en production
# Sur le serveur:
git pull
alembic upgrade head  # Applique la nouvelle migration
```

**Phase 3: Utilisation quotidienne** (SQLAlchemy)
```python
db = SessionLocal()
author = Author(name="Camus", bio="Philosophe et écrivain")
db.add(author)
db.commit()
db.close()
```

---

## 6. Pourquoi c'est la référence en Python

### SQLAlchemy

- **18 ans d'existence** (depuis 2006)
- Utilisé par Reddit, Dropbox, Yelp
- L'ORM le plus mature et flexible de Python
- Standard recommandé par FastAPI

### Alembic

- Créé par le même auteur que SQLAlchemy
- La solution officielle pour les migrations SQLAlchemy
- Pas de vraie alternative sérieuse
- Standard de l'industrie

### Pour ton POS

Cette stack est **parfaite** pour:
- Application commerciale en production
- Déploiement multi-sites (plusieurs restaurants)
- Évolution continue du produit
- Maintenance long terme

---

## 7. Analogie finale: La construction d'un immeuble

**SQLAlchemy** = Les outils pour **vivre** dans l'immeuble au quotidien
- Ouvrir les portes (lire des données)
- Installer des meubles (ajouter des données)
- Réarranger (modifier des données)
- Jeter des meubles (supprimer des données)

**Alembic** = Les plans et permis pour **faire évoluer** l'immeuble
- Version 1: Construire l'immeuble initial
- Version 2: Ajouter un étage
- Version 3: Installer un ascenseur
- Chaque modification est documentée et réversible

**Sans Alembic**, tu pourrais démolir l'immeuble et le reconstruire à chaque changement... mais tu perds tous les habitants (données) à chaque fois! 💀

---

## Récapitulatif: Ce que tu as appris aujourd'hui

### Bases de données SQL
✅ Tables, lignes, colonnes, relations  
✅ SQL = le langage de la base de données  
✅ Limites du SQL brut dans une application Python  

### SQLAlchemy (l'ORM)
✅ ORM = Pont entre Python et SQL  
✅ Engine = Connexion à la base  
✅ Session = Conversation avec la base  
✅ Models = Définition des tables en Python  
✅ CRUD en Python orienté objet au lieu de SQL  

### Alembic (les migrations)
✅ Migration = Version du schéma de base  
✅ Fait évoluer la structure sans perdre les données  
✅ Historique complet et réversible  
✅ Synchronisation dev/prod  

### Le duo
✅ SQLAlchemy pour utiliser la base au quotidien  
✅ Alembic pour faire évoluer le schéma dans le temps  
✅ Standard de l'industrie Python  
✅ Parfait pour ton POS commercial  

---

## Prochaine étape

**Demain**, tu vas:
1. Pratiquer avec le mini-projet library_demo
2. Ajouter une deuxième table (Books) avec relation vers Authors
3. Créer ta première vraie migration avec Alembic
4. Voir comment les relations fonctionnent en pratique

**Objectif**: Maîtriser les bases avant d'intégrer la vraie base de données dans ton POS.

---

**Note**: Ce n'est pas aussi compliqué que ça en a l'air au début. Après quelques heures de pratique, SQLAlchemy et Alembic deviennent naturels, et tu ne voudras plus revenir au SQL brut! 🚀