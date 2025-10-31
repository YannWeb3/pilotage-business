# Pilotage Business - Dashboard CEO

Dashboard de gestion d'entreprise permettant de centraliser et piloter tous les KPIs, deals, projets, finances et objectifs en un seul endroit.

## 🎯 Fonctionnalités Principales

- **Dashboard CEO** : Vue d'ensemble avec KPIs, actions prioritaires, projection trésorerie
- **Sales CRM** : Vue Kanban avec pipeline commercial (6 colonnes)
- **Contacts & Entreprises** : Galerie de clients et partenaires
- **Ops & Projets** : Gestion opérationnelle avec suivi de progression
- **Finance & P&L** : Synthèse financière et transactions
- **Objectifs & OKR** : Suivi des objectifs et KPIs
- **Admin** : Panel administrateur

## 🛠️ Stack Technique

- **Frontend** : React 19 + TypeScript + Tailwind CSS 4
- **Backend** : Next.js 14 + Express 4 + tRPC 11
- **Base de données** : Supabase (PostgreSQL)
- **Authentification** : OAuth Manus
- **Déploiement** : Vercel (recommandé) ou Coolify

## 📚 Documentation

### Pour les Développeurs

1. **[PRD_PILOTAGE_BUSINESS.md](./PRD_PILOTAGE_BUSINESS.md)** - Spécification complète
   - Architecture et structure du projet
   - Schéma Supabase avec 13 tables
   - Guide d'installation pas à pas
   - Modules fonctionnels
   - Procédures tRPC

2. **[SETUP_GUIDE.md](./SETUP_GUIDE.md)** - Instructions pratiques
   - Configuration GitHub
   - Configuration Supabase
   - Configuration Vercel
   - Installation locale
   - Workflow de développement

3. **[DATABASE_RELATIONS.md](./DATABASE_RELATIONS.md)** - Relations Supabase
   - Diagramme ER complet
   - Détail de chaque relation
   - Exemples de requêtes SQL
   - Bonnes pratiques

## 🚀 Démarrage Rapide

### Prérequis

- Node.js 18+
- pnpm ou npm
- Compte Supabase
- Compte Vercel (optionnel)

### Installation Locale

```bash
# Cloner le repository
git clone https://github.com/YannWeb3/pilotage-business.git
cd pilotage-business

# Installer les dépendances
pnpm install

# Configurer les variables d'environnement
cp .env.example .env.local
# Éditer .env.local avec vos identifiants Supabase

# Démarrer le serveur de développement
pnpm dev
```

L'application sera disponible à `http://localhost:5173`

## 📊 Schéma Supabase

Le projet utilise 13 tables PostgreSQL :

```
USERS → COMPANIES → CONTACTS
                  ↓
                DEALS
                  ↓
              PROJECTS → PROJECT_ASSIGNMENTS
                  ↓
        FINANCE_TRANSACTIONS → FINANCE_CATEGORIES
                  ↓
            FINANCE_FORECAST
                  
OBJECTIVES
PRIORITY_ACTIONS
KPI_CACHE
WORKLOAD_USERS
```

Voir [DATABASE_RELATIONS.md](./DATABASE_RELATIONS.md) pour les détails complets.

## 🔑 Variables d'Environnement

Créer un fichier `.env.local` :

```env
# Database
DATABASE_URL=postgresql://postgres:PASSWORD@host:5432/postgres

# Supabase
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=your_anon_key

# OAuth
VITE_APP_ID=your_app_id
OAUTH_SERVER_URL=https://api.manus.im
VITE_OAUTH_PORTAL_URL=https://login.manus.im
JWT_SECRET=your_jwt_secret

# App Config
VITE_APP_TITLE=Pilotage Business
VITE_APP_LOGO=https://your-logo-url.png

# Owner Info
OWNER_OPEN_ID=your_owner_id
OWNER_NAME=Your Name

# APIs
BUILT_IN_FORGE_API_URL=https://api.manus.im
BUILT_IN_FORGE_API_KEY=your_api_key
```

## 📦 Commandes Utiles

