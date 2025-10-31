# Guide de Configuration - Pilotage Business

## 1. Configuration GitHub

### 1.1 Créer un Repository

1. Aller à https://github.com/new
2. Remplir les informations :
   - **Repository name :** `pilotage-business`
   - **Description :** Dashboard CEO pour gestion d'entreprise
   - **Visibility :** Private (recommandé)
   - **Initialize with README :** Cocher

3. Cliquer "Create repository"

### 1.2 Cloner et Configurer Localement

```bash
# Cloner le repo
git clone https://github.com/YOUR_USERNAME/pilotage-business.git
cd pilotage-business

# Ajouter les fichiers du projet
# (Copier tous les fichiers du checkpoint)

# Configurer Git
git config user.name "Your Name"
git config user.email "your.email@example.com"

# Ajouter les fichiers
git add .

# Premier commit
git commit -m "Initial commit: Pilotage Business Dashboard"

# Pousser vers GitHub
git push -u origin main
```

### 1.3 .gitignore

Créer un fichier `.gitignore` à la racine :

```
# Dependencies
node_modules/
.pnp
.pnp.js

# Environment variables
.env
.env.local
.env.*.local

# Build outputs
dist/
build/
.next/
out/

# IDE
.vscode/
.idea/
*.swp
*.swo
*~

# OS
.DS_Store
Thumbs.db

# Logs
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# Misc
.cache/
.turbo/
```

### 1.4 Branches Recommandées

```bash
# Créer une branche de développement
git checkout -b develop
git push -u origin develop

# Créer une branche pour les features
git checkout -b feature/kanban-crm
# ... faire les modifications ...
git commit -m "feat: Add Kanban CRM view"
git push origin feature/kanban-crm

# Créer une Pull Request sur GitHub
# Puis merger dans develop après review
```

---

## 2. Configuration Supabase

### 2.1 Créer un Projet Supabase

1. Aller à https://supabase.com
2. Cliquer "New Project"
3. Remplir les informations :
   - **Project name :** `pilotage-business`
   - **Database password :** Générer un mot de passe fort
   - **Region :** Choisir la région la plus proche

4. Cliquer "Create new project"

### 2.2 Récupérer les Identifiants

Dans Supabase Dashboard :

1. **Settings** → **API**
   - Copier `Project URL` → `SUPABASE_URL`
   - Copier `anon public` → `SUPABASE_ANON_KEY`

2. **Settings** → **Database**
   - Copier `Connection string` → `DATABASE_URL`
   - Format : `postgresql://postgres:PASSWORD@HOST:5432/postgres`

### 2.3 Créer les Tables

1. Aller à **SQL Editor**
2. Cliquer "New Query"
3. Copier-coller le SQL de création des tables (voir PRD section 3.2)
4. Cliquer "Run"

Exemple pour la première table :

```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  openId VARCHAR(64) UNIQUE NOT NULL,
  name TEXT,
  email VARCHAR(320),
  loginMethod VARCHAR(64),
  role VARCHAR(50) DEFAULT 'collaborator',
  companyId INT,
  isActive BOOLEAN DEFAULT TRUE,
  createdAt TIMESTAMP DEFAULT NOW(),
  updatedAt TIMESTAMP DEFAULT NOW(),
  lastSignedIn TIMESTAMP DEFAULT NOW()
);
```

### 2.4 Activer RLS (Row Level Security)

Pour chaque table sensible :

1. Aller à **Authentication** → **Policies**
2. Sélectionner la table
3. Cliquer "New Policy"
4. Créer une policy :

```sql
-- Policy de lecture pour tous les utilisateurs authentifiés
CREATE POLICY "Users can view companies" ON companies
  FOR SELECT USING (auth.uid() IS NOT NULL);

-- Policy de création pour admins
CREATE POLICY "Admins can create companies" ON companies
  FOR INSERT WITH CHECK (
    EXISTS (
      SELECT 1 FROM users 
      WHERE users.id = auth.uid() AND users.role = 'admin'
    )
  );
```

---

## 3. Configuration Vercel

### 3.1 Déployer sur Vercel

1. Aller à https://vercel.com
2. Cliquer "New Project"
3. Sélectionner le repo GitHub `pilotage-business`
4. Cliquer "Import"

### 3.2 Configurer les Variables d'Environnement

Dans Vercel Dashboard → Settings → Environment Variables

Ajouter toutes les variables :

```
DATABASE_URL=postgresql://postgres:...
SUPABASE_URL=https://nfzjxrllfihkuuydishh.supabase.co
SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
VITE_APP_ID=your_app_id
OAUTH_SERVER_URL=https://api.manus.im
VITE_OAUTH_PORTAL_URL=https://login.manus.im
JWT_SECRET=your_jwt_secret_here
VITE_APP_TITLE=Pilotage Business
VITE_APP_LOGO=https://your-logo-url.png
OWNER_OPEN_ID=your_owner_id
OWNER_NAME=Your Name
BUILT_IN_FORGE_API_URL=https://api.manus.im
BUILT_IN_FORGE_API_KEY=your_api_key
```

### 3.3 Déployer

1. Cliquer "Deploy"
2. Vercel va construire et déployer l'application
3. Une fois terminé, vous recevrez une URL publique

### 3.4 Configurer un Domain Personnalisé

1. Dans Vercel → Settings → Domains
2. Ajouter votre domaine
3. Suivre les instructions pour configurer les DNS
4. SSL sera activé automatiquement

---

## 4. Configuration Locale pour Développement

### 4.1 Installation

```bash
# Cloner le repo
git clone https://github.com/YOUR_USERNAME/pilotage-business.git
cd pilotage-business

# Installer les dépendances
pnpm install
# ou npm install
```

