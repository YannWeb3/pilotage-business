# PRD - Pilotage Business Dashboard CEO

## 1. Vue d'ensemble

**Pilotage Business** est une application web de gestion d'entreprise type dashboard CEO permettant de centraliser et piloter tous les KPIs, deals, projets, finances et objectifs en un seul endroit.

**Stack technique :**
- Frontend : React 19 + TypeScript + Tailwind CSS 4
- Backend : Next.js 14 + Express 4 + tRPC 11
- Base de données : Supabase (PostgreSQL)
- Authentification : OAuth Manus
- Déploiement : Vercel (recommandé) ou Coolify (auto-hébergé)

**Lien du checkpoint stable :** `manus-webdev://90c803cc`

---

## 2. Architecture et Structure du Projet

### 2.1 Structure des Dossiers

```
pilotage-business/
├── client/                          # Frontend React
│   ├── public/                      # Assets statiques
│   ├── src/
│   │   ├── components/              # Composants réutilisables
│   │   │   ├── DashboardLayout.tsx  # Layout sidebar principal
│   │   │   ├── Modal.tsx            # Composant modal réutilisable
│   │   │   └── ui/                  # Composants shadcn/ui
│   │   ├── contexts/                # React contexts
│   │   │   └── ThemeContext.tsx     # Gestion du thème sombre
│   │   ├── hooks/                   # Custom hooks
│   │   │   └── useAuth.ts           # Hook authentification
│   │   ├── lib/
│   │   │   ├── trpc.ts              # Client tRPC
│   │   │   └── supabase.ts          # Client Supabase (optionnel)
│   │   ├── pages/                   # Pages principales
│   │   │   ├── Home.tsx             # Page d'accueil/login
│   │   │   ├── Dashboard.tsx        # Dashboard CEO
│   │   │   ├── CRM.tsx              # Vue Kanban CRM
│   │   │   ├── Companies.tsx        # Galerie Contacts & Entreprises
│   │   │   ├── Projects.tsx         # Gestion des projets
│   │   │   ├── Finance.tsx          # Gestion financière
│   │   │   ├── Objectives.tsx       # Objectifs & OKR
│   │   │   └── Admin.tsx            # Panel admin
│   │   ├── App.tsx                  # Routes principales
│   │   ├── main.tsx                 # Point d'entrée
│   │   └── index.css                # Styles globaux + thème
│   ├── index.html                   # HTML template
│   └── vite.config.ts               # Config Vite
│
├── server/                          # Backend Node.js/Express
│   ├── _core/                       # Framework plumbing
│   │   ├── context.ts               # Contexte tRPC (user, req, res)
│   │   ├── trpc.ts                  # Procédures tRPC (public, protected)
│   │   ├── cookies.ts               # Gestion des cookies de session
│   │   ├── env.ts                   # Variables d'environnement
│   │   ├── llm.ts                   # Intégration LLM
│   │   ├── voiceTranscription.ts    # Transcription audio
│   │   ├── imageGeneration.ts       # Génération d'images
│   │   └── notification.ts          # Notifications propriétaire
│   ├── routers.ts                   # Procédures tRPC (mutations & queries)
│   ├── db.ts                        # Helpers de base de données
│   ├── storage.ts                   # Helpers S3
│   └── index.ts                     # Point d'entrée serveur
│
├── drizzle/                         # ORM Drizzle
│   ├── schema.ts                    # Définition des tables
│   └── migrations/                  # Migrations SQL
│
├── storage/                         # Utilitaires S3
├── shared/                          # Code partagé client/serveur
│   └── const.ts                     # Constantes globales
│
├── .env.local                       # Variables d'env locales
├── .env.example                     # Template variables d'env
├── package.json                     # Dépendances
├── tsconfig.json                    # Config TypeScript
├── drizzle.config.ts                # Config Drizzle ORM
└── README.md                        # Documentation
```

### 2.2 Flux de Données

```
Client (React)
    ↓
tRPC Client (client/src/lib/trpc.ts)
    ↓
tRPC Server (server/routers.ts)
    ↓
Database Helpers (server/db.ts)
    ↓
Drizzle ORM (drizzle/schema.ts)
    ↓
Supabase PostgreSQL
```