```bash
# Développement
pnpm dev                    # Démarrer le serveur
pnpm tsc --noEmit          # Vérifier les types TypeScript
pnpm format                # Formater le code
pnpm lint                  # Linter

# Base de données
pnpm db:push               # Synchroniser le schéma
pnpm db:generate           # Générer les migrations

# Production
pnpm build                 # Build production
pnpm preview               # Prévisualiser la build
```

## 🌐 Déploiement

### Vercel (Recommandé)

1. Connecter le repo GitHub à Vercel
2. Configurer les variables d'environnement
3. Déployer automatiquement

Voir [SETUP_GUIDE.md - Configuration Vercel](./SETUP_GUIDE.md#3-configuration-vercel)

### Coolify (Auto-hébergé)

1. Installer Coolify sur votre serveur
2. Créer une nouvelle application
3. Configurer les variables d'environnement

Voir [SETUP_GUIDE.md - Déploiement Coolify](./PRD_PILOTAGE_BUSINESS.md#62-déploiement-sur-coolify-auto-hébergé)

## 🏗️ Structure du Projet

```
pilotage-business/
├── client/                 # Frontend React
│   ├── src/
│   │   ├── pages/         # Pages principales
│   │   ├── components/    # Composants réutilisables
│   │   ├── lib/           # Utilitaires (tRPC, Supabase)
│   │   └── contexts/      # React contexts
│   └── index.html
├── server/                # Backend Node.js
│   ├── routers.ts         # Procédures tRPC
│   ├── db.ts              # Helpers de base de données
│   └── _core/             # Framework plumbing
├── drizzle/               # ORM Drizzle
│   └── schema.ts          # Schéma des tables
├── PRD_PILOTAGE_BUSINESS.md
├── SETUP_GUIDE.md
├── DATABASE_RELATIONS.md
└── README.md
```

## 🔐 Sécurité

- Authentification OAuth Manus
- Row Level Security (RLS) activé sur Supabase
- Procédures tRPC protégées par rôle
- Variables d'environnement sécurisées

## 📝 Modules Fonctionnels

### Dashboard CEO
- KPIs principaux (CA, Projets, Taux Conversion, MRR)
- Actions prioritaires
- Projection trésorerie (6 mois)
- Runway (mois de trésorerie)

### Sales CRM
- Vue Kanban avec 6 colonnes (Lead → Gagné/Perdu)
- Drag & drop entre colonnes
- Création de deals
- Affichage montant + date

### Contacts & Entreprises
- Galerie de cartes
- Secteur, CA cumulé, pays
- Création d'entreprise
- Recherche par nom/secteur

### Ops & Projets
- Statuts (En cours, À risque, Terminés)
- Barre de progression
- Dates début/fin
- Création de projet

### Finance & P&L
- Synthèse (Revenus, Dépenses, Résultat Net)
- Tableau transactions
- Catégories hiérarchiques
- Création de transaction

### Objectifs & OKR
- Titre + description
- Période (Q1 2024, 2024, etc.)
- Barre de progression
- Responsable

## 🤝 Contribution

Les contributions sont bienvenues ! Veuillez :

1. Créer une branche feature (`git checkout -b feature/ma-feature`)
2. Commiter vos changements (`git commit -m 'feat: Add ma-feature'`)
3. Pousser vers la branche (`git push origin feature/ma-feature`)
4. Ouvrir une Pull Request

## 📞 Support

Pour toute question ou problème :

1. Consulter la [documentation complète](./PRD_PILOTAGE_BUSINESS.md)
2. Vérifier les [relations Supabase](./DATABASE_RELATIONS.md)
3. Suivre le [guide de configuration](./SETUP_GUIDE.md)

## 📄 Licence

Ce projet est sous licence MIT.

## 👨‍💻 Auteur

Créé par **Manus AI** - Dashboard CEO pour gestion d'entreprise

---

**Prêt à démarrer ? Consultez le [SETUP_GUIDE.md](./SETUP_GUIDE.md) pour les instructions d'installation ! 🚀**
