# Guide Détaillé des Relations Supabase - Pilotage Business

## 1. Vue d'Ensemble des Relations

### 1.1 Diagramme Entité-Relation (ER)

```
                                    ┌─────────────┐
                                    │   USERS     │
                                    │ (Auth)      │
                                    └──────┬──────┘
                                           │
                    ┌──────────────────────┼──────────────────────┐
                    │                      │                      │
                    │ (companyId)          │ (responsibleUserId)  │ (userId)
                    ↓                      ↓                      ↓
        ┌───────────────────┐  ┌──────────────────┐  ┌──────────────────┐
        │    COMPANIES      │  │     DEALS        │  │  PROJECT_ASSIGN  │
        │  (Clients)        │  │  (Pipeline CRM)  │  │  (Équipe projets)│
        └────────┬──────────┘  └────────┬─────────┘  └──────────────────┘
                 │                      │
        ┌────────┴──────────┐           │
        │                   │           │
        │ (companyId)       │ (contactId)
        ↓                   ↓
    ┌──────────────┐  ┌──────────────┐
    │  CONTACTS    │  │   DEALS      │
    │  (Personnes) │  │  (Continued) │
    └──────────────┘  └──────────────┘
        │
        │ (companyId)
        ↓
    ┌──────────────────┐
    │    PROJECTS      │
    │  (Ops & Projets) │
    └────────┬─────────┘
             │
             │ (projectId)
             ↓
    ┌──────────────────────┐
    │ FINANCE_TRANSACTIONS │
    │  (Revenus/Dépenses)  │
    └──────────────────────┘
             │
             │ (categoryId)
             ↓
    ┌──────────────────────┐
    │ FINANCE_CATEGORIES   │
    │  (Structure P&L)     │
    └──────────────────────┘

    ┌──────────────────────┐
    │   OBJECTIVES         │
    │  (OKR / KPIs)        │
    └──────────────────────┘

    ┌──────────────────────┐
    │  PRIORITY_ACTIONS    │
    │  (Actions Priorit.)  │
    └──────────────────────┘

    ┌──────────────────────┐
    │   FINANCE_FORECAST   │
    │  (Projections)       │
    └──────────────────────┘

    ┌──────────────────────┐
    │    KPI_CACHE         │
    │  (Cache Dashboard)   │
    └──────────────────────┘

    ┌──────────────────────┐
    │   WORKLOAD_USERS     │
    │  (Surcharge équipe)  │
    └──────────────────────┘
```

---

## 2. Détail des Relations

### 2.1 Relation : USERS ↔ COMPANIES

**Type :** Many-to-One (Plusieurs utilisateurs par entreprise)

```sql
-- Foreign Key
ALTER TABLE users ADD CONSTRAINT fk_users_company
  FOREIGN KEY (companyId) REFERENCES companies(id) ON DELETE SET NULL;
```

**Cas d'usage :**
- Un utilisateur appartient à une entreprise (son employeur)
- Plusieurs utilisateurs peuvent appartenir à la même entreprise

**Exemple de requête :**
```sql
-- Tous les utilisateurs d'une entreprise
SELECT * FROM users WHERE companyId = 5;

-- Entreprise d'un utilisateur
SELECT c.* FROM companies c
  JOIN users u ON u.companyId = c.id
  WHERE u.id = 10;
```

---

### 2.2 Relation : COMPANIES ↔ CONTACTS

**Type :** One-to-Many (Une entreprise a plusieurs contacts)

```sql
-- Foreign Key
ALTER TABLE contacts ADD CONSTRAINT fk_contacts_company
  FOREIGN KEY (companyId) REFERENCES companies(id) ON DELETE CASCADE;
```

**Cas d'usage :**
- Une entreprise a plusieurs personnes de contact
- Quand une entreprise est supprimée, ses contacts le sont aussi

