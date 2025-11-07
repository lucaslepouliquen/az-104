# AZ-104 - Section 1: Manage Azure Identities and Governance (15-20%)

## 📑 Table des matières

- [1.1 Azure Active Directory (Azure AD)](#11-azure-active-directory-azure-ad)
  - [Concepts Fondamentaux](#concepts-fondamentaux)
  - [Utilisateurs et Groupes](#utilisateurs-et-groupes)
  - [Membership Types](#membership-types)
  - [Custom Domains](#custom-domains)
  - [Licensing et Dynamic Groups](#licensing-et-dynamic-groups)
  - [B2B Collaboration](#b2b-collaboration)
  - [Azure AD Connect - Synchronisation Hybrid](#azure-ad-connect---synchronisation-hybrid)
- [1.2 Role-Based Access Control (RBAC)](#12-role-based-access-control-rbac)
  - [Rôles Built-in Essentiels](#rôles-built-in-essentiels)
  - [Rôles Administratifs Azure AD](#rôles-administratifs-azure-ad)
  - [Rôles Spécialisés](#rôles-spécialisés)
  - [Scopes d'assignation RBAC - Détaillé](#scopes-dassignation-rbac---détaillé)
- [1.3 Azure Policy](#13-azure-policy)
  - [Concepts Clés](#concepts-clés)
  - [Effects Principaux - Détaillé](#effects-principaux---détaillé)
  - [Built-in Policies Courantes](#built-in-policies-courantes)
- [1.4 Management Groups](#14-management-groups)
  - [Hiérarchie](#hiérarchie)
  - [Limites](#limites)
  - [Resource Locks](#resource-locks)

---

## 1.1 Azure Active Directory (Azure AD)

### Concepts Fondamentaux
- **Tenant** : Instance Azure AD pour une organisation
- **Subscription** : Conteneur de facturation lié à un tenant
- **Directory** : Synonyme de tenant Azure AD

### Utilisateurs et Groupes

**Types d'utilisateurs :**
- **Cloud Identity** : Créé directement dans Azure AD
- **Directory Synchronized** : Synchronisé depuis AD on-premises
- **Guest User** : Utilisateur externe (Azure AD B2B)

**Types de groupes :**

**1. Security Groups**
- **Gestion** : Azure AD Portal
- **Usage** : Gestion des permissions RBAC, accès aux ressources Azure
- **Membres** : Utilisateurs, devices, service principals, autres groupes
- **Membership** : Assigned ou Dynamic

**2. Microsoft 365 Groups**
- **Gestion** : Azure AD Portal, Microsoft 365 Admin Center
- **Usage** : Collaboration (Teams, SharePoint, Outlook, Planner)
- **Membres** : Utilisateurs uniquement
- **Membership** : Assigned ou Dynamic
- **Caractéristiques** : Boîte mail partagée, calendrier, SharePoint site

**3. Distribution Groups (Mail-Enabled Groups)**
- **Gestion** : **Exchange Admin Center** (Exchange Online)
- **⚠️ Important** : NE sont PAS gérés directement dans Azure AD Portal
- **Usage** : Listes de diffusion email uniquement
- **Membres** : Utilisateurs avec adresses email
- **Membership** : Assigned uniquement (pas de dynamic)
- **Limitation** : Ne peuvent PAS être utilisés pour les permissions Azure

**⚠️ Point de Confusion Fréquent :**

| Type | Géré dans Azure AD Portal? | Usage Permissions? | Usage Email? |
|------|---------------------------|-------------------|-------------|
| **Security Group** | ✅ Oui | ✅ Oui | ❌ Non* |
| **Microsoft 365 Group** | ✅ Oui | ✅ Oui | ✅ Oui |
| **Distribution Group** | ❌ Non (Exchange Admin) | ❌ Non | ✅ Oui |

\*Peut être mail-enabled si configuré dans Exchange

**Accès aux Distribution Groups :**
```
Exchange Admin Center → Recipients → Groups → Distribution list
OU
Microsoft 365 Admin Center → Teams & groups → Distribution lists
```

**⚠️ Pour l'examen AZ-104 :**
- Distribution Groups = **Exchange Online**, pas Azure AD
- Pour permissions Azure = Utiliser **Security Groups**
- Pour collaboration + email = Utiliser **Microsoft 365 Groups**

### Membership Types
- **Assigned** : Ajout manuel des membres
- **Dynamic User** : Règles basées sur les attributs utilisateur
- **Dynamic Device** : Règles basées sur les attributs d'appareil

** Erreur fréquente identifiée :** Syntaxe des règles dynamiques
```
// Correct
(user.department -eq "Marketing") and (user.country -eq "France")

// Propriétés utilisateur courantes
user.department, user.country, user.city, user.jobTitle, user.userPrincipalName
```

### Custom Domains

**Processus d'ajout :**
1. Ajouter le domaine dans Azure AD
2. Créer un enregistrement DNS pour vérification
3. Vérifier la propriété du domaine

** Point d'attention identifié :** Types d'enregistrements DNS acceptés
- **TXT** : Méthode recommandée (plus flexible)
- **MX** : Alternative acceptable
- Exemple : `MS=ms12345678` dans un enregistrement TXT

### Licensing et Dynamic Groups

** Processus d'assignation automatique de licences :**
1. **Créer un groupe de sécurité dynamique** basé sur des attributs personnalisés
2. **Configurer les règles** du groupe dynamique
3. **Ajouter le groupe à un groupe de licences** pour synchronisation automatique
4. **Tous les utilisateurs** du groupe reçoivent automatiquement la licence

** Points clés identifiés :**
- **Dynamic security groups** : Obligatoires pour assignation automatique
- **Custom attributes** : Base des règles de groupe
- **License groups** : Synchronisation automatique requise
- **Automatic assignment policies** : Non utilisées pour les licences

### B2B Collaboration

** Configuration des paramètres de collaboration externe :**
- **External collaboration settings** : Contrôlent qui peut inviter des utilisateurs externes
- **Domain restrictions** : Autoriser/bloquer des domaines spécifiques
- **Guest user visibility** : Contrôler ce que voient les invités dans l'annuaire
- **Conditional Access** : Renforcer l'authentification et bloquer l'accès depuis des emplacements inconnus
- **Cross-tenant access** : Configuration de collaboration avec des organisations Microsoft Entra spécifiques

** Format UPN des utilisateurs invités :**
- **Guest users** : `bsmith_contoso.com#EXT#@fabrikam.com`
- **Regular users** : `user@fabrikam.com`
- **Access reviews** : Non utilisées pour contrôler les invitations d'invités

** Prérequis pour assignation de licences :**
- **Usage location** : Obligatoire avant assignation de licence
- **Not all Microsoft 365 services** disponibles dans tous les emplacements
- **First name, Last name, Other email, User type** : Non obligatoires pour assignation de licence

### Azure AD Connect - Synchronisation Hybrid

**⚠️ Erreur Courante QCM : Synchronisation des Licences Microsoft 365**

**❌ FAUX :** Azure AD Connect synchronise les licences Microsoft 365
**✅ CORRECT :** Azure AD Connect synchronise UNIQUEMENT les objets utilisateur et leurs attributs, **PAS les licences**

**Ce qui est synchronisé par Azure AD Connect :**
- Utilisateurs (User objects)
- Groupes (Groups)
- Attributs utilisateur (UPN, displayName, email, etc.)
- Mots de passe (Password Hash Sync ou Pass-through Authentication)
- Objets d'appareil (si configuré)

**Ce qui N'EST PAS synchronisé :**
- ❌ Licences Microsoft 365
- ❌ Paramètres Exchange Online
- ❌ Permissions SharePoint
- ❌ Rôles Azure AD (ils doivent être réassignés)

**Actions nécessaires après synchronisation :**
```powershell
# 1. Assigner des licences via PowerShell
Connect-MsolService
Set-MsolUser -UserPrincipalName "user@contoso.com" -UsageLocation "FR"
Set-MsolUserLicense -UserPrincipalName "user@contoso.com" -AddLicenses "contoso:ENTERPRISEPACK"

# 2. Ou via Azure Portal
# Azure AD → Users → Select user → Licenses → Add assignments

# 3. Ou via Microsoft 365 Admin Center
# Users → Active users → Select user → Manage product licenses
```

**Best Practice - Assignation automatique de licences :**
1. Créer un groupe de sécurité dynamique basé sur des attributs
2. Assigner des licences au groupe (Group-based licensing)
3. Les nouveaux utilisateurs synchronisés reçoivent automatiquement les licences

```powershell
# Exemple de règle de groupe dynamique
(user.department -eq "Sales") -and (user.usageLocation -eq "FR")
```

## 1.2 Role-Based Access Control (RBAC)

### Rôles Built-in Essentiels
- **Owner** : Accès complet + gestion des accès
- **Contributor** : Accès complet sauf gestion des accès
- **Reader** : Lecture seule
- **User Access Administrator** : Gestion des accès uniquement

### Rôles Administratifs Azure AD

**User Administrator**
- **Permissions** : Création et gestion des utilisateurs et groupes
- **Capacités** : Gestion des tickets de support, monitoring de la santé des services
- **Usage** : Administration dédiée aux utilisateurs sans privilèges excessifs
- **Scope** : Gestion des identités uniquement

**Global Administrator**
- **Permissions** : Accès complet à toutes les fonctionnalités Azure AD
- **Capacités** : Plus de permissions que nécessaire pour la gestion basique
- **Usage** : Administration complète du tenant
- **Principe** : Utiliser le rôle le moins privilégié possible

**Billing Administrator**
- **Permissions** : Gestion des aspects financiers et de facturation
- **Capacités** : Achats, abonnements, gestion des coûts
- **Usage** : Administration financière
- **Scope** : Facturation et finances uniquement

**Service Administrator**
- **Type** : Rôle classique (Classic)
- **Permissions** : Accès complet aux services Azure
- **Usage** : Rôle legacy, non requis pour gestion utilisateurs
- **Note** : Remplacé par les rôles RBAC modernes

** Point clé identifié :** Principe du moindre privilège
- **User Administrator** : Suffisant pour gestion utilisateurs et groupes
- **Global Administrator** : Trop de permissions pour tâches simples
- **Règle** : Toujours utiliser le rôle avec le minimum de permissions requis

### Rôles Spécialisés
- **Virtual Machine Contributor** : Gestion des VMs
- **Storage Account Contributor** : Gestion des comptes de stockage
- **Network Contributor** : Gestion des ressources réseau

** Différenciation des rôles essentiels :**

**Contributor**
- **Création et gestion** : Tous types de ressources
- **Limitation** : Ne peut pas déléguer l'accès à d'autres utilisateurs
- **Usage** : Développement et administration des ressources

**Reader**
- **Visualisation** : Ressources Azure existantes
- **Aucune action** : Pas d'actions autorisées sur les ressources
- **Usage** : Monitoring et audit

**API Management Service Contributor**
- **Scope limité** : Services API Management et APIs uniquement
- **Gestion spécialisée** : Configuration et maintenance des APIs
- **Usage** : Administration des services API

**Owner**
- **Accès complet** : Toutes les ressources
- **Délégation** : Possibilité de déléguer l'accès à d'autres utilisateurs
- **Usage** : Administration complète avec gestion des accès

### Scopes d'assignation RBAC - Détaillé

**⚠️ Erreur Courante QCM : Niveaux d'assignation et héritage**

**Hiérarchie des Scopes (du plus large au plus précis) :**

```
Management Group (Racine)
    ↓ (héritage automatique vers le bas)
Subscriptions
    ↓ (héritage automatique vers le bas)
Resource Groups
    ↓ (héritage automatique vers le bas)
Resources (VM, Storage, VNet, etc.)
```

**✅ Principe d'Héritage RBAC :**
- Rôle au **Management Group** → S'applique à **toutes** les subscriptions et ressources sous-jacentes
- Rôle à la **Subscription** → S'applique à **tous** les Resource Groups et ressources
- Rôle au **Resource Group** → S'applique à **toutes** les ressources du groupe
- Rôle à une **Resource** → S'applique **uniquement** à cette ressource

**Exemples Pratiques :**

```bash
# 1. Niveau Subscription - Accès à TOUS les Resource Groups
az role assignment create \
  --assignee user@contoso.com \
  --role "Contributor" \
  --scope "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

# 2. Niveau Resource Group - Accès à TOUTES les ressources du RG
az role assignment create \
  --assignee user@contoso.com \
  --role "Virtual Machine Contributor" \
  --resource-group "Production-RG"

# 3. Niveau Resource - Accès SEULEMENT à cette VM spécifique
az role assignment create \
  --assignee user@contoso.com \
  --role "Virtual Machine Contributor" \
  --scope "/subscriptions/xxx/resourceGroups/Production-RG/providers/Microsoft.Compute/virtualMachines/VM1"
```

**Scénarios d'Examen :**

| Besoin | Scope | Justification |
|--------|-------|---------------|
| Gérer toutes les VMs de l'entreprise | Management Group | Accès multi-subscriptions |
| Gérer toutes les ressources d'un environnement | Subscription | Accès à tous les RGs |
| Gérer les ressources d'un projet | Resource Group | Limité au projet |
| Gérer une VM critique | Resource | Accès ultra-restreint |

**⚠️ Best Practice - Least Privilege :**
- ✅ TOUJOURS assigner au scope le **plus restreint** possible
- ❌ ÉVITER Owner/Contributor au niveau Subscription
- ✅ UTILISER des rôles spécifiques (Storage Blob Data Contributor, etc.)

**Validation des assignations :**
```bash
# Lister les assignations d'un utilisateur
az role assignment list --assignee user@contoso.com --output table

# Vérifier les permissions sur une ressource
az role assignment list --scope "/subscriptions/xxx/resourceGroups/myRG"
```

** Erreur identifiée :** Root Management Group
- **Aucun accès par défaut** au root management group
- Seuls les **Global Administrators** peuvent s'élever
- Process : Global Admin → "Access management for Azure resources" → Assign roles

## 1.3 Azure Policy

### Concepts Clés
- **Policy Definition** : Règle de conformité
- **Policy Assignment** : Application d'une policy à un scope
- **Initiative** : Collection de policies
- **Compliance** : État de conformité des ressources

### Effects Principaux - Détaillé

**⚠️ Erreur Courante QCM : Différence entre Deny et Audit**

| Effet | Action | Quand utiliser |
|-------|--------|----------------|
| **Deny** | ❌ **BLOQUE** la création/modification | Standards stricts, compliance obligatoire |
| **Audit** | ✅ Permet mais **LOG** comme non-compliant | Identifier les ressources non conformes |
| **Append** | Ajoute des propriétés manquantes | Tags automatiques |
| **Modify** | Modifie des propriétés existantes | Corriger configurations |
| **DeployIfNotExists** | Déploie une ressource si absente | Agents de monitoring |
| **AuditIfNotExists** | Audit si ressource absente | Vérifier présence de sécurité |

**Différence Clé Deny vs Audit :**

**Deny - Prévention (Enforcement)**
- ❌ **Bloque AVANT** la création de la ressource
- ✅ **Assure compliance dès le départ**
- **Use case** : Empêcher création de VMs sans tags, bloquer régions non autorisées
- **Impact** : Les utilisateurs ne peuvent PAS créer de ressources non conformes

**Audit - Détection (Visibility)**
- ✅ **Permet la création**, mais log comme non-compliant
- 📊 **Identifie les ressources** à corriger plus tard
- **Use case** : Découvrir les ressources existantes non conformes, phase de test
- **Impact** : Les ressources sont créées, mais marquées pour révision

**Exemple Pratique - Bloquer VMs sans tag "Environment" :**

```json
{
  "mode": "Indexed",
  "policyRule": {
    "if": {
      "allOf": [
        {
          "field": "type",
          "equals": "Microsoft.Compute/virtualMachines"
        },
        {
          "field": "tags['Environment']",
          "exists": "false"
        }
      ]
    },
    "then": {
      "effect": "deny"
    }
  }
}
```

**Assigner une policy avec effet Deny :**
```bash
az policy assignment create \
  --name "require-environment-tag" \
  --policy "/subscriptions/xxx/providers/Microsoft.Authorization/policyDefinitions/xxx" \
  --scope "/subscriptions/xxx" \
  --params '{"effect": {"value": "Deny"}}'
```

**Scénarios d'examen :**
- **"Prevent users from..."** → Utiliser **Deny**
- **"Identify resources that..."** → Utiliser **Audit**
- **"Automatically add tags..."** → Utiliser **Append**
- **"Deploy monitoring agent if missing..."** → Utiliser **DeployIfNotExists**

### Built-in Policies Courantes
- Require tags on resources
- Allowed virtual machine SKUs
- Allowed storage account SKUs
- Require SSL for storage accounts

## 1.4 Management Groups

### Hiérarchie
```
Root Management Group
├── Production MG
│   ├── Prod Subscription 1
│   └── Prod Subscription 2
└── Development MG
    ├── Dev Subscription 1
    └── Test Subscription 1
```

### Limites

**⚠️ Limites Techniques des Management Groups :**

**Hiérarchie :**
- **6 niveaux** de profondeur maximum (en dessous du root management group)
- **Note** : Le Root Management Group n'est PAS compté dans les 6 niveaux
- **Exemple** : Root → Level 1 → Level 2 → Level 3 → Level 4 → Level 5 → Level 6 (Subscriptions)

**Capacité :**
- **10,000 management groups** maximum par tenant Azure AD
- **Note** : Cette limite inclut tous les management groups (root + enfants)

**Contraintes :**
- ✅ **Une subscription** peut appartenir à **un seul** management group
- ✅ **Un management group** peut avoir **plusieurs** enfants (subscriptions ou MG)
- ✅ **Un management group** ne peut avoir **qu'un seul** parent

**Délai de propagation :**
- **Policy assignments** : Jusqu'à 30 minutes pour propager
- **RBAC assignments** : Jusqu'à 10 minutes pour propager

**Référence** : [Azure Management Groups - Limits and recommendations](https://learn.microsoft.com/azure/governance/management-groups/overview)

### Resource Locks

**⚠️ Point d'Attention : Comprendre les Niveaux de Protection des Locks**

**Types de Locks :**

**1. Delete Lock (CanNotDelete)**
- **Protection** : Empêche la suppression de la ressource
- **Permet** : Modifications et lecture de la ressource
- **Usage** : Protection contre suppression accidentelle

**2. Read-Only Lock (ReadOnly)**
- **Protection** : Empêche suppression ET modification
- **Permet** : Lecture seule
- **Usage** : Protection maximale (compliance, audit)

**⚠️ CLARIFICATION IMPORTANTE : Storage Account Locks**

**Ce que les locks protègent :**
- ✅ **Le Storage Account lui-même** : Ne peut pas être supprimé (Delete Lock)
- ✅ **Les propriétés du compte** : Configuration, SKU, réplication

**Ce que les locks NE protègent PAS :**
- ❌ **Les données DANS le Storage Account** : Blobs, fichiers, tables, queues
- ❌ **Les opérations sur les données** : Upload, modification, suppression de blobs
- ❌ **Les containers/shares** : Peuvent être créés/supprimés

**Exemple concret :**
```
Storage Account avec Delete Lock :
✅ Cannot delete the storage account
❌ CAN delete/modify blobs inside the account
❌ CAN delete containers
❌ CAN upload/overwrite files
```

**Pour protéger les DONNÉES dans un Storage Account, utilisez :**

**1. Immutable Storage (WORM - Write Once, Read Many)**
```bash
# Activer immutable storage sur un container
az storage container immutability-policy create \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --period 365 \
  --policy-mode Locked
```
- ✅ Blobs ne peuvent pas être modifiés ou supprimés
- ✅ Protection contre ransomware
- ✅ Compliance réglementaire (SEC 17a-4, FINRA, etc.)

**2. Soft Delete**
```bash
# Activer soft delete pour blobs
az storage account blob-service-properties update \
  --account-name mystorageaccount \
  --enable-delete-retention true \
  --delete-retention-days 30
```
- ✅ Récupération de blobs supprimés (7-365 jours)
- ✅ Protection contre suppression accidentelle
- ✅ Snapshots et versions préservés

**3. Versioning**
```bash
# Activer versioning
az storage account blob-service-properties update \
  --account-name mystorageaccount \
  --enable-versioning true
```
- ✅ Historique complet des versions
- ✅ Récupération de versions précédentes
- ✅ Protection contre écrasement

**Ressources Supportées pour Locks :**
- ✅ **Virtual Machines** : Empêche suppression de la VM
- ✅ **Subscriptions** : Protège toutes les ressources de la souscription
- ✅ **Resource Groups** : Protège le groupe et toutes ses ressources
- ✅ **Storage Accounts** : Protège le compte, PAS les données
- ✅ **Networks, Databases, etc.** : Toutes ressources Azure

**Ressources NON Supportées pour Locks :**
- ❌ **Management Groups** : Cannot add locks to management groups
- ❌ **Data inside resources** : Blobs, SQL rows, files

**Scénarios d'Examen :**

| Besoin | Solution | Raison |
|--------|----------|--------|
| **Empêcher suppression Storage Account** | **Delete Lock** | Protège la ressource |
| **Protéger données (blobs) dans Storage** | **Immutable Storage + Soft Delete** | Locks ne protègent pas les données |
| **Empêcher suppression accidentelle VM** | **Delete Lock** | Protection de la ressource VM |
| **Protection contre ransomware** | **Immutable Storage** | Blobs non modifiables |
| **Compliance réglementaire** | **Immutable Storage (Locked)** | WORM garantit intégrité |