**Exemple de mutation :**
```typescript
// Client
const createMutation = trpc.companies.create.useMutation();
createMutation.mutate({ name: "Acme Corp", sector: "Tech" });

// Server (routers.ts)
companies: router({
  create: protectedProcedure
    .input(z.object({ name: z.string(), sector: z.string().optional() }))
    .mutation(async ({ input }) => {
      return await createCompany(input);
    }),
})

// Database (db.ts)
export async function createCompany(data) {
  const db = await getDb();
  return await db.insert(companies).values(data);
}
```

---

## 3. Schéma de Base de Données Supabase

### 3.1 Diagramme des Relations

```
┌─────────────────────────────────────────────────────────────────┐
│                        USERS (Authentification)                  │
│  id | openId | name | email | role | companyId | isActive       │
└─────────────────────────────────────────────────────────────────┘
              │                    │
              ├────────────────────┼─────────────────────────┐
              │                    │                         │
              ↓                    ↓                         ↓
    ┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
    │   COMPANIES      │  │   CONTACTS       │  │  PROJECT_ASSIGN  │
    │  (Clients)       │  │  (Personnes)     │  │  (Équipe projets)│
    │                  │  │                  │  │                  │
    │ id | name        │  │ id | companyId   │  │ id | projectId   │
    │ sector | country │  │ firstName | email│  │ userId | role    │
    │ cumulativeRev    │  │ phone | position │  │                  │
    │ lastContactDate  │  │ isMainContact    │  │                  │
    └──────────────────┘  └──────────────────┘  └──────────────────┘
              │                    │
              │                    │
              ├────────────────────┘
              │
              ↓
    ┌──────────────────────────────────────────┐
    │   DEALS (Pipeline Commercial)             │
    │                                           │
    │ id | name | companyId | contactId         │
    │ amount | status | expectedCloseDate       │
    │ responsibleUserId | createdAt | updatedAt │
    │                                           │
    │ Status: lead | negotiation | proposition  │
    │         | closing | won | lost            │
    └──────────────────────────────────────────┘
              │
              ↓
    ┌──────────────────────────────────────────┐
    │   PROJECTS (Ops & Projets)                │
    │                                           │
    │ id | name | companyId | description       │
    │ status | progress | startDate | endDate   │
    │ estimatedDelay | createdAt | updatedAt    │
    │                                           │
    │ Status: planning | in_progress | at_risk  │
    │         | completed | cancelled           │
    └──────────────────────────────────────────┘
              │
              ↓
    ┌──────────────────────────────────────────┐
    │   FINANCE TRANSACTIONS (Revenus/Dépenses) │
    │                                           │
    │ id | categoryId | companyId | amount      │
    │ type | description | date | createdAt     │
    │                                           │
    │ Type: revenue | expense                   │
    └──────────────────────────────────────────┘
              │
              ↓
    ┌──────────────────────────────────────────┐
    │   FINANCE CATEGORIES (Structure P&L)      │
    │                                           │
    │ id | name | type | parentId | description │
    │ createdAt                                 │
    │                                           │
    │ Type: revenue | expense                   │
    │ parentId: Pour hiérarchie (ex: Salaires)  │
    └──────────────────────────────────────────┘

    ┌──────────────────────────────────────────┐
    │   FINANCE FORECAST (Projections 6 mois)   │
    │                                           │
    │ id | month | projectedRevenue             │
    │ projectedExpenses | projectedCashflow     │
    │ createdAt | updatedAt                     │
    └──────────────────────────────────────────┘

    ┌──────────────────────────────────────────┐
    │   OBJECTIVES (OKR / KPIs)                 │
    │                                           │
    │ id | title | description | period         │
    │ progress | status | responsibleUserId     │
    │ createdAt | updatedAt                     │
    │                                           │
    │ Status: in_progress | completed | paused  │
    │ Period: Q1 2024, 2024, etc.               │
    └──────────────────────────────────────────┘

    ┌──────────────────────────────────────────┐
    │   PRIORITY ACTIONS (Actions Prioritaires)  │
    │                                           │
    │ id | title | description | priority       │
    │ status | dueDate | assignedUserId         │
    │ createdAt | updatedAt                     │
    │                                           │
    │ Priority: low | medium | high | critical  │
    │ Status: open | in_progress | completed    │
    └──────────────────────────────────────────┘

    ┌──────────────────────────────────────────┐
    │   KPI CACHE (Cache pour Dashboard)        │
    │                                           │
    │ id | metric | value | lastUpdated         │
    │                                           │
    │ Metrics: totalRevenue, activeProjects,    │
    │          conversionRate, etc.             │
    └──────────────────────────────────────────┘

    ┌──────────────────────────────────────────┐
    │   WORKLOAD USERS (Surcharge d'équipe)     │
    │                                           │
    │ id | userId | projectCount | isOverloaded │
    │ lastUpdated                               │
    └──────────────────────────────────────────┘
```