**Exemple de requête :**
```sql
-- Tous les contacts d'une entreprise
SELECT * FROM contacts WHERE companyId = 5;

-- Entreprise d'un contact
SELECT c.* FROM companies c
  JOIN contacts co ON co.companyId = c.id
  WHERE co.id = 12;

-- Contact principal d'une entreprise
SELECT * FROM contacts 
  WHERE companyId = 5 AND isMainContact = TRUE;
```

---

### 2.3 Relation : COMPANIES ↔ DEALS

**Type :** One-to-Many (Une entreprise a plusieurs deals)

```sql
-- Foreign Key
ALTER TABLE deals ADD CONSTRAINT fk_deals_company
  FOREIGN KEY (companyId) REFERENCES companies(id) ON DELETE CASCADE;
```

**Cas d'usage :**
- Une entreprise cliente peut avoir plusieurs deals en cours
- Quand une entreprise est supprimée, ses deals le sont aussi

**Exemple de requête :**
```sql
-- Tous les deals d'une entreprise
SELECT * FROM deals WHERE companyId = 5 ORDER BY amount DESC;

-- Deals en cours pour une entreprise
SELECT * FROM deals 
  WHERE companyId = 5 
  AND status NOT IN ('won', 'lost');

-- Montant total des deals en cours
SELECT SUM(amount) as total_pipeline 
  FROM deals 
  WHERE companyId = 5 AND status NOT IN ('won', 'lost');
```

---

### 2.4 Relation : CONTACTS ↔ DEALS

**Type :** One-to-Many (Un contact peut être lié à plusieurs deals)

```sql
-- Foreign Key
ALTER TABLE deals ADD CONSTRAINT fk_deals_contact
  FOREIGN KEY (contactId) REFERENCES contacts(id) ON DELETE SET NULL;
```

**Cas d'usage :**
- Un deal est associé à une personne de contact (décideur)
- Quand un contact est supprimé, ses deals restent mais sans contact

**Exemple de requête :**
```sql
-- Deals d'un contact spécifique
SELECT * FROM deals WHERE contactId = 12;

-- Contact principal pour un deal
SELECT co.* FROM contacts co
  JOIN deals d ON d.contactId = co.id
  WHERE d.id = 100;

-- Deals par contact (avec nombre)
SELECT co.firstName, co.lastName, COUNT(d.id) as deal_count
  FROM contacts co
  LEFT JOIN deals d ON d.contactId = co.id
  GROUP BY co.id
  ORDER BY deal_count DESC;
```

---

### 2.5 Relation : USERS ↔ DEALS

**Type :** One-to-Many (Un utilisateur responsable de plusieurs deals)

```sql
-- Foreign Key
ALTER TABLE deals ADD CONSTRAINT fk_deals_user
  FOREIGN KEY (responsibleUserId) REFERENCES users(id) ON DELETE SET NULL;
```

**Cas d'usage :**
- Un deal est assigné à un commercial (responsable)
- Quand un utilisateur est supprimé, ses deals restent non assignés

**Exemple de requête :**
```sql
-- Deals d'un commercial
SELECT * FROM deals WHERE responsibleUserId = 7;

-- Montant total des deals par commercial
SELECT u.name, SUM(d.amount) as total_amount
  FROM users u
  LEFT JOIN deals d ON d.responsibleUserId = u.id
  GROUP BY u.id
  ORDER BY total_amount DESC;

-- Deals à risque (non fermés depuis 30 jours)
SELECT d.*, u.name as responsible
  FROM deals d
  LEFT JOIN users u ON d.responsibleUserId = u.id
  WHERE d.status NOT IN ('won', 'lost')
  AND d.updatedAt < NOW() - INTERVAL 30 DAY;
```

---

### 2.6 Relation : COMPANIES ↔ PROJECTS

**Type :** One-to-Many (Une entreprise peut avoir plusieurs projets)

```sql
-- Foreign Key
ALTER TABLE projects ADD CONSTRAINT fk_projects_company
  FOREIGN KEY (companyId) REFERENCES companies(id) ON DELETE CASCADE;
```