### 4.2 Variables d'Environnement Locales

Créer `.env.local` :

```env
# Database
DATABASE_URL=postgresql://postgres:PASSWORD@localhost:5432/postgres

# Supabase
SUPABASE_URL=https://nfzjxrllfihkuuydishh.supabase.co
SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# OAuth
VITE_APP_ID=dev_app_id
OAUTH_SERVER_URL=https://api.manus.im
VITE_OAUTH_PORTAL_URL=https://login.manus.im
JWT_SECRET=dev_secret_key_change_in_production

# App
VITE_APP_TITLE=Pilotage Business
VITE_APP_LOGO=https://your-logo-url.png

# Owner
OWNER_OPEN_ID=your_id
OWNER_NAME=Your Name

# APIs
BUILT_IN_FORGE_API_URL=https://api.manus.im
BUILT_IN_FORGE_API_KEY=your_key
```

### 4.3 Démarrer le Serveur

```bash
# Démarrer en développement
pnpm dev

# L'app sera disponible à http://localhost:5173
```

### 4.4 Commandes Utiles

```bash
# Vérifier les types TypeScript
pnpm tsc --noEmit

# Formater le code
pnpm format

# Linter
pnpm lint

# Build production
pnpm build

# Prévisualiser la build
pnpm preview

# Synchroniser la base de données
pnpm db:push

# Générer les migrations
pnpm db:generate
```

---

## 5. Workflow de Développement Recommandé

### 5.1 Ajouter une Nouvelle Fonctionnalité

```bash
# 1. Créer une branche feature
git checkout -b feature/ma-fonctionnalite

# 2. Modifier le schéma Drizzle si nécessaire
# Éditer drizzle/schema.ts

# 3. Générer la migration
pnpm db:generate

# 4. Appliquer la migration
pnpm db:push

# 5. Ajouter la procédure tRPC
# Éditer server/routers.ts

# 6. Ajouter le composant React
# Créer client/src/pages/MaFonctionnalite.tsx

# 7. Tester localement
pnpm dev

# 8. Commit et push
git add .
git commit -m "feat: Add ma-fonctionnalite"
git push origin feature/ma-fonctionnalite

# 9. Créer une Pull Request sur GitHub
# Attendre la review et merger
```

### 5.2 Déployer en Production

```bash
# 1. Merger la branche dans main
git checkout main
git merge develop

# 2. Créer un tag de version
git tag -a v1.0.0 -m "Release version 1.0.0"
git push origin main --tags

# 3. Vercel déploiera automatiquement
# Vérifier sur https://vercel.com/dashboard

# 4. Tester en production
# Visiter votre domaine
```

---

## 6. Monitoring et Maintenance

### 6.1 Vercel Monitoring

1. Aller à Vercel Dashboard
2. Cliquer sur votre projet
3. Voir les métriques :
   - **Deployments** : Historique des déploiements
   - **Analytics** : Trafic et performance
   - **Logs** : Logs du serveur

### 6.2 Supabase Monitoring

1. Aller à Supabase Dashboard
2. Voir les métriques :
   - **Database** : Utilisation de la base de données
   - **Auth** → **Users** : Nombre d'utilisateurs
   - **Storage** : Utilisation du stockage

### 6.3 Sauvegardes Supabase

1. Aller à **Settings** → **Backups**
2. Activer les sauvegardes automatiques
3. Configurer la fréquence (quotidienne recommandée)

---

## 7. Dépannage Courant

### Erreur : "DATABASE_URL not found"

```bash
# Vérifier que .env.local existe
ls -la .env.local

# Vérifier le contenu
cat .env.local | grep DATABASE_URL

# Redémarrer le serveur
pnpm dev
```

### Erreur : "Cannot find module '@trpc/client'"

```bash
# Réinstaller les dépendances
rm -rf node_modules pnpm-lock.yaml
pnpm install
```

### Erreur : "CORS error"

```bash
# Vérifier que OAUTH_SERVER_URL est correct
# Vérifier que VITE_OAUTH_PORTAL_URL est correct
# Redémarrer le serveur
```

### Erreur : "RLS policy violation"

```bash
# Vérifier que l'utilisateur est authentifié
# Vérifier les policies dans Supabase
# Assurez-vous que la policy autorise l'opération
```

---

## 8. Sécurité

### 8.1 Secrets à Protéger

**JAMAIS** commiter ces fichiers :
- `.env.local`
- `.env.production`
- Clés API privées

Utiliser Vercel Secrets ou GitHub Secrets pour les variables sensibles.

### 8.2 Authentification

- Toutes les procédures sensibles utilisent `protectedProcedure`
- Les rôles sont vérifiés côté serveur
- Les cookies de session sont signés avec `JWT_SECRET`

### 8.3 Base de Données

- RLS est activé sur les tables sensibles
- Les policies limitent l'accès par rôle
- Les mots de passe sont hashés (si applicable)

---

## 9. Checklist Finale

- [ ] Repository GitHub créé et configuré
- [ ] `.gitignore` ajouté
- [ ] Projet Supabase créé
- [ ] Tables créées dans Supabase
- [ ] RLS activé sur les tables sensibles
- [ ] Variables d'env configurées localement
- [ ] `pnpm dev` fonctionne sans erreurs
- [ ] Vercel connecté au repository
- [ ] Variables d'env configurées dans Vercel
- [ ] Déploiement Vercel réussi
- [ ] Domain personnalisé configuré (optionnel)
- [ ] Tests en production réussis

---

**Prêt à développer ! 🚀**

Pour toute question, consulter le PRD complet : `PRD_PILOTAGE_BUSINESS.md`