### 3.2 Définition Détaillée des Tables

#### **USERS** (Authentification & Rôles)
```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  openId VARCHAR(64) UNIQUE NOT NULL,           -- ID OAuth Manus
  name TEXT,
  email VARCHAR(320),
  loginMethod VARCHAR(64),
  role ENUM('admin', 'collaborator', 'client', 'guest') DEFAULT 'collaborator',
  companyId INT,                                -- Entreprise de l'utilisateur
  isActive BOOLEAN DEFAULT TRUE,
  createdAt TIMESTAMP DEFAULT NOW(),
  updatedAt TIMESTAMP DEFAULT NOW(),
  lastSignedIn TIMESTAMP DEFAULT NOW()
);
```

**Relations :**
- `companyId` → `companies.id` (FK)
- Utilisé dans `deals.responsibleUserId`, `projectAssignments.userId`, `objectives.responsibleUserId`

---

#### **COMPANIES** (Clients & Partenaires)
```sql
CREATE TABLE companies (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  sector VARCHAR(100),                         -- Technologie, Consulting, etc.
  size ENUM('startup', 'pme', 'etpi', 'enterprise'),
  country VARCHAR(100),
  logo TEXT,                                   -- URL S3
  cumulativeRevenue DECIMAL(12, 2) DEFAULT 0,  -- CA total avec cette entreprise
  lastContactDate TIMESTAMP,
  createdAt TIMESTAMP DEFAULT NOW(),
  updatedAt TIMESTAMP DEFAULT NOW()
);
```

**Relations :**
- Parent de `contacts`, `deals`, `projects`
- Référencée par `users.companyId`

---

#### **CONTACTS** (Personnes dans les Entreprises)
```sql
CREATE TABLE contacts (
  id SERIAL PRIMARY KEY,
  companyId INT NOT NULL,                      -- FK → companies.id
  firstName VARCHAR(100) NOT NULL,
  lastName VARCHAR(100) NOT NULL,
  email VARCHAR(320) NOT NULL,
  phone VARCHAR(20),
  position VARCHAR(100),                       -- Titre du poste
  isMainContact BOOLEAN DEFAULT FALSE,         -- Contact principal
  createdAt TIMESTAMP DEFAULT NOW(),
  updatedAt TIMESTAMP DEFAULT NOW()
);
```

**Relations :**
- `companyId` → `companies.id` (FK)
- Référencée par `deals.contactId`

---

#### **DEALS** (Pipeline Commercial - Kanban)
```sql
CREATE TABLE deals (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL,                  -- Nom du deal
  companyId INT NOT NULL,                      -- FK → companies.id
  contactId INT,                               -- FK → contacts.id (optionnel)
  amount DECIMAL(12, 2) NOT NULL,              -- Montant en euros
  status ENUM('lead', 'negotiation', 'proposition', 'closing', 'won', 'lost'),
  expectedCloseDate DATE,
  responsibleUserId INT,                       -- FK → users.id
  createdAt TIMESTAMP DEFAULT NOW(),
  updatedAt TIMESTAMP DEFAULT NOW()
);
```

**Colonnes Kanban :**
- Lead → Négociation → Proposition → Closing → Gagné/Perdu

**Relations :**
- `companyId` → `companies.id` (FK)
- `contactId` → `contacts.id` (FK, optionnel)
- `responsibleUserId` → `users.id` (FK)

---

#### **PROJECTS** (Gestion Opérationnelle)
```sql
CREATE TABLE projects (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  companyId INT NOT NULL,                      -- FK → companies.id
  description TEXT,
  status ENUM('planning', 'in_progress', 'at_risk', 'completed', 'cancelled'),
  progress INT DEFAULT 0,                      -- 0-100%
  startDate DATE,
  endDate DATE,
  estimatedDelay INT,                          -- Jours de retard estimé
  createdAt TIMESTAMP DEFAULT NOW(),
  updatedAt TIMESTAMP DEFAULT NOW()
);
```

**Relations :**
- `companyId` → `companies.id` (FK)
- Parent de `projectAssignments`

---