**Cas d'usage :**
- Une entreprise cliente peut avoir plusieurs projets en cours
- Quand une entreprise est supprimée, ses projets le sont aussi

**Exemple de requête :**
```sql
-- Tous les projets d'une entreprise
SELECT * FROM projects WHERE companyId = 5;

-- Projets en cours avec progression
SELECT * FROM projects 
  WHERE companyId = 5 AND status = 'in_progress'
  ORDER BY progress DESC;

-- Projets à risque
SELECT * FROM projects 
  WHERE companyId = 5 AND status = 'at_risk';

-- Progression moyenne des projets
SELECT AVG(progress) as avg_progress
  FROM projects 
  WHERE companyId = 5 AND status = 'in_progress';
```

---

### 2.7 Relation : PROJECTS ↔ PROJECT_ASSIGNMENTS

**Type :** One-to-Many (Un projet a plusieurs assignations)

```sql
-- Foreign Keys
ALTER TABLE projectAssignments ADD CONSTRAINT fk_assignments_project
  FOREIGN KEY (projectId) REFERENCES projects(id) ON DELETE CASCADE;

ALTER TABLE projectAssignments ADD CONSTRAINT fk_assignments_user
  FOREIGN KEY (userId) REFERENCES users(id) ON DELETE CASCADE;
```

**Cas d'usage :**
- Un projet a une équipe de plusieurs personnes
- Une personne peut travailler sur plusieurs projets
- Relation Many-to-Many via table de jonction

**Exemple de requête :**
```sql
-- Équipe d'un projet
SELECT u.*, pa.role
  FROM projectAssignments pa
  JOIN users u ON pa.userId = u.id
  WHERE pa.projectId = 20;

-- Projets d'un utilisateur
SELECT p.*, pa.role
  FROM projectAssignments pa
  JOIN projects p ON pa.projectId = p.id
  WHERE pa.userId = 7;

-- Nombre de projets par utilisateur
SELECT u.name, COUNT(pa.projectId) as project_count
  FROM users u
  LEFT JOIN projectAssignments pa ON pa.userId = u.id
  GROUP BY u.id
  ORDER BY project_count DESC;

-- Équipe complète avec détails
SELECT p.name as project, u.name as user, pa.role, p.progress
  FROM projectAssignments pa
  JOIN projects p ON pa.projectId = p.id
  JOIN users u ON pa.userId = u.id
  WHERE p.status = 'in_progress'
  ORDER BY p.name, u.name;
```

---

### 2.8 Relation : FINANCE_TRANSACTIONS ↔ FINANCE_CATEGORIES

**Type :** Many-to-One (Plusieurs transactions par catégorie)

```sql
-- Foreign Key
ALTER TABLE financeTransactions ADD CONSTRAINT fk_transactions_category
  FOREIGN KEY (categoryId) REFERENCES financeCategories(id) ON DELETE RESTRICT;
```

**Cas d'usage :**
- Une transaction appartient à une catégorie (ex: Salaires, Marketing)
- Impossible de supprimer une catégorie si elle a des transactions

**Exemple de requête :**
```sql
-- Transactions d'une catégorie
SELECT * FROM financeTransactions 
  WHERE categoryId = 3 
  ORDER BY date DESC;

-- Total par catégorie
SELECT fc.name, fc.type, SUM(ft.amount) as total
  FROM financeTransactions ft
  JOIN financeCategories fc ON ft.categoryId = fc.id
  WHERE MONTH(ft.date) = 10 AND YEAR(ft.date) = 2024
  GROUP BY fc.id
  ORDER BY total DESC;

-- Dépenses par catégorie (hiérarchie)
SELECT fc.name, SUM(ft.amount) as total
  FROM financeTransactions ft
  JOIN financeCategories fc ON ft.categoryId = fc.id
  WHERE fc.type = 'expense' AND MONTH(ft.date) = 10
  GROUP BY fc.id
  ORDER BY total DESC;
```

---

### 2.9 Relation : FINANCE_CATEGORIES (Hiérarchie)

