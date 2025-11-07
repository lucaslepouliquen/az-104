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

**Gestion des Licences après Synchronisation :**

**⚠️ Important :** Les licences doivent être assignées manuellement ou automatiquement après synchronisation.

**Méthode 1 - Assignation Manuelle :**
```powershell
# Via PowerShell
Connect-MsolService
Set-MsolUser -UserPrincipalName "user@contoso.com" -UsageLocation "FR"
Set-MsolUserLicense -UserPrincipalName "user@contoso.com" -AddLicenses "contoso:ENTERPRISEPACK"

# Ou via Azure Portal
# Azure AD → Users → Select user → Licenses → Add assignments

# Ou via Microsoft 365 Admin Center
# Users → Active users → Select user → Manage product licenses
```

**Méthode 2 - Assignation Automatique (Recommandée) :**

**Processus d'assignation automatique de licences :**
1. **Créer un groupe de sécurité dynamique** basé sur des attributs personnalisés
2. **Configurer les règles** du groupe dynamique
3. **Assigner des licences au groupe** (Group-based licensing)
4. **Les nouveaux utilisateurs synchronisés** reçoivent automatiquement les licences

```powershell
# Exemple de règle de groupe dynamique
(user.department -eq "Sales") -and (user.usageLocation -eq "FR")
```

**Points clés :**
- **Dynamic security groups** : Obligatoires pour assignation automatique
- **Custom attributes** : Base des règles de groupe
- **Group-based licensing** : Synchronisation automatique
- **Usage Location** : Doit être défini avant assignation de licence

## 1.2 Role-Based Access Control (RBAC)

### Rôles Built-in Essentiels

**⚠️ Principe du Moindre Privilège :** Toujours utiliser le rôle avec le minimum de permissions requis

**Rôles Azure RBAC (Gestion des Ressources) :**

**Owner**
- **Permissions** : Accès complet à toutes les ressources
- **Capacités** : Création, modification, suppression + délégation d'accès
- **Usage** : Administration complète avec gestion des accès
- **Scope** : Ressources Azure (VMs, Storage, Networks, etc.)

**Contributor**
- **Permissions** : Accès complet aux ressources
- **Limitation** : ❌ Ne peut PAS déléguer l'accès à d'autres utilisateurs
- **Usage** : Développement et administration des ressources sans gestion des accès
- **Scope** : Tous types de ressources Azure

**Reader**
- **Permissions** : Lecture seule
- **Limitation** : ❌ Aucune action de modification autorisée
- **Usage** : Monitoring, audit, consultation
- **Scope** : Visualisation des ressources existantes

**User Access Administrator**
- **Permissions** : Gestion des accès uniquement
- **Limitation** : ❌ Ne peut PAS créer/modifier les ressources
- **Usage** : Délégation des permissions RBAC
- **Scope** : Assignations de rôles uniquement

### Rôles Administratifs Azure AD (Gestion des Identités)

**User Administrator**
- **Permissions** : Création et gestion des utilisateurs et groupes
- **Capacités** : Gestion des tickets de support, monitoring de la santé des services
- **Usage** : Administration dédiée aux utilisateurs sans privilèges excessifs
- **Scope** : Gestion des identités uniquement
- **✅ Recommandé** : Pour gestion quotidienne des utilisateurs

**Global Administrator**
- **Permissions** : Accès complet à toutes les fonctionnalités Azure AD
- **Capacités** : Plus de permissions que nécessaire pour la gestion basique
- **Usage** : Administration complète du tenant
- **⚠️ Attention** : Utiliser avec parcimonie, trop de permissions pour tâches simples

**Billing Administrator**
- **Permissions** : Gestion des aspects financiers et de facturation
- **Capacités** : Achats, abonnements, gestion des coûts
- **Usage** : Administration financière
- **Scope** : Facturation et finances uniquement

**Service Administrator (Classic)**
- **Type** : Rôle classique (Legacy)
- **Permissions** : Accès complet aux services Azure
- **⚠️ Deprecated** : Remplacé par les rôles RBAC modernes
- **Usage** : Non recommandé pour nouvelles installations

### Rôles Spécialisés

**Rôles Ressources Spécifiques :**
- **Virtual Machine Contributor** : Gestion complète des VMs
- **Storage Account Contributor** : Gestion des comptes de stockage
- **Network Contributor** : Gestion des ressources réseau
- **API Management Service Contributor** : Configuration et maintenance des APIs

**⚠️ Comparaison pour l'Examen :**

| Besoin | Rôle Approprié | Raison |
|--------|---------------|--------|
| Gérer utilisateurs et groupes | **User Administrator** | Moindre privilège suffisant |
| Créer et gérer VMs | **Contributor** ou **VM Contributor** | Pas besoin de gestion d'accès |
| Consulter les ressources | **Reader** | Lecture seule |
| Gérer les permissions | **Owner** ou **User Access Administrator** | Délégation requise |
| Administration complète tenant | **Global Administrator** | Privilèges maximum |

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