#### **PROJECT_ASSIGNMENTS** (Équipe sur les Projets)
```sql
CREATE TABLE projectAssignments (
  id SERIAL PRIMARY KEY,
  projectId INT NOT NULL,                      -- FK → projects.id
  userId INT NOT NULL,                         -- FK → users.id
  role VARCHAR(100),                           -- Chef de projet, Développeur, etc.
  assignedAt TIMESTAMP DEFAULT NOW()
);
```

**Relations :**
- `projectId` → `projects.id` (FK)
- `userId` → `users.id` (FK)

---

#### **FINANCE_TRANSACTIONS** (Revenus & Dépenses)
```sql
CREATE TABLE financeTransactions (
  id SERIAL PRIMARY KEY,
  categoryId INT NOT NULL,                     -- FK → financeCategories.id
  companyId INT,                               -- FK → companies.id (optionnel)
  amount DECIMAL(12, 2) NOT NULL,
  type ENUM('revenue', 'expense') NOT NULL,
  description TEXT,
  date DATE NOT NULL,
  createdAt TIMESTAMP DEFAULT NOW()
);
```

**Relations :**
- `categoryId` → `financeCategories.id` (FK)
- `companyId` → `companies.id` (FK, optionnel)

---

#### **FINANCE_CATEGORIES** (Structure P&L)
```sql
CREATE TABLE financeCategories (
  id SERIAL PRIMARY KEY,
  name VARCHAR(255) NOT NULL,                  -- Salaires, Marketing, etc.
  type ENUM('revenue', 'expense') NOT NULL,
  parentId INT,                                -- FK → financeCategories.id (hiérarchie)
  description TEXT,
  createdAt TIMESTAMP DEFAULT NOW()
);
```

**Exemple d'hiérarchie :**
```
Dépenses
├── Salaires
│   ├── Salaires CDI
│   └── Salaires Freelance
├── Marketing
│   ├── Publicité
│   └── Événements
└── Infrastructure
    ├── Serveurs
    └── Logiciels
```

---

#### **FINANCE_FORECAST** (Projections 6 Mois)
```sql
CREATE TABLE financeForecast (
  id SERIAL PRIMARY KEY,
  month DATE NOT NULL,                         -- Premier jour du mois
  projectedRevenue DECIMAL(12, 2) NOT NULL,
  projectedExpenses DECIMAL(12, 2) NOT NULL,
  projectedCashflow DECIMAL(12, 2) NOT NULL,   -- Revenue - Expenses
  createdAt TIMESTAMP DEFAULT NOW(),
  updatedAt TIMESTAMP DEFAULT NOW()
);
```

---

#### **OBJECTIVES** (OKR / KPIs)
```sql
CREATE TABLE objectives (
  id SERIAL PRIMARY KEY,
  title VARCHAR(255) NOT NULL,                 -- Ex: "Augmenter CA de 30%"
  description TEXT,
  period VARCHAR(50) NOT NULL,                 -- "Q1 2024", "2024", etc.
  progress INT DEFAULT 0,                      -- 0-100%
  status ENUM('in_progress', 'completed', 'paused'),
  responsibleUserId INT,                       -- FK → users.id
  createdAt TIMESTAMP DEFAULT NOW(),
  updatedAt TIMESTAMP DEFAULT NOW()
);
```

**Relations :**
- `responsibleUserId` → `users.id` (FK)

---

#### **PRIORITY_ACTIONS** (Actions Prioritaires Dashboard)
```sql
CREATE TABLE priorityActions (
  id SERIAL PRIMARY KEY,
  title VARCHAR(255) NOT NULL,
  description TEXT,
  priority ENUM('low', 'medium', 'high', 'critical'),
  status ENUM('open', 'in_progress', 'completed'),
  dueDate DATE,
  assignedUserId INT,                          -- FK → users.id
  createdAt TIMESTAMP DEFAULT NOW(),
  updatedAt TIMESTAMP DEFAULT NOW()
);
```

---

#### **KPI_CACHE** (Cache Dashboard)
```sql
CREATE TABLE kpiCache (
  id SERIAL PRIMARY KEY,
  metric VARCHAR(100) NOT NULL UNIQUE,         -- "totalRevenue", "activeProjects", etc.
  value TEXT,                                  -- JSON stringifié
  lastUpdated TIMESTAMP DEFAULT NOW()
);
```