**Type :** Self-Join (Une catégorie peut avoir une parent)

```sql
-- Foreign Key (Self-referencing)
ALTER TABLE financeCategories ADD CONSTRAINT fk_categories_parent
  FOREIGN KEY (parentId) REFERENCES financeCategories(id) ON DELETE CASCADE;
```

**Cas d'usage :**
- Créer une hiérarchie de catégories (parent → enfants)
- Exemple : Dépenses → Salaires → Salaires CDI

**Exemple de requête :**
```sql
-- Catégories racine (sans parent)
SELECT * FROM financeCategories WHERE parentId IS NULL;

-- Sous-catégories d'une catégorie
SELECT * FROM financeCategories WHERE parentId = 5;

-- Hiérarchie complète (avec récursion)
WITH RECURSIVE category_tree AS (
  SELECT id, name, parentId, 0 as level
  FROM financeCategories
  WHERE parentId IS NULL
  
  UNION ALL
  
  SELECT fc.id, fc.name, fc.parentId, ct.level + 1
  FROM financeCategories fc
  JOIN category_tree ct ON fc.parentId = ct.id
)
SELECT * FROM category_tree ORDER BY level, name;
```

---

### 2.10 Relation : FINANCE_TRANSACTIONS ↔ COMPANIES

**Type :** Many-to-One (Plusieurs transactions par entreprise)

```sql
-- Foreign Key
ALTER TABLE financeTransactions ADD CONSTRAINT fk_transactions_company
  FOREIGN KEY (companyId) REFERENCES companies(id) ON DELETE SET NULL;
```

**Cas d'usage :**
- Une transaction peut être liée à une entreprise cliente (optionnel)
- Permet de tracker les revenus par client

**Exemple de requête :**
```sql
-- Revenus d'une entreprise cliente
SELECT SUM(amount) as total_revenue
  FROM financeTransactions
  WHERE companyId = 5 AND type = 'revenue';

-- Revenus par client
SELECT c.name, SUM(ft.amount) as revenue
  FROM financeTransactions ft
  JOIN companies c ON ft.companyId = c.id
  WHERE ft.type = 'revenue'
  GROUP BY c.id
  ORDER BY revenue DESC;
```

---

### 2.11 Relation : USERS ↔ OBJECTIVES

**Type :** One-to-Many (Un utilisateur responsable de plusieurs objectifs)

```sql
-- Foreign Key
ALTER TABLE objectives ADD CONSTRAINT fk_objectives_user
  FOREIGN KEY (responsibleUserId) REFERENCES users(id) ON DELETE SET NULL;
```

**Cas d'usage :**
- Un objectif est assigné à une personne responsable
- Permet de tracker les OKR par personne

**Exemple de requête :**
```sql
-- Objectifs d'une personne
SELECT * FROM objectives WHERE responsibleUserId = 7;

-- Progression moyenne des objectifs par personne
SELECT u.name, AVG(o.progress) as avg_progress
  FROM objectives o
  JOIN users u ON o.responsibleUserId = u.id
  WHERE o.status = 'in_progress'
  GROUP BY u.id;
```

---

### 2.12 Relation : USERS ↔ PRIORITY_ACTIONS

**Type :** One-to-Many (Un utilisateur assigné à plusieurs actions)

```sql
-- Foreign Key
ALTER TABLE priorityActions ADD CONSTRAINT fk_actions_user
  FOREIGN KEY (assignedUserId) REFERENCES users(id) ON DELETE SET NULL;
```

**Cas d'usage :**
- Une action prioritaire est assignée à une personne
- Permet de tracker les tâches urgentes

**Exemple de requête :**
```sql
-- Actions assignées à une personne
SELECT * FROM priorityActions 
  WHERE assignedUserId = 7 AND status != 'completed'
  ORDER BY priority DESC, dueDate ASC;

-- Actions critiques non complétées
SELECT * FROM priorityActions 
  WHERE priority = 'critical' AND status != 'completed'
  ORDER BY dueDate ASC;
```