### Limites et Accès

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

**⚠️ Accès au Root Management Group :**
- **Aucun accès par défaut** au root management group
- Seuls les **Global Administrators** peuvent s'élever pour obtenir l'accès
- **Processus** : 
  1. Global Admin → Azure Portal
  2. Azure AD → Properties
  3. "Access management for Azure resources" → Enable
  4. Assign roles au root management group

**⚠️ Délais de Propagation (RBAC et Policies) :**

Ces délais concernent **uniquement** les assignations RBAC et Policy, **PAS les Resource Locks**.

| Type d'assignation | Délai de propagation | Scope |
|-------------------|---------------------|-------|
| **RBAC assignments** | Jusqu'à 10 minutes | Management Groups, Subscriptions, Resource Groups |
| **Policy assignments** | Jusqu'à 30 minutes | Management Groups, Subscriptions, Resource Groups |
| **Resource Locks** | ✅ **Immédiat** | Subscriptions, Resource Groups, Resources |

**⚠️ Important pour l'examen :**
- **Locks** : Effectifs **immédiatement** après création
- **RBAC** : Peut prendre jusqu'à **10 minutes** pour que les permissions soient actives
- **Policies** : Peut prendre jusqu'à **30 minutes** pour évaluation de la conformité

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

**⚠️ Héritage des Locks - Point Critique pour l'Examen :**

Les locks suivent un principe d'**héritage hiérarchique** :

**Delete Lock sur Resource Group :**
```
Resource Group avec Delete Lock
├── ✅ Empêche suppression du Resource Group lui-même
├── ✅ Empêche AUSSI suppression des ressources enfants (héritage)
└── ✅ PERMET modifications des ressources enfants
```

**Exemple concret :**
```
RG "Production-RG" avec Delete Lock :
✅ Cannot delete the Resource Group
✅ Cannot delete VM1 in Production-RG (inherited)
✅ Cannot delete Storage Account in Production-RG (inherited)
✅ CAN modify/stop VM1
✅ CAN upload/delete blobs in Storage Account
✅ CAN change VM size, add disks, etc.
```

**Hiérarchie d'héritage des Locks :**

| Scope Lock | Impact sur Enfants | Exemple |
|-----------|-------------------|---------|
| **Subscription Lock** | ✅ S'applique à **tous** les Resource Groups et ressources | Lock sur Subscription → Protège toutes les ressources |
| **Resource Group Lock** | ✅ S'applique à **toutes** les ressources du RG | Lock sur RG → Protège toutes les VMs, Storage, etc. |
| **Resource Lock** | ❌ S'applique **uniquement** à cette ressource | Lock sur VM1 → Protège uniquement VM1 |

**⚠️ Piège d'examen classique :**

| Question | Réponse Incorrecte ❌ | Réponse Correcte ✅ |
|----------|----------------------|---------------------|
| "Delete Lock sur RG permet de supprimer les ressources ?" | "Oui, le lock ne concerne que le RG" | "Non, le lock est hérité par toutes les ressources enfants" |
| "Delete Lock sur RG empêche de modifier les VMs ?" | "Oui, tout est bloqué" | "Non, on peut modifier, juste pas supprimer" |
| "Comment supprimer une VM dans un RG avec Delete Lock ?" | "Impossible" | "Retirer le lock du RG d'abord, puis supprimer" |

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

**⚠️ Note** : La syntaxe CLI peut varier selon la version. Vérifiez toujours avec `az storage container immutability-policy --help`

```bash
# Option 1: Créer une politique d'immutabilité time-based (période de rétention)
az storage container immutability-policy create \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --period 365 \
  --account-key <storage-key>

# Option 2: Verrouiller la politique (mode Locked - irréversible)
az storage container immutability-policy lock \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --if-match "<etag>" \
  --account-key <storage-key>

# Option 3: Via Account-level (recommandé pour 2024+)
az storage account blob-service-properties update \
  --account-name mystorageaccount \
  --resource-group myRG \
  --enable-versioning true \
  --default-service-version "2021-06-08"
```

**Modes de politique :**
- **Unlocked** : Test mode, peut être modifié/supprimé
- **Locked** : Production mode, **irréversible**, compliance garantie

**Caractéristiques :**
- ✅ Blobs ne peuvent pas être modifiés ou supprimés pendant la période de rétention
- ✅ Protection contre ransomware et suppression accidentelle
- ✅ Compliance réglementaire (SEC 17a-4(f), FINRA 4511, CFTC, etc.)
- ✅ Legal Hold disponible pour rétention indéfinie

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