**Exemples de métriques :**
```json
{
  "totalRevenue": 245000,
  "activeProjects": 18,
  "conversionRate": 68,
  "mrr": 45000
}
```

---

#### **WORKLOAD_USERS** (Surcharge d'Équipe)
```sql
CREATE TABLE workloadUsers (
  id SERIAL PRIMARY KEY,
  userId INT NOT NULL,                         -- FK → users.id
  projectCount INT DEFAULT 0,
  isOverloaded BOOLEAN DEFAULT FALSE,
  lastUpdated TIMESTAMP DEFAULT NOW()
);
```

---

### 3.3 Clés Étrangères (Foreign Keys)

```sql
ALTER TABLE contacts ADD CONSTRAINT fk_contacts_company 
  FOREIGN KEY (companyId) REFERENCES companies(id) ON DELETE CASCADE;

ALTER TABLE deals ADD CONSTRAINT fk_deals_company 
  FOREIGN KEY (companyId) REFERENCES companies(id) ON DELETE CASCADE;

ALTER TABLE deals ADD CONSTRAINT fk_deals_contact 
  FOREIGN KEY (contactId) REFERENCES contacts(id) ON DELETE SET NULL;

ALTER TABLE deals ADD CONSTRAINT fk_deals_user 
  FOREIGN KEY (responsibleUserId) REFERENCES users(id) ON DELETE SET NULL;

ALTER TABLE projects ADD CONSTRAINT fk_projects_company 
  FOREIGN KEY (companyId) REFERENCES companies(id) ON DELETE CASCADE;

ALTER TABLE projectAssignments ADD CONSTRAINT fk_assignments_project 
  FOREIGN KEY (projectId) REFERENCES projects(id) ON DELETE CASCADE;

ALTER TABLE projectAssignments ADD CONSTRAINT fk_assignments_user 
  FOREIGN KEY (userId) REFERENCES users(id) ON DELETE CASCADE;

ALTER TABLE financeTransactions ADD CONSTRAINT fk_transactions_category 
  FOREIGN KEY (categoryId) REFERENCES financeCategories(id) ON DELETE RESTRICT;

ALTER TABLE financeTransactions ADD CONSTRAINT fk_transactions_company 
  FOREIGN KEY (companyId) REFERENCES companies(id) ON DELETE SET NULL;

ALTER TABLE financeCategories ADD CONSTRAINT fk_categories_parent 
  FOREIGN KEY (parentId) REFERENCES financeCategories(id) ON DELETE CASCADE;

ALTER TABLE objectives ADD CONSTRAINT fk_objectives_user 
  FOREIGN KEY (responsibleUserId) REFERENCES users(id) ON DELETE SET NULL;

ALTER TABLE priorityActions ADD CONSTRAINT fk_actions_user 
  FOREIGN KEY (assignedUserId) REFERENCES users(id) ON DELETE SET NULL;

ALTER TABLE workloadUsers ADD CONSTRAINT fk_workload_user 
  FOREIGN KEY (userId) REFERENCES users(id) ON DELETE CASCADE;

ALTER TABLE users ADD CONSTRAINT fk_users_company 
  FOREIGN KEY (companyId) REFERENCES companies(id) ON DELETE SET NULL;
```

---

## 4. Configuration Supabase

### 4.1 Connexion à Supabase

**Identifiants :**
```
URL: https://nfzjxrllfihkuuydishh.supabase.co
ANON_KEY: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Im5memp4cmxsZmloa3V1eWRpc2hoIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NjEyMTEwNTQsImV4cCI6MjA3Njc4NzA1NH0.enFiTKviJL5avNaL8LCq-D6zgEhBhg-A-38VaRhgmG8
```

### 4.2 Créer les Tables dans Supabase

1. Aller à **SQL Editor** dans Supabase
2. Créer une nouvelle query
3. Copier-coller le SQL des tables (voir section 3.2)
4. Exécuter

### 4.3 Activer RLS (Row Level Security)

Pour chaque table, activer RLS et créer des policies :

```sql
-- Exemple pour companies
ALTER TABLE companies ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view companies" ON companies
  FOR SELECT USING (auth.uid() IS NOT NULL);

CREATE POLICY "Admins can modify companies" ON companies
  FOR ALL USING (
    EXISTS (
      SELECT 1 FROM users 
      WHERE users.id = auth.uid() AND users.role = 'admin'
    )
  );
```

---

## 5. Guide d'Installation pour Développeur

### 5.1 Prérequis