---

### 2.13 Relation : USERS ↔ WORKLOAD_USERS

**Type :** One-to-One (Un utilisateur a un enregistrement de surcharge)

```sql
-- Foreign Key (Unique pour One-to-One)
ALTER TABLE workloadUsers ADD CONSTRAINT fk_workload_user
  FOREIGN KEY (userId) REFERENCES users(id) ON DELETE CASCADE;

ALTER TABLE workloadUsers ADD UNIQUE(userId);
```

**Cas d'usage :**
- Tracker la surcharge de chaque utilisateur
- Nombre de projets assignés
- Flag si surcharge détectée

**Exemple de requête :**
```sql
-- Utilisateurs surchargés
SELECT u.name, wu.projectCount
  FROM workloadUsers wu
  JOIN users u ON wu.userId = u.id
  WHERE wu.isOverloaded = TRUE
  ORDER BY wu.projectCount DESC;

-- Utilisateurs avec moins de 3 projets
SELECT u.name, wu.projectCount
  FROM workloadUsers wu
  JOIN users u ON wu.userId = u.id
  WHERE wu.projectCount < 3
  ORDER BY wu.projectCount ASC;
```

---

## 3. Requêtes Complexes Multi-Tables

### 3.1 Pipeline Commercial Complet

```sql
-- Vue complète du pipeline avec tous les détails
SELECT 
  d.id,
  d.name as deal_name,
  d.amount,
  d.status,
  d.expectedCloseDate,
  c.name as company_name,
  c.sector,
  co.firstName || ' ' || co.lastName as contact_name,
  u.name as responsible_user,
  d.createdAt,
  d.updatedAt
FROM deals d
LEFT JOIN companies c ON d.companyId = c.id
LEFT JOIN contacts co ON d.contactId = co.id
LEFT JOIN users u ON d.responsibleUserId = u.id
ORDER BY d.amount DESC;
```

### 3.2 Dashboard KPIs

```sql
-- Tous les KPIs en une requête
SELECT 
  (SELECT SUM(amount) FROM deals WHERE status = 'won') as total_revenue,
  (SELECT COUNT(*) FROM projects WHERE status = 'in_progress') as active_projects,
  (SELECT COUNT(*) FROM deals WHERE status = 'won') as won_deals,
  (SELECT COUNT(*) FROM deals WHERE status IN ('lead', 'negotiation', 'proposition', 'closing')) as open_deals,
  (SELECT COUNT(*) FROM companies) as total_companies,
  (SELECT COUNT(*) FROM users WHERE isActive = TRUE) as active_users;
```

### 3.3 Analyse Financière

```sql
-- Synthèse financière du mois en cours
SELECT 
  fc.name as category,
  fc.type,
  SUM(CASE WHEN ft.type = 'revenue' THEN ft.amount ELSE 0 END) as revenue,
  SUM(CASE WHEN ft.type = 'expense' THEN ft.amount ELSE 0 END) as expense,
  SUM(CASE WHEN ft.type = 'revenue' THEN ft.amount ELSE -ft.amount END) as net_result
FROM financeTransactions ft
JOIN financeCategories fc ON ft.categoryId = fc.id
WHERE MONTH(ft.date) = MONTH(NOW()) AND YEAR(ft.date) = YEAR(NOW())
GROUP BY fc.id
ORDER BY fc.type, revenue DESC;
```

### 3.4 Surcharge d'Équipe

```sql
-- Détail de la surcharge d'équipe
SELECT 
  u.name,
  COUNT(DISTINCT pa.projectId) as project_count,
  COUNT(DISTINCT d.id) as deal_count,
  COUNT(DISTINCT o.id) as objective_count,
  COUNT(DISTINCT pa.projectId) + COUNT(DISTINCT d.id) as total_assignments,
  CASE 
    WHEN COUNT(DISTINCT pa.projectId) + COUNT(DISTINCT d.id) > 5 THEN 'OVERLOADED'
    WHEN COUNT(DISTINCT pa.projectId) + COUNT(DISTINCT d.id) > 3 THEN 'HIGH'
    ELSE 'NORMAL'
  END as workload_status
FROM users u
LEFT JOIN projectAssignments pa ON u.id = pa.userId
LEFT JOIN deals d ON u.id = d.responsibleUserId AND d.status NOT IN ('won', 'lost')
LEFT JOIN objectives o ON u.id = o.responsibleUserId
WHERE u.isActive = TRUE
GROUP BY u.id
ORDER BY total_assignments DESC;
```

---

## 4. Bonnes Pratiques

### 4.1 Indexation

Pour optimiser les requêtes, créer des index sur les colonnes fréquemment utilisées :

```sql
-- Index sur les foreign keys
CREATE INDEX idx_users_companyId ON users(companyId);
CREATE INDEX idx_contacts_companyId ON contacts(companyId);
CREATE INDEX idx_deals_companyId ON deals(companyId);
CREATE INDEX idx_deals_contactId ON deals(contactId);
CREATE INDEX idx_deals_responsibleUserId ON deals(responsibleUserId);
CREATE INDEX idx_projects_companyId ON projects(companyId);
CREATE INDEX idx_projectAssignments_projectId ON projectAssignments(projectId);
CREATE INDEX idx_projectAssignments_userId ON projectAssignments(userId);
CREATE INDEX idx_financeTransactions_categoryId ON financeTransactions(categoryId);
CREATE INDEX idx_financeTransactions_companyId ON financeTransactions(companyId);
CREATE INDEX idx_financeTransactions_date ON financeTransactions(date);

-- Index sur les colonnes de recherche
CREATE INDEX idx_companies_name ON companies(name);
CREATE INDEX idx_contacts_email ON contacts(email);
CREATE INDEX idx_deals_status ON deals(status);
CREATE INDEX idx_projects_status ON projects(status);
```

### 4.2 Contraintes d'Intégrité

```sql
-- Vérifier que les montants sont positifs
ALTER TABLE deals ADD CONSTRAINT check_deal_amount CHECK (amount > 0);
ALTER TABLE financeTransactions ADD CONSTRAINT check_transaction_amount CHECK (amount > 0);

-- Vérifier que les dates de fin sont après les dates de début
ALTER TABLE projects ADD CONSTRAINT check_project_dates CHECK (endDate IS NULL OR endDate >= startDate);

-- Vérifier que la progression est entre 0 et 100
ALTER TABLE projects ADD CONSTRAINT check_project_progress CHECK (progress >= 0 AND progress <= 100);
ALTER TABLE objectives ADD CONSTRAINT check_objective_progress CHECK (progress >= 0 AND progress <= 100);
```

### 4.3 Triggers pour Maintenance

```sql
-- Mettre à jour lastContactDate quand un deal est gagné
CREATE TRIGGER update_company_last_contact
AFTER UPDATE ON deals
FOR EACH ROW
WHEN NEW.status = 'won'
BEGIN
  UPDATE companies SET lastContactDate = NOW() WHERE id = NEW.companyId;
END;

-- Mettre à jour la surcharge d'équipe
CREATE TRIGGER update_workload_on_assignment
AFTER INSERT ON projectAssignments
FOR EACH ROW
BEGIN
  UPDATE workloadUsers 
  SET projectCount = (SELECT COUNT(*) FROM projectAssignments WHERE userId = NEW.userId),
      lastUpdated = NOW()
  WHERE userId = NEW.userId;
END;
```

---

## 5. Exemples d'Utilisation en Code

### 5.1 Récupérer les Deals d'un Commercial (Drizzle ORM)