- Node.js 18+ et npm/pnpm
- Git
- Compte Supabase
- Compte Vercel (optionnel, pour déploiement)

### 5.2 Installation Locale

#### **Étape 1 : Cloner le Repository**

```bash
git clone https://github.com/votre-org/pilotage-business.git
cd pilotage-business
```

#### **Étape 2 : Installer les Dépendances**

```bash
# Avec pnpm (recommandé)
pnpm install

# Ou avec npm
npm install
```

#### **Étape 3 : Configurer les Variables d'Environnement**

Créer un fichier `.env.local` à la racine du projet :

```env
# Supabase
DATABASE_URL=postgresql://postgres:PASSWORD@db.nfzjxrllfihkuuydishh.supabase.co:5432/postgres
SUPABASE_URL=https://nfzjxrllfihkuuydishh.supabase.co
SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# OAuth Manus
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

# Built-in APIs
BUILT_IN_FORGE_API_URL=https://api.manus.im
BUILT_IN_FORGE_API_KEY=your_api_key
```

#### **Étape 4 : Créer les Tables Supabase**

1. Aller à Supabase Dashboard
2. SQL Editor → New Query
3. Exécuter le SQL de création des tables (voir section 3.2)

#### **Étape 5 : Générer le Client Drizzle**

```bash
pnpm db:push
```

Cela va :
- Générer les types TypeScript depuis le schéma
- Créer les migrations
- Synchroniser avec Supabase

#### **Étape 6 : Démarrer le Serveur de Développement**

```bash
pnpm dev
```

L'application sera disponible à `http://localhost:5173`

### 5.3 Structure des Fichiers Clés

#### **drizzle/schema.ts** - Définition des Tables

```typescript
import { int, varchar, text, timestamp, mysqlEnum, decimal } from "drizzle-orm/mysql-core";
import { mysqlTable } from "drizzle-orm/mysql-core";

export const companies = mysqlTable("companies", {
  id: int("id").autoincrement().primaryKey(),
  name: varchar("name", { length: 255 }).notNull(),
  sector: varchar("sector", { length: 100 }),
  country: varchar("country", { length: 100 }),
  cumulativeRevenue: decimal("cumulativeRevenue", { precision: 12, scale: 2 }).default("0"),
  lastContactDate: timestamp("lastContactDate"),
  createdAt: timestamp("createdAt").defaultNow().notNull(),
  updatedAt: timestamp("updatedAt").defaultNow().onUpdateNow().notNull(),
});

export type Company = typeof companies.$inferSelect;
export type InsertCompany = typeof companies.$inferInsert;
```

#### **server/db.ts** - Helpers de Base de Données

```typescript
import { drizzle } from "drizzle-orm/mysql2";
import { companies } from "../drizzle/schema";

let _db: ReturnType<typeof drizzle> | null = null;

export async function getDb() {
  if (!_db && process.env.DATABASE_URL) {
    _db = drizzle(process.env.DATABASE_URL);
  }
  return _db;
}

export async function getCompanies() {
  const db = await getDb();
  if (!db) return [];
  return db.select().from(companies).orderBy(desc(companies.cumulativeRevenue));
}

export async function createCompany(data: {
  name: string;
  sector?: string;
  country?: string;
  cumulativeRevenue?: number;
}) {
  const db = await getDb();
  if (!db) throw new Error("Database not available");
  
  return await db.insert(companies).values({
    name: data.name,
    sector: data.sector || null,
    country: data.country || null,
    cumulativeRevenue: (data.cumulativeRevenue || 0).toString(),
    lastContactDate: new Date(),
  });
}
```

#### **server/routers.ts** - Procédures tRPC

```typescript
import { z } from "zod";
import { protectedProcedure, router } from "./_core/trpc";
import { getCompanies, createCompany } from "./db";

export const appRouter = router({
  companies: router({
    list: protectedProcedure.query(async () => {
      return await getCompanies();
    }),
    
    create: protectedProcedure
      .input(z.object({
        name: z.string(),
        sector: z.string().optional(),
        country: z.string().optional(),
        cumulativeRevenue: z.number().optional(),
      }))
      .mutation(async ({ input }) => {
        return await createCompany(input);
      }),
  }),
});
```

#### **client/src/pages/Companies.tsx** - Composant React

```typescript
import { trpc } from "@/lib/trpc";
import { useState } from "react";
import { toast } from "sonner";

export default function Companies() {
  const [isModalOpen, setIsModalOpen] = useState(false);
  const [formData, setFormData] = useState({ name: "", sector: "", country: "" });

  const { data: companies, refetch } = trpc.companies.list.useQuery();
  
  const createMutation = trpc.companies.create.useMutation({
    onSuccess: () => {
      toast.success("Entreprise créée");
      refetch();
      setIsModalOpen(false);
    },
    onError: (error) => {
      toast.error("Erreur: " + error.message);
    },
  });

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    createMutation.mutate(formData);
  };

  return (
    <div>
      <button onClick={() => setIsModalOpen(true)}>+ Nouvelle</button>
      {isModalOpen && (
        <form onSubmit={handleSubmit}>
          <input
            value={formData.name}
            onChange={(e) => setFormData({ ...formData, name: e.target.value })}
          />
          <button type="submit">Créer</button>
        </form>
      )}
      {companies?.map((c) => <div key={c.id}>{c.name}</div>)}
    </div>
  );
}
```

---

## 6. Déploiement

### 6.1 Déploiement sur Vercel

1. **Connecter le repo GitHub à Vercel**
   - Aller à vercel.com
   - Cliquer "New Project"
   - Sélectionner le repo GitHub

2. **Configurer les variables d'environnement**
   - Dans Vercel Settings → Environment Variables
   - Ajouter toutes les variables du `.env.local`

3. **Déployer**
   - Vercel déploiera automatiquement à chaque push sur `main`

### 6.2 Déploiement sur Coolify (Auto-hébergé)

1. **Installer Coolify sur votre serveur**
   ```bash
   curl -fsSL https://get.coollabs.io/coolify/install.sh | bash
   ```

2. **Créer une nouvelle application**
   - Source: Git repository
   - Sélectionner le repo GitHub

3. **Configurer les variables d'environnement**
   - Dans Coolify UI → Environment Variables

4. **Déployer**
   - Coolify construira et déploiera l'application

---

## 7. Modules Fonctionnels

### 7.1 Dashboard CEO

**Fichier :** `client/src/pages/Dashboard.tsx`

**Fonctionnalités :**
- KPIs principaux (CA, Projets, Taux Conversion, MRR)
- Actions prioritaires
- Projection trésorerie (6 mois)
- Runway (mois de trésorerie restants)

**Données :**
- Requête `trpc.dashboard.getKPIs`
- Requête `trpc.dashboard.getPriorityActions`

---

### 7.2 Sales CRM - Vue Kanban

**Fichier :** `client/src/pages/CRM.tsx`

**Colonnes :**
1. Lead
2. Négociation
3. Proposition
4. Closing
5. Gagné
6. Perdu

**Fonctionnalités :**
- Drag & drop entre colonnes
- Affichage montant + date
- Création de deals via modal

**Données :**
- Requête `trpc.crm.getDeals`
- Mutation `trpc.crm.createDeal`

---

### 7.3 Contacts & Entreprises - Galerie

**Fichier :** `client/src/pages/Companies.tsx`

**Affichage :**
- Cartes avec gradient amber
- Secteur, CA cumulé, pays
- Dernier contact

**Fonctionnalités :**
- Recherche par nom/secteur
- Création d'entreprise via modal

**Données :**
- Requête `trpc.companies.list`
- Mutation `trpc.companies.create`

---

### 7.4 Ops & Projets

**Fichier :** `client/src/pages/Projects.tsx`

**Statuts :**
- En cours (barre de progression)
- À risque (alertes)
- Terminés

**Fonctionnalités :**
- Affichage dates début/fin
- Barre de progression
- Création de projet

**Données :**
- Requête `trpc.projects.list`
- Mutation `trpc.projects.create`

---

### 7.5 Finance & P&L

**Fichier :** `client/src/pages/Finance.tsx`

**Affichage :**
- Synthèse (Revenus, Dépenses, Résultat Net)
- Tableau transactions récentes
- Catégories (Salaires, Marketing, etc.)

**Fonctionnalités :**
- Création de transaction
- Filtrage par date/catégorie

**Données :**
- Requête `trpc.finance.getTransactions`
- Mutation `trpc.finance.createTransaction`

---

### 7.6 Objectifs & OKR

**Fichier :** `client/src/pages/Objectives.tsx`

**Affichage :**
- Titre + description
- Période (Q1 2024, 2024, etc.)
- Barre de progression
- Responsable