```typescript
import { drizzle } from "drizzle-orm/mysql2";
import { eq } from "drizzle-orm";
import { deals, companies, contacts, users } from "./schema";

const db = drizzle(process.env.DATABASE_URL);

export async function getDealsByUser(userId: number) {
  return await db
    .select({
      id: deals.id,
      name: deals.name,
      amount: deals.amount,
      status: deals.status,
      companyName: companies.name,
      contactName: contacts.firstName,
      expectedCloseDate: deals.expectedCloseDate,
    })
    .from(deals)
    .leftJoin(companies, eq(deals.companyId, companies.id))
    .leftJoin(contacts, eq(deals.contactId, contacts.id))
    .where(eq(deals.responsibleUserId, userId))
    .orderBy(deals.amount);
}
```

### 5.2 Créer un Deal avec Toutes les Relations

```typescript
export async function createDealWithRelations(data: {
  name: string;
  amount: number;
  companyId: number;
  contactId?: number;
  responsibleUserId?: number;
  expectedCloseDate?: Date;
}) {
  const db = drizzle(process.env.DATABASE_URL);

  // Vérifier que la compagnie existe
  const company = await db
    .select()
    .from(companies)
    .where(eq(companies.id, data.companyId))
    .limit(1);

  if (!company.length) {
    throw new Error("Company not found");
  }

  // Créer le deal
  const result = await db.insert(deals).values({
    name: data.name,
    amount: data.amount.toString(),
    companyId: data.companyId,
    contactId: data.contactId || null,
    responsibleUserId: data.responsibleUserId || null,
    expectedCloseDate: data.expectedCloseDate || null,
    status: "lead",
  });

  return result;
}
```

### 5.3 Assigner une Personne à un Projet

```typescript
export async function assignUserToProject(
  projectId: number,
  userId: number,
  role: string
) {
  const db = drizzle(process.env.DATABASE_URL);

  // Vérifier que le projet existe
  const project = await db
    .select()
    .from(projects)
    .where(eq(projects.id, projectId))
    .limit(1);

  if (!project.length) {
    throw new Error("Project not found");
  }

  // Vérifier que l'utilisateur existe
  const user = await db
    .select()
    .from(users)
    .where(eq(users.id, userId))
    .limit(1);

  if (!user.length) {
    throw new Error("User not found");
  }

  // Créer l'assignation
  await db.insert(projectAssignments).values({
    projectId,
    userId,
    role,
  });

  // Mettre à jour la surcharge
  const assignmentCount = await db
    .select()
    .from(projectAssignments)
    .where(eq(projectAssignments.userId, userId));

  await db
    .update(workloadUsers)
    .set({
      projectCount: assignmentCount.length,
      isOverloaded: assignmentCount.length > 5,
      lastUpdated: new Date(),
    })
    .where(eq(workloadUsers.userId, userId));
}
```

---

## 6. Résumé des Relations

| De | Vers | Type | Clé Étrangère | Cas d'Usage |
|---|---|---|---|---|
| users | companies | Many-to-One | companyId | Utilisateur appartient à une entreprise |
| contacts | companies | One-to-Many | companyId | Entreprise a plusieurs contacts |
| deals | companies | One-to-Many | companyId | Entreprise a plusieurs deals |
| deals | contacts | Many-to-One | contactId | Deal lié à un contact |
| deals | users | Many-to-One | responsibleUserId | Deal assigné à un commercial |
| projects | companies | One-to-Many | companyId | Entreprise a plusieurs projets |
| projectAssignments | projects | Many-to-One | projectId | Assignation liée à un projet |
| projectAssignments | users | Many-to-One | userId | Assignation liée à un utilisateur |
| financeTransactions | financeCategories | Many-to-One | categoryId | Transaction dans une catégorie |
| financeTransactions | companies | Many-to-One | companyId | Transaction liée à une entreprise |
| financeCategories | financeCategories | Self-Join | parentId | Hiérarchie de catégories |
| objectives | users | Many-to-One | responsibleUserId | Objectif assigné à une personne |
| priorityActions | users | Many-to-One | assignedUserId | Action assignée à une personne |
| workloadUsers | users | One-to-One | userId | Surcharge d'une personne |

---

**Prêt à implémenter les relations dans votre application ! 🚀**