**Fonctionnalités :**
- Création d'objectif
- Suivi progression

**Données :**
- Requête `trpc.objectives.list`
- Mutation `trpc.objectives.create`

---

## 8. Authentification OAuth

### 8.1 Flux d'Authentification

```
1. Utilisateur clique "Se connecter"
2. Redirection vers VITE_OAUTH_PORTAL_URL
3. Authentification Manus
4. Callback à /api/oauth/callback
5. Création/Mise à jour utilisateur en BD
6. Cookie de session créé
7. Redirection vers dashboard
```

### 8.2 Utilisation dans les Composants

```typescript
import { useAuth } from "@/_core/hooks/useAuth";

export default function MyComponent() {
  const { user, isAuthenticated, logout } = useAuth();

  if (!isAuthenticated) {
    return <div>Non authentifié</div>;
  }

  return (
    <div>
      Bienvenue {user?.name}
      <button onClick={logout}>Déconnexion</button>
    </div>
  );
}
```

---

## 9. Procédures tRPC - Référence Complète

### 9.1 Queries (Lecture)

```typescript
// Dashboard
trpc.dashboard.getKPIs.useQuery()
trpc.dashboard.getPriorityActions.useQuery()

// CRM
trpc.crm.getDeals.useQuery()
trpc.crm.getDealsByStatus.useQuery("won")

// Companies
trpc.companies.list.useQuery()
trpc.companies.getById.useQuery(1)
trpc.companies.getContacts.useQuery(1)

// Projects
trpc.projects.list.useQuery()
trpc.projects.getById.useQuery(1)

// Finance
trpc.finance.getTransactions.useQuery()

// Objectives
trpc.objectives.list.useQuery()

// Admin
trpc.admin.getWorkloadUsers.useQuery()
```

### 9.2 Mutations (Écriture)

```typescript
// Companies
trpc.companies.create.useMutation({
  onSuccess: () => { /* refetch */ }
})

// CRM
trpc.crm.createDeal.useMutation()

// Projects
trpc.projects.create.useMutation()

// Finance
trpc.finance.createTransaction.useMutation()

// Objectives
trpc.objectives.create.useMutation()

// Auth
trpc.auth.logout.useMutation()
```

---

## 10. Styling & Thème

### 10.1 Couleurs Principales

```css
/* Thème Sombre + Amber */
--background: #0f172a (slate-950)
--foreground: #ffffff
--accent: #fbbf24 (amber-500)
--accent-dark: #f59e0b (amber-600)
--border: #334155 (slate-700)
--card-bg: #1e293b (slate-800)
```

### 10.2 Tailwind Configuration

Le projet utilise Tailwind CSS 4 avec les variables CSS personnalisées.

Voir `client/src/index.css` pour les tokens de design.

---

## 11. Checklist de Déploiement

- [ ] Toutes les variables d'env configurées
- [ ] Tables Supabase créées
- [ ] RLS activé sur les tables sensibles
- [ ] Tests locaux réussis (`pnpm dev`)
- [ ] Build production réussi (`pnpm build`)
- [ ] Secrets Vercel/Coolify configurés
- [ ] Domain name configuré
- [ ] SSL/TLS activé
- [ ] Monitoring & alertes configurés
- [ ] Backup Supabase activé

---

## 12. Troubleshooting

### Erreur : "Database not available"

**Cause :** `DATABASE_URL` non configurée ou invalide

**Solution :**
```bash
# Vérifier la variable
echo $DATABASE_URL

# Récupérer depuis Supabase
# Settings → Database → Connection string
```

### Erreur : "tRPC procedure not found"

**Cause :** Procédure non définie dans `server/routers.ts`

**Solution :**
1. Vérifier le nom de la procédure
2. Vérifier que le router est exporté
3. Redémarrer le serveur

### Erreur : "Unauthorized"

**Cause :** Utilisateur non authentifié ou token expiré

**Solution :**
1. Vérifier le cookie de session
2. Se reconnecter
3. Vérifier `JWT_SECRET`

---

## 13. Contact & Support

Pour toute question sur l'architecture ou l'implémentation :
- Consulter la documentation Drizzle : https://orm.drizzle.team
- Consulter la documentation tRPC : https://trpc.io
- Consulter la documentation Supabase : https://supabase.com/docs

---

**Version :** 1.0  
**Dernière mise à jour :** 30 Oct 2024  
**Auteur :** Manus AI
