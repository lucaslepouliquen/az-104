# AZ-104 Microsoft Azure Administrator - Guide Complet d'Examen

## 📋 Table des Matières
1. [Manage Azure Identities and Governance (15-20%)](#1-manage-azure-identities-and-governance)
2. [Implement and Manage Storage (15-20%)](#2-implement-and-manage-storage)
   - [Azure Data Lake Storage Gen2](#24-azure-data-lake-storage-gen2)
3. [Deploy and Manage Azure Compute Resources (20-25%)](#3-deploy-and-manage-azure-compute-resources)
4. [Configure and Manage Virtual Networking (25-30%)](#4-configure-and-manage-virtual-networking)
5. [Monitor and Backup Azure Resources (10-15%)](#5-monitor-and-backup-azure-resources)

---

## 1. Manage Azure Identities and Governance (15-20%)

### 1.1 Azure Active Directory (Azure AD)

#### Concepts Fondamentaux
- **Tenant** : Instance Azure AD pour une organisation
- **Subscription** : Conteneur de facturation lié à un tenant
- **Directory** : Synonyme de tenant Azure AD

#### Utilisateurs et Groupes

**Types d'utilisateurs :**
- **Cloud Identity** : Créé directement dans Azure AD
- **Directory Synchronized** : Synchronisé depuis AD on-premises
- **Guest User** : Utilisateur externe (Azure AD B2B)

**Types de groupes :**
- **Security Groups** : Gestion des permissions
- **Microsoft 365 Groups** : Collaboration (Teams, SharePoint, etc.)
- **Distribution Groups** : Listes de diffusion email

#### Membership Types
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

#### Custom Domains

**Processus d'ajout :**
1. Ajouter le domaine dans Azure AD
2. Créer un enregistrement DNS pour vérification
3. Vérifier la propriété du domaine

** Point d'attention identifié :** Types d'enregistrements DNS acceptés
- **TXT** : Méthode recommandée (plus flexible)
- **MX** : Alternative acceptable
- Exemple : `MS=ms12345678` dans un enregistrement TXT

#### Licensing et Dynamic Groups

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

#### B2B Collaboration

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

#### Azure AD Connect - Synchronisation Hybrid

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

### 1.2 Role-Based Access Control (RBAC)

#### Rôles Built-in Essentiels
- **Owner** : Accès complet + gestion des accès
- **Contributor** : Accès complet sauf gestion des accès
- **Reader** : Lecture seule
- **User Access Administrator** : Gestion des accès uniquement

#### Rôles Administratifs Azure AD

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

#### Rôles Spécialisés
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

#### Scopes d'assignation RBAC - Détaillé

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

### 1.3 Azure Policy

#### Concepts Clés
- **Policy Definition** : Règle de conformité
- **Policy Assignment** : Application d'une policy à un scope
- **Initiative** : Collection de policies
- **Compliance** : État de conformité des ressources

#### Effects Principaux - Détaillé

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

#### Built-in Policies Courantes
- Require tags on resources
- Allowed virtual machine SKUs
- Allowed storage account SKUs
- Require SSL for storage accounts

### 1.4 Management Groups

#### Hiérarchie
```
Root Management Group
├── Production MG
│   ├── Prod Subscription 1
│   └── Prod Subscription 2
└── Development MG
    ├── Dev Subscription 1
    └── Test Subscription 1
```

#### Limites
- **6 niveaux** de profondeur maximum
- **10,000 management groups** par tenant
- Chaque subscription dans un seul management group

#### Resource Locks

** Types de verrous et limitations :**

**Delete Locks**
- **Protection** : Bloque la suppression de ressources
- **Ressources supportées** : Virtual machines, subscriptions, resource groups
- **Ressources non supportées** : Management groups, storage account data
- **Usage** : Protection contre suppression accidentelle

** Points clés identifiés :**
- **Delete locks** : Empêchent la suppression mais pas la modification
- **Management groups** : Ne peuvent pas être verrouillés
- **Storage account data** : Données non protégées par les locks
- **Scope** : Applicable aux VMs, subscriptions, et resource groups uniquement

---

## 2. Implement and Manage Storage (15-20%)

### 2.1 Storage Accounts

#### Types de Storage Accounts (Mise à jour 2024)

**General Purpose v2 (GPv2) - Standard**
- **Services** : Blobs, Files, Queues, Tables
- **Performance** : Standard (HDD) ou Premium (SSD) selon le service
- **Réplication** : Toutes les options (LRS, ZRS, GRS, GZRS, RA-GRS, RA-GZRS)
- **Usage** : Polyvalent, recommandé pour la plupart des cas
- **Nouveauté 2024** : Support des Premium File Shares (SSD) sur GPv2

**Premium Block Blobs (BlobStorage)**
- **Services** : Blobs uniquement (Block, Page, Append)
- **Performance** : Premium (SSD) uniquement
- **Réplication** : LRS, ZRS uniquement
- **Usage** : Applications haute performance, bases de données
- **Avantage** : IOPS élevées, latence faible

**Premium File Shares (FileStorage)**
- **Services** : Files uniquement
- **Performance** : Premium (SSD) uniquement
- **Réplication** : LRS, ZRS uniquement
- **Usage** : Partages de fichiers haute performance
- **Nouveauté 2024** : Support NFS 4.1 avec chiffrement en transit

**Erreur fréquente identifiée :** Confusion entre types de Storage Accounts pour Azure Files

**Types de Storage Accounts et support Azure Files :**

**FileStorage Accounts**
- **Support** : Premium File Shares uniquement (SSD, haute performance)
- **Usage** : Applications nécessitant des performances élevées
- **Limitation** : Ne supporte PAS les Standard File Shares

**General Purpose v2 (StorageV2)**
- **Support** : Standard File Shares uniquement (HDD, performance standard)
- **Usage** : Applications générales, partages de fichiers basiques
- **Limitation** : Ne supporte PAS les Premium File Shares

**BlobStorage Accounts**
- **Support** : Aucun support d'Azure Files
- **Usage** : Stockage de blobs uniquement
- **Limitation** : Pas de file shares du tout

**Piège d'examen courant :**
- **Erreur** : Essayer de créer des Premium File Shares sur un compte StorageV2
- **Correct** : Utiliser un FileStorage account pour les Premium File Shares
- **Règle** : Type de compte = Type de file share supporté

#### Services de Stockage Azure - Différences Clés

**Comparaison des 4 services de stockage principaux :**

**1. Blob Storage (Binary Large Objects)**
- **Usage** : Stockage de fichiers non structurés (documents, images, vidéos, backups)
- **Types de blobs** : Block, Page, Append
- **Accès** : REST API, SDK, Azure Storage Explorer
- **Cas d'usage** : Sites web statiques, archives, médias, sauvegardes
- **Niveaux** : Hot, Cool, Archive (optimisation des coûts)

**2. Azure Files (File Shares)**
- **Usage** : Partages de fichiers réseau (comme un NAS/SAN cloud)
- **Protocoles** : SMB 3.0/3.1, NFS 4.1 (Premium uniquement)
- **Accès** : Mappage de lecteurs réseau, montage Linux
- **Cas d'usage** : Migration d'applications on-premises, partage de fichiers entre VMs
- **Port requis** : 445 TCP pour SMB

**3. Azure Queues (Message Queuing)**
- **Usage** : Messaging asynchrone entre composants d'application
- **Fonctionnalités** : FIFO, TTL, visibility timeout
- **Accès** : REST API, SDK
- **Cas d'usage** : Découplage d'applications, traitement asynchrone, workflows
- **Limite** : Messages jusqu'à 64 KB

**4. Azure Tables (NoSQL Database)**
- **Usage** : Base de données NoSQL pour données structurées
- **Structure** : Entités avec propriétés (clé-valeur)
- **Accès** : REST API, SDK, OData
- **Cas d'usage** : Logs d'application, métadonnées, données de configuration
- **Limite** : Entités jusqu'à 1 MB

**Matrice de décision rapide :**

| Besoin | Service | Raison |
|--------|---------|--------|
| Stocker des fichiers (images, docs) | **Blob Storage** | Optimisé pour fichiers non structurés |
| Partager des fichiers entre VMs | **Azure Files** | Protocoles SMB/NFS natifs |
| Communication asynchrone | **Azure Queues** | Messaging découplé |
| Stocker des données structurées | **Azure Tables** | Base NoSQL simple |
| Site web statique | **Blob Storage** | Hébergement web statique |
| Migration d'applications | **Azure Files** | Compatibilité SMB |
| Workflow de traitement | **Azure Queues** | Orchestration asynchrone |
| Logs et métadonnées | **Azure Tables** | Stockage clé-valeur |

**Points d'attention pour l'examen :**
- **Blob Storage** : Le plus polyvalent, supporte tous les types de fichiers
- **Azure Files** : Seul service avec protocoles réseau natifs (SMB/NFS)
- **Azure Queues** : Seul service de messaging asynchrone
- **Azure Tables** : Seule base de données NoSQL intégrée
- **Performance** : Premium uniquement pour Blobs et Files
- **Réplication** : Tous supportent LRS, certains limités pour ZRS/GRS

#### Blob Access Tiers - Optimisation des Coûts

**⚠️ Erreur Courante QCM : Choisir le bon tier selon le pattern d'accès**

**Vue d'ensemble :**
Les Access Tiers permettent d'optimiser les coûts de stockage en fonction de la fréquence d'accès aux données.

**Comparaison Complète des Tiers :**

| Tier | Use Case | Disponibilité | Coût Stockage | Coût Accès | Latence | Durée min | Suppression anticipée |
|------|----------|--------------|---------------|------------|---------|-----------|----------------------|
| **Hot** | Données fréquemment accédées | Immédiate | $$$ Élevé | $ Faible | Ms | Aucune | Non |
| **Cool** | Données peu accédées (>30 jours) | Immédiate | $$ Moyen | $$ Moyen | Ms | 30 jours | Oui |
| **Cold** | Données rarement accédées (>90 jours) | Immédiate | $ Faible | $$$ Élevé | Ms | 90 jours | Oui |
| **Archive** | Archivage long terme (>180 jours) | Après réhydratation | $ Très faible | $$$$ Très élevé | Heures | 180 jours | Oui |

**1. Hot Tier - Données Actives**

**Caractéristiques :**
- **Pattern d'accès** : Données accédées fréquemment (quotidiennement)
- **Coût stockage** : ~$0.018/GB/mois (le plus élevé)
- **Coût accès** : ~$0.0004/10,000 read operations (le plus faible)
- **SLA** : Identique aux autres tiers (99.9% pour LRS)
- **Latence** : Millisecondes

**Use Cases :**
- Sites web actifs (images, CSS, JS)
- Données applicatives en production
- Fichiers logs actifs
- Bases de données actives
- Contenu média streaming

**Configuration :**
```bash
# Set default tier à Hot lors de la création
az storage account create \
  --name mystorageaccount \
  --resource-group myRG \
  --location eastus \
  --sku Standard_LRS \
  --access-tier Hot

# Changer le tier par défaut d'un compte existant
az storage account update \
  --name mystorageaccount \
  --resource-group myRG \
  --access-tier Hot
```

**2. Cool Tier - Données Occasionnelles**

**Caractéristiques :**
- **Pattern d'accès** : Données accédées occasionnellement (1x/mois minimum)
- **Coût stockage** : ~$0.010/GB/mois (45% moins cher que Hot)
- **Coût accès** : ~$0.01/10,000 read operations (25x plus cher que Hot)
- **Durée minimum** : 30 jours (facturation complète même si supprimé avant)
- **Pénalité** : Si supprimé avant 30 jours, facturation du reste de la période

**Use Cases :**
- Backups court terme (30-90 jours)
- Données de compliance
- Fichiers logs anciens mais accessibles
- Archives à court terme
- Données de développement/test

**Configuration :**
```bash
# Définir Cool tier comme défaut
az storage account update \
  --name mystorageaccount \
  --resource-group myRG \
  --access-tier Cool

# Changer un blob spécifique vers Cool
az storage blob set-tier \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myblob.txt \
  --tier Cool
```

**3. Cold Tier - Données Rarement Accédées (Nouveau en 2024)**

**Caractéristiques :**
- **Pattern d'accès** : Données rarement accédées (quelques fois/an)
- **Coût stockage** : ~$0.005/GB/mois (72% moins cher que Hot)
- **Coût accès** : Plus élevé que Cool
- **Durée minimum** : 90 jours
- **Disponibilité** : Immédiate (pas de réhydratation)

**Use Cases :**
- Backups moyen terme (90-180 jours)
- Archives réglementaires accessibles
- Données forensiques
- Logs long terme avec accès occasionnel

**4. Archive Tier - Archivage Long Terme**

**Caractéristiques :**
- **Pattern d'accès** : Très rarement accédé (plusieurs mois/années)
- **Coût stockage** : ~$0.002/GB/mois (91% moins cher que Hot)
- **Coût accès** : Très élevé + coût de réhydratation
- **Durée minimum** : 180 jours
- **Latence** : Heures (réhydratation requise)
- **Offline** : Blob doit être réhydraté avant lecture

**⚠️ POINT CRITIQUE pour l'examen :**
Les blobs en Archive tier sont **OFFLINE** et doivent être réhydratés avant accès.

**Réhydratation (2 options) :**

**A. Standard Rehydration (Économique)**
- **Durée** : Jusqu'à 15 heures
- **Coût** : Standard
- **Use case** : Accès non urgent

```bash
# Réhydratation Standard vers Hot
az storage blob set-tier \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name archivedblob.txt \
  --tier Hot \
  --rehydrate-priority Standard
```

**B. High Priority Rehydration (Rapide)**
- **Durée** : Moins de 1 heure (généralement 30 min pour <10GB)
- **Coût** : ~10x plus cher que Standard
- **Use case** : Accès urgent

```bash
# Réhydratation High Priority vers Hot
az storage blob set-tier \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name archivedblob.txt \
  --tier Hot \
  --rehydrate-priority High
```

**Copy Rehydration (Alternative) :**
```bash
# Copier vers un nouveau blob (garde l'original en Archive)
az storage blob copy start \
  --account-name mystorageaccount \
  --destination-container mycontainer \
  --destination-blob rehydrated-blob.txt \
  --source-uri https://mystorageaccount.blob.core.windows.net/mycontainer/archivedblob.txt \
  --tier Hot \
  --rehydrate-priority High
```

**Use Cases Archive :**
- Compliance long terme (7-10 ans)
- Archives légales
- Backups annuels
- Données historiques
- Forensics cold case

**Lifecycle Management - Automatisation des Transitions**

**⚠️ Feature Clé pour l'AZ-104**

**Vue d'ensemble :**
Lifecycle Management permet d'automatiser les transitions de tiers et la suppression de blobs selon des règles définies.

**Configuration via Azure CLI :**
```bash
# Créer une politique de lifecycle
az storage account management-policy create \
  --account-name mystorageaccount \
  --resource-group myRG \
  --policy @policy.json
```

**Exemple de Politique Complète (policy.json) :**
```json
{
  "rules": [
    {
      "name": "MoveToArchive",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["backups/"]
        },
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 180
            },
            "delete": {
              "daysAfterModificationGreaterThan": 2555
            }
          },
          "snapshot": {
            "tierToCool": {
              "daysAfterCreationGreaterThan": 90
            },
            "delete": {
              "daysAfterCreationGreaterThan": 365
            }
          }
        }
      }
    },
    {
      "name": "DeleteOldLogs",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["logs/"]
        },
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 7
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 90
            },
            "delete": {
              "daysAfterModificationGreaterThan": 365
            }
          }
        }
      }
    }
  ]
}
```

**Actions Disponibles :**

| Action | Description | Use Case |
|--------|-------------|----------|
| **tierToCool** | Déplacer vers Cool | Données peu accédées |
| **tierToCold** | Déplacer vers Cold (2024) | Données rarement accédées |
| **tierToArchive** | Déplacer vers Archive | Archivage long terme |
| **delete** | Supprimer le blob | Nettoyage automatique |
| **enableAutoTierToHotFromCool** | Réhydrater automatiquement si accédé | Optimisation coûts |

**Filtres Disponibles :**
- **blobTypes** : blockBlob, appendBlob, pageBlob
- **prefixMatch** : Filtrer par préfixe (ex: "backups/", "logs/2024/")
- **blobIndexMatch** : Filtrer par tags de métadonnées

**PowerShell - Lifecycle Management :**
```powershell
# Créer règle de lifecycle
$action = New-AzStorageAccountManagementPolicyAction -BaseBlobAction TierToCool `
  -DaysAfterModificationGreaterThan 30
$filter = New-AzStorageAccountManagementPolicyFilter -PrefixMatch "backups/"
$rule = New-AzStorageAccountManagementPolicyRule -Name "MoveToArchive" `
  -Action $action -Filter $filter
$policy = Set-AzStorageAccountManagementPolicy `
  -ResourceGroupName "myRG" `
  -AccountName "mystorageaccount" `
  -Rule $rule
```

**Scénarios d'Examen - Access Tiers**

| Scénario | Solution | Raison |
|----------|----------|--------|
| **Site web avec 10,000 visiteurs/jour** | **Hot tier** | Accès fréquent, coût accès faible critique |
| **Backups mensuels accessibles** | **Cool tier** | Accès occasionnel, durée min 30 jours OK |
| **Archives conformité 7 ans** | **Archive tier** | Accès très rare, coût stockage minimal |
| **Logs applicatifs (30 jours actifs)** | **Hot → Cool (lifecycle)** | Transition automatique après 30 jours |
| **Données dev/test** | **Cool tier** | Accès intermittent, économie 45% |
| **Recovery point long terme** | **Archive tier** | Restauration rare, réhydratation acceptable |

**Matrice de Décision - Calcul de Coût**

**Exemple : 1TB de données pendant 1 an**

| Tier | Stockage/mois | Accès (100 read/mois) | Total/an | Économie vs Hot |
|------|---------------|----------------------|----------|-----------------|
| **Hot** | $18 | $0.04 | $216.48 | - |
| **Cool** | $10 | $1.00 | $132.00 | 39% |
| **Cold** | $5 | $2.00 | $84.00 | 61% |
| **Archive** | $2 | $10.00 + réhydratation | $144.00* | 33% |

*Archive moins avantageux si accès fréquent

**Best Practices - Access Tiers**

✅ **À FAIRE :**
- **Lifecycle policies** pour toutes données avec cycle de vie prévisible
- **Hot tier** pour données production accédées quotidiennement
- **Cool tier** pour backups 30-90 jours
- **Archive tier** pour compliance >180 jours
- **Prefixes** pour faciliter les règles de lifecycle (`/hot/`, `/cool/`, `/archive/`)
- **Monitoring** des coûts par tier (Cost Management)
- **Test réhydratation** avant archivage critique

❌ **À ÉVITER :**
- Archive tier pour données nécessitant accès rapide (15h réhydratation)
- Hot tier pour données rarement accédées (gaspillage)
- Suppression avant durée minimum (pénalités)
- Lifecycle sans préfixes (règles trop larges)
- Oublier coûts d'accès (peut dépasser économies de stockage)

#### Niveaux d'accès (Blob Storage)
- **Hot** : Accès fréquent, coût stockage élevé, coût accès faible
- **Cool** : Accès occasionnel (30 jours minimum), coût moyen
- **Archive** : Accès rare (180 jours minimum), coût très faible, latence haute

#### Réplication et Durabilité

** Points d'attention identifiés :**

**Local Redundant Storage (LRS)**
- 3 copies dans le même datacenter
- Durabilité : 99.999 999 999% (11 nines)
- Protection : Pannes matérielles

**Zone Redundant Storage (ZRS)**
- 3 copies dans 3 zones de disponibilité
- Durabilité : 99.9999999999% (12 nines)
- Protection : Panne d'un datacenter entier
- **Limitation** : Pas disponible dans toutes les régions

**Geo-Redundant Storage (GRS)**
- LRS dans région primaire + 3 copies dans région secondaire
- Durabilité : 99.99999999999999% (16 nines)
- Protection : Panne régionale

**Geo-Zone-Redundant Storage (GZRS)**
- ZRS dans région primaire + 3 copies dans région secondaire
- Durabilité : 99.99999999999999% (16 nines)
- Protection : Panne régionale + panne de zone
- **Limitation** : Pas disponible dans toutes les régions

**Read-Access GRS (RA-GRS)**
- Comme GRS + accès lecture sur région secondaire
- Useful pour applications nécessitant haute disponibilité lecture

**Read-Access GZRS (RA-GZRS)**
- Comme GZRS + accès lecture sur région secondaire
- Combinaison de haute disponibilité et résilience géographique

#### Changement de Type de Réplication (Upgrade/Downgrade)

**⚠️ Erreur Courante QCM : Upgrade LRS → GRS**

**❌ FAUX :** Il faut créer un nouveau Storage Account et migrer les données
**✅ CORRECT :** Vous pouvez **upgrader directement** le type de réplication sans migration

**Conversions de Réplication Supportées :**

| De | Vers | Supporté | Méthode |
|----|------|----------|---------|
| **LRS** | GRS, ZRS, GZRS, RA-GRS, RA-GZRS | ✅ Oui | Portal, CLI, PowerShell |
| **GRS** | LRS, RA-GRS | ✅ Oui | Portal, CLI, PowerShell |
| **ZRS** | GZRS, RA-GZRS | ✅ Oui | Portal, CLI, PowerShell |
| **Premium_LRS** | GRS, ZRS | ❌ Non | Premium ne supporte que LRS/ZRS |

**Méthodes d'Upgrade - Exemples Pratiques :**

**1. Via Azure Portal :**
```
Storage Account → Configuration → Replication
→ Sélectionner GRS ou RA-GRS
→ Save
```

**2. Via Azure CLI :**
```bash
# Upgrade LRS → GRS
az storage account update \
  --name mystorageaccount \
  --resource-group myRG \
  --sku Standard_GRS

# Upgrade LRS → RA-GRS
az storage account update \
  --name mystorageaccount \
  --resource-group myRG \
  --sku Standard_RAGRS

# Upgrade LRS → ZRS
az storage account update \
  --name mystorageaccount \
  --resource-group myRG \
  --sku Standard_ZRS
```

**3. Via PowerShell :**
```powershell
# Upgrade LRS → GRS
Set-AzStorageAccount `
  -ResourceGroupName "myRG" `
  -Name "mystorageaccount" `
  -SkuName "Standard_GRS"

# Upgrade LRS → RA-GRS
Set-AzStorageAccount `
  -ResourceGroupName "myRG" `
  -Name "mystorageaccount" `
  -SkuName "Standard_RAGRS"
```

**⚠️ Points Importants :**

**Limitations :**
- ❌ **Premium Storage** (Premium_LRS pour Page Blobs) ne peut PAS être converti en GRS
- ❌ **FileStorage** et **BlockBlobStorage** limités à LRS/ZRS
- ✅ **General Purpose v2** supporte toutes les options

**Timing et Impact :**
- **Durée** : Peut prendre jusqu'à 24-72 heures pour la réplication initiale
- **Downtime** : ✅ **AUCUN** downtime pendant la conversion
- **Données** : Les données existantes sont automatiquement répliquées
- **Coût** : Augmentation du coût mensuel selon le type choisi

**Vérifier le statut de réplication :**
```bash
# Vérifier le SKU actuel
az storage account show \
  --name mystorageaccount \
  --resource-group myRG \
  --query "sku.name" \
  --output tsv

# Vérifier le statut de geo-replication
az storage account show \
  --name mystorageaccount \
  --resource-group myRG \
  --query "statusOfPrimary" \
  --output tsv
```

**Scénarios d'examen :**
- **"How to enable geo-redundancy?"** → Upgrade to GRS
- **"Minimal downtime during replication change?"** → Direct upgrade (zero downtime)
- **"Can Premium storage use GRS?"** → No, only LRS/ZRS
- **"Read access to secondary region?"** → Use RA-GRS or RA-GZRS

#### Storage Account Firewall et Sécurité Réseau

**⚠️ Erreur Courante QCM : Autoriser les Services Azure via Firewall**

**Scénario :** Vous activez le firewall sur un Storage Account. Comment autoriser Azure Backup ou autres services Azure à accéder ?

**❌ FAUX :** Ajouter les adresses IP publiques des services Azure
**✅ CORRECT :** Utiliser **Trusted Microsoft Services** ou **Service Endpoints**

**Par défaut :**
- Storage Account = **Ouvert à Internet** ("Allow all networks")
- Après activation firewall = **Tout est BLOQUÉ** sauf autorisations explicites

**Solutions pour Autoriser les Services Azure :**

**Solution 1 : Trusted Microsoft Services (Recommandé)**

```bash
# Activer le firewall et autoriser les services Microsoft de confiance
az storage account update \
  --name mystorageaccount \
  --resource-group myRG \
  --default-action Deny \
  --bypass AzureServices
```

**Via Azure Portal :**
```
Storage Account → Networking → Firewalls and virtual networks
→ Sélectionner "Enabled from selected virtual networks and IP addresses"
→ ✅ Cocher "Allow trusted Microsoft services to access this storage account"
```

**Services Concernés (Trusted Microsoft Services) :**
- ✅ **Azure Backup** - Sauvegarde de VMs et données
- ✅ **Azure Site Recovery** - Réplication et DR
- ✅ **Azure File Sync** - Synchronisation de fichiers
- ✅ **Azure Import/Export** - Migration de données
- ✅ **Azure Networking (logs)** - Logs diagnostiques
- ✅ **Azure DevOps** - Pipelines et artefacts
- ✅ **Azure Monitor** - Métriques et logs

**Solution 2 : Service Endpoints (Accès depuis VNet)**

```bash
# 1. Activer le service endpoint sur le subnet
az network vnet subnet update \
  --vnet-name myVNet \
  --name mySubnet \
  --resource-group myRG \
  --service-endpoints Microsoft.Storage

# 2. Ajouter une règle réseau au Storage Account
az storage account network-rule add \
  --account-name mystorageaccount \
  --resource-group myRG \
  --vnet-name myVNet \
  --subnet mySubnet
```

**Avantages Service Endpoints :**
- 🔒 Le trafic reste sur le backbone Azure (pas Internet)
- 🚀 Performance améliorée et latence réduite
- 💰 Pas de frais de transfert de données sortantes
- 🎯 Accès depuis VNet spécifiques uniquement

**Solution 3 : Private Endpoint (Sécurité Maximale)**

```bash
# Créer un Private Endpoint pour le Blob Storage
az network private-endpoint create \
  --name myPrivateEndpoint \
  --resource-group myRG \
  --vnet-name myVNet \
  --subnet mySubnet \
  --private-connection-resource-id "/subscriptions/xxx/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/mystorageaccount" \
  --group-id blob \
  --connection-name myConnection
```

**Avantages Private Endpoint :**
- 🔒 Storage Account obtient une IP **privée** dans votre VNet
- ❌ **Jamais exposé** à Internet
- 🎯 Accès via IP privée uniquement (ex: 10.0.1.10)
- ✅ Compatible avec on-premises via ExpressRoute/VPN

**Solution 4 : Autoriser des IPs Publiques Spécifiques**

```bash
# Autoriser une IP publique spécifique
az storage account network-rule add \
  --account-name mystorageaccount \
  --resource-group myRG \
  --ip-address 203.0.113.50

# Autoriser une plage d'IPs (CIDR)
az storage account network-rule add \
  --account-name mystorageaccount \
  --resource-group myRG \
  --ip-address 203.0.113.0/24
```

**Configuration PowerShell :**
```powershell
# Activer le firewall avec Trusted Services
Update-AzStorageAccountNetworkRuleSet `
  -ResourceGroupName "myRG" `
  -Name "mystorageaccount" `
  -DefaultAction Deny `
  -Bypass AzureServices

# Ajouter une Virtual Network Rule
Add-AzStorageAccountNetworkRule `
  -ResourceGroupName "myRG" `
  -Name "mystorageaccount" `
  -VirtualNetworkResourceId "/subscriptions/xxx/resourceGroups/myRG/providers/Microsoft.Network/virtualNetworks/myVNet/subnets/mySubnet"
```

**⚠️ Points Critiques :**

**Bypass Options :**
- `AzureServices` - Autorise les services Microsoft de confiance
- `Logging` - Autorise les logs Storage Analytics
- `Metrics` - Autorise les métriques Storage Analytics
- `None` - Aucun bypass (tout bloqué)

**Ordre de Priorité des Règles :**
1. **Allow rules** (IP ou VNet) sont évaluées en premier
2. **Default action** (Allow ou Deny) s'applique si aucune règle ne match

**Scénarios d'Examen :**

| Besoin | Solution | Raison |
|--------|----------|--------|
| Azure Backup doit accéder au Storage | **Trusted Services** | Azure Backup est un service de confiance |
| VMs dans VNet doivent accéder | **Service Endpoint** | Accès depuis subnet spécifique |
| Aucun accès Internet nécessaire | **Private Endpoint** | IP privée uniquement |
| Admin depuis bureau doit accéder | **IP Whitelist** | Autoriser IP publique spécifique |

**Vérification et Troubleshooting :**

```bash
# Lister les règles réseau
az storage account network-rule list \
  --account-name mystorageaccount \
  --resource-group myRG

# Vérifier la configuration actuelle
az storage account show \
  --name mystorageaccount \
  --resource-group myRG \
  --query "networkRuleSet"

# Tester l'accès depuis une VM
# (Depuis la VM)
nslookup mystorageaccount.blob.core.windows.net
curl -I https://mystorageaccount.blob.core.windows.net/mycontainer
```

**⚠️ ATTENTION - Pièges Courants :**

1. **Après activation du firewall, pensez à autoriser VOTRE IP** pour continuer à accéder via le Portal !
2. **Service Endpoints** ne fonctionnent QUE pour le trafic depuis Azure (pas on-premises)
3. **Private Endpoints** nécessitent une configuration DNS spécifique
4. **Trusted Services** ne couvre PAS tous les services Azure (vérifier la liste)

**Best Practices :**
- ✅ Utiliser **Trusted Services** pour les services Azure intégrés
- ✅ Utiliser **Service Endpoints** pour les VMs/Apps dans VNets
- ✅ Utiliser **Private Endpoints** pour sécurité maximale
- ❌ **Éviter** "Allow all networks" en production
- ✅ **Toujours** autoriser votre IP admin pour gestion

### 2.2 Azure Files (Mise à jour 2024)

#### Protocoles Supportés
- **SMB 3.0/3.1** : Windows, Linux, macOS
- **NFS 4.1** : Linux, Premium uniquement
- **REST API** : Accès programmatique
- **Nouveauté 2024** : Chiffrement en transit pour NFS 4.1

** Point clé identifié :** Port SMB
- **Port 445 TCP** obligatoire pour accès SMB
- Doit être ouvert sur les firewalls clients
- Nécessaire pour mapper des lecteurs réseau

#### Types de File Shares (Mise à jour 2024)

**Standard File Shares**
- **Comptes** : General Purpose v2 (GPv2)
- **Performance** : Standard (HDD)
- **Capacité** : Jusqu'à 5 TB par share
- **Usage** : Applications générales, partages basiques

**Premium File Shares (SSD)**
- **Comptes** : General Purpose v2 (GPv2) ou FileStorage
- **Performance** : Premium (SSD)
- **Capacité** : Jusqu'à 100 TB par share (Standard) ou 256 TiB (v2 approvisionné)
- **Usage** : Applications haute performance, bases de données
- **Nouveauté 2024** : Modèle v2 approvisionné avec prévisibilité des coûts

**Nouveautés 2024 - Fonctionnalités Avancées**
- **Chiffrement en transit NFS** : Sécurité renforcée pour partages NFS 4.1
- **Mise en cache des métadonnées** : Réduction de latence, augmentation IOPS
- **Identités managées** : Authentification sécurisée sans clés partagées
- **Sauvegarde archivée** : Protection contre ransomwares, rétention jusqu'à 10 ans
- **Azure File Sync via Azure Arc** : Gestion simplifiée des agents de synchronisation

#### Capacités et Limites (Mise à jour 2024)
- **Standard** : Maximum 5 TB par share
- **Premium** : Maximum 100 TB par share (Standard) ou 256 TiB (v2 approvisionné)
- **Azure Import/Export** : Support Blob Storage et Azure Files
- **Nouveauté** : Support des identités managées pour Azure File Sync

### 2.3 Blob Storage

#### Types de Blobs

** Comprendre les 3 types de blobs Azure - Points critiques pour l'examen :**
                                         
**1. Block Blobs (Mise à jour 2024)**
- **Usage principal** : Stockage de fichiers standard (documents, images, vidéos, archives)
- **Structure** : Composés de blocs individuels (jusqu'à 50,000 blocs par blob)
- **Taille maximale** : 190.7 TiB (depuis version service 2019-12-12)
- **Taille de bloc** : Jusqu'à 4000 MiB par bloc (environ 4.19 GB)
- **Calcul** : 4000 MiB × 50,000 blocs = ~190.7 TiB
- **Optimisation** : Idéal pour streaming et accès aléatoire
- **Nouveautés 2024** : 
  - Support TLS 1.3 pour sécurité renforcée
  - Azure Storage Actions pour automatisation
  - Microsoft Defender pour Storage intégré
- **Cas d'usage typiques** :
  - Sites web statiques (HTML, CSS, JS, images)
  - Stockage de documents et médias
  - Sauvegardes et archives
  - Distribution de contenu (CDN)
- **Note importante** : Les limites dépendent de la version du service Azure utilisée

**2. Page Blobs**
- **Usage principal** : Disques de machines virtuelles Azure (VHD/VHDX)
- **Structure** : Pages de 512 octets, accès aléatoire optimisé
- **Taille maximale** : 8 TB par page blob
- **Performance** : Optimisé pour opérations de lecture/écriture aléatoires fréquentes
- **Cas d'usage typiques** :
  - Disques OS et disques de données des VMs
  - Bases de données nécessitant accès aléatoire
  - Applications nécessitant des performances I/O élevées
- ** Point clé** : Seul type de blob supportant les disques de VMs

**3. Append Blobs**
- **Usage principal** : Données ajoutées séquentiellement (logs, audit trails)
- **Structure** : Optimisé pour opérations d'ajout uniquement
- **Taille maximale** : 195 GB par append blob
- **Limitation** : Pas de modification des données existantes, ajout uniquement
- **Cas d'usage typiques** :
  - Fichiers de logs d'applications
  - Journaux d'audit et de sécurité
  - Streaming de données en temps réel
  - Données IoT collectées en continu

** Matrice de décision rapide :**

| Besoin | Type de Blob | Raison |
|--------|--------------|--------|
| Stocker des images/vidéos | **Block Blob** | Accès aléatoire, streaming optimisé |
| Disque de VM | **Page Blob** | Seul type supporté pour VHD |
| Logs d'application | **Append Blob** | Ajout séquentiel optimisé |
| Site web statique | **Block Blob** | Hébergement web, CDN |
| Base de données | **Page Blob** | Accès aléatoire haute performance |
| Données IoT | **Append Blob** | Collecte continue, ajout uniquement |

** Erreurs fréquentes identifiées :**
- ** Erreur** : Utiliser Append Blobs pour des fichiers modifiables
- ** Correct** : Block Blobs pour fichiers modifiables, Append Blobs pour ajout uniquement
- ** Erreur** : Essayer d'utiliser Block Blobs pour disques de VMs
- ** Correct** : Page Blobs obligatoires pour tous les disques de VMs

#### Lifecycle Management
- **Règles automatiques** : Transition entre niveaux
- **Suppression automatique** : Basée sur l'âge
- **Conditions** : Dernière modification, dernière accès, création

### 2.4 Azure Data Lake Storage Gen2

#### Vue d'ensemble et Concepts Fondamentaux

**Azure Data Lake Storage Gen2** est une solution de stockage optimisée pour l'analyse de données massives (Big Data)

**Caractéristiques principales :**
- **Basé sur Blob Storage** : Construit sur Azure Blob Storage avec fonctionnalités additionnelles
- **Hierarchical Namespace (HNS)** : Organisation hiérarchique des fichiers et répertoires
- **Haute performance** : Optimisé pour analytics et traitement parallèle
- **Compatibilité Hadoop** : Support natif des systèmes de fichiers distribués
- **Sécurité granulaire** : ACLs POSIX au niveau fichier/répertoire

#### Hierarchical Namespace - Concept Clé

**Hierarchical Namespace (Espace de noms hiérarchique)**

**Qu'est-ce que c'est ?**
- **Organisation** : Structure de répertoires et fichiers comme un système de fichiers traditionnel
- **Activation** : Doit être activé lors de la création du compte de stockage
- **Irréversible** : Une fois activé, ne peut pas être désactivé.
- **Impact** : Change fondamentalement la façon dont les données sont organisées

**Comparaison Blob Storage vs Data Lake Storage Gen2 :**

**Blob Storage (Sans HNS)**
```
container/
├── folder1-file1.txt
├── folder1-file2.txt
└── folder2-subfolder1-file3.txt
```
- **Structure** : Flat namespace (plat)
- **Organisation** : Tout est stocké au même niveau
- **Séparateurs** : Les "/" dans les noms sont des caractères, pas de vrais dossiers
- **Performance** : Opérations sur répertoires = opérations sur tous les blobs

**Data Lake Storage Gen2 (Avec HNS)**
```
container/
├── folder1/
│   ├── file1.txt
│   └── file2.txt
└── folder2/
    └── subfolder1/
        └── file3.txt
```
- **Structure** : Hierarchical namespace (hiérarchique)
- **Organisation** : Vrais répertoires et sous-répertoires
- **Opérations** : Rename/delete de dossiers = opération atomique
- **Performance** : Opérations sur répertoires = instantanées

#### Avantages de Hierarchical Namespace

**Performance :**
- **Renommage de dossier** : Opération de métadonnées uniquement (instantané)
- **Suppression de dossier** : Opération atomique unique
- **Requêtes** : Filtrage de répertoires plus rapide
- **Analytics** : Traitement parallèle optimisé

**Gestion :**
- **Organisation intuitive** : Structure familière aux utilisateurs
- **Navigation** : Exploration de données simplifiée
- **Maintenance** : Gestion de grandes quantités de données facilitée

**Sécurité :**
- **ACLs POSIX** : Permissions au niveau fichier/répertoire
- **Héritage** : Permissions héritées des dossiers parents
- **Granularité** : Contrôle d'accès précis

#### Différences Clés : Blob Storage vs Data Lake Storage Gen2

**Matrice comparative complète :**

| Caractéristique | Blob Storage | Data Lake Storage Gen2 |
|-----------------|--------------|------------------------|
| **Namespace** | Flat (plat) | Hierarchical (hiérarchique) |
| **Structure** | Container → Blobs | Container → Directories → Files |
| **Renommage dossier** | Opération coûteuse | Opération atomique |
| **ACLs POSIX** | Non supportées | Supportées |
| **Hadoop compatibility** | Limitée | Native |
| **Analytics performance** | Bonne | Excellente |
| **Use case principal** | Stockage général | Big Data analytics |
| **Prix** | Standard | Standard + coût HNS |
| **Protocoles** | REST, NFS 3.0 | REST, NFS 3.0, ABFS |

#### Activation de Hierarchical Namespace

**Processus de création :**
1. **Créer un compte de stockage** : StorageV2 (General Purpose v2)
2. **Advanced settings** : Activer "Hierarchical namespace"
3. **Validation** : Compte devient Data Lake Storage Gen2
4. **Impact** : Activation irréversible

** Point d'attention critique identifié :**
- **Irréversibilité** : HNS ne peut pas être désactivé après activation
- **Migration** : Migrer les données existantes vers nouveau compte si besoin
- **Planification** : Décider en amont si HNS est nécessaire

#### Sécurité et Contrôle d'Accès

**Access Control Lists (ACLs) POSIX**

**Permissions supportées :**
- **Read (r)** : Lecture de fichiers, liste de répertoires
- **Write (w)** : Modification de fichiers, création dans répertoires
- **Execute (x)** : Traversée de répertoires

**Types d'ACLs :**
- **Access ACLs** : Contrôlent l'accès aux fichiers/répertoires
- **Default ACLs** : Template pour nouveaux enfants (répertoires uniquement)

**Niveaux d'application :**
- **User** : Permissions pour utilisateur spécifique
- **Group** : Permissions pour groupe spécifique
- **Other** : Permissions pour tous les autres
- **Mask** : Limite les permissions maximales

**Exemple de configuration :**
```
user::rwx               # Propriétaire a tous les droits
user:john:r-x          # John peut lire et traverser
group::r-x             # Groupe propriétaire peut lire
group:analysts:rwx     # Groupe analysts a tous les droits
mask::rwx              # Masque maximal
other::---             # Autres n'ont aucun droit
```

#### Méthodes d'Authentification

**Azure Active Directory (Recommandé)**
- **OAuth 2.0** : Authentification moderne
- **Managed Identities** : Authentification sans secrets
- **RBAC + ACLs** : Contrôle d'accès à deux niveaux
- **Audit** : Traçabilité complète

**Shared Key (Déconseillé en production)**
- **Clés d'accès** : Accès complet au compte
- **Risque** : Compromission = accès total
- **Usage** : Développement uniquement

**Shared Access Signature (SAS)**
- **Accès délégué** : Limité dans le temps
- **Granularité** : Permissions spécifiques
- **Usage** : Accès temporaire externe

#### Rôles RBAC pour Data Lake Storage Gen2

**Storage Blob Data Owner**
- **Accès complet** : Toutes les données + gestion des ACLs
- **Super user** : Bypass les ACLs POSIX
- **Usage** : Administrateurs de données

**Storage Blob Data Contributor**
- **Lecture/écriture** : Tous les blobs et fichiers
- **Limitation** : Ne peut pas modifier les ACLs
- **Usage** : Applications et utilisateurs nécessitant accès complet aux données

**Storage Blob Data Reader**
- **Lecture seule** : Tous les blobs et fichiers
- **Usage** : Utilisateurs nécessitant accès lecture uniquement

** Stratégie de sécurité identifiée :**
1. **RBAC au niveau compte** : Contrôle d'accès global
2. **ACLs au niveau fichier/répertoire** : Contrôle d'accès granulaire
3. **Combinaison** : RBAC + ACLs pour sécurité maximale
4. **Principe** : Le plus restrictif entre RBAC et ACLs s'applique

#### Intégration avec Services Analytics Azure

**Azure Synapse Analytics**
- **Data warehousing** : Analyse de données massives
- **PolyBase** : Requêtes SQL sur Data Lake
- **Pipelines** : ETL/ELT intégrés
- **Performance** : Optimisé pour analytics

**Azure Databricks**
- **Apache Spark** : Traitement distribué
- **Delta Lake** : Couche ACID sur Data Lake
- **Machine Learning** : Pipelines ML/AI
- **Performance** : Scaling automatique

**Azure HDInsight**
- **Hadoop ecosystem** : Hive, Spark, HBase
- **Compatibilité** : Compatibilité native HDFS
- **Processing** : Batch et streaming
- **Clusters** : Gestion de clusters Hadoop

**Azure Data Factory**
- **ETL/ELT** : Pipelines d'ingestion de données
- **Connecteurs** : 90+ connecteurs de sources
- **Orchestration** : Workflows automatisés
- **Monitoring** : Surveillance intégrée

#### Protocoles et APIs

**ABFS (Azure Blob File System)**
- **Driver Hadoop** : abfs:// ou abfss:// (secure)
- **Performance** : Optimisé pour analytics
- **Usage** : HDInsight, Databricks, Synapse
- **Recommandé** : Pour tous les workloads Big Data

**REST API**
- **Blob Service API** : Compatible Blob Storage
- **Data Lake Storage API** : Opérations HNS spécifiques
- **Usage** : Applications custom
- **Compatibilité** : Backward compatible

**NFS 3.0 (Premium uniquement)**
- **Linux native** : Montage direct
- **Performance** : Faible latence
- **Limitation** : Premium tiers uniquement
- **Usage** : Workloads Linux haute performance

#### Cas d'Usage et Scénarios

**Big Data Analytics**
- **Data Lake** : Centralisation de toutes les données
- **Structure** : Raw → Curated → Enriched zones
- **Processing** : Spark, Hive, Presto
- **Avantage** : Scale illimité, coût optimisé

**Machine Learning et IA**
- **Training data** : Stockage de datasets d'entraînement
- **Feature store** : Partage de features entre modèles
- **Model registry** : Versioning de modèles
- **Avantage** : Performance pour larges datasets

**Data Warehousing**
- **External tables** : Requêtes SQL sur Data Lake
- **Data archival** : Archivage de données anciennes
- **Tiering** : Hot/Cool/Archive pour optimisation coûts
- **Avantage** : Séparation compute et storage

**IoT et Streaming**
- **Time-series data** : Stockage de données IoT
- **Event capture** : Archive Event Hubs/IoT Hub
- **Lambda architecture** : Batch + streaming layers
- **Avantage** : Ingestion haute vitesse, stockage illimité

#### Organisation des Données - Best Practices

**Structure en zones (Medallion Architecture) :**

**Bronze Layer (Raw)**
```
/bronze/
├── source1/
│   └── 2024/10/23/data.parquet
└── source2/
    └── 2024/10/23/data.json
```
- **Données brutes** : Format original, non transformées
- **Partitionnement** : Par date, source, type
- **Rétention** : Court/moyen terme selon besoin

**Silver Layer (Curated)**
```
/silver/
├── cleaned_data/
│   └── year=2024/month=10/day=23/
└── validated_data/
    └── year=2024/month=10/day=23/
```
- **Données nettoyées** : Validation, déduplication
- **Format optimisé** : Parquet, Delta Lake
- **Partitionnement** : Optimisé pour requêtes

**Gold Layer (Enriched)**
```
/gold/
├── aggregated/
│   └── monthly_summary/
└── ml_features/
    └── feature_set_v1/
```
- **Données business** : Agrégées, enrichies
- **Prêtes pour analytics** : Requêtes directes
- **Performance** : Indexation, caching

** Points clés identifiés :**
- **Séparation des couches** : Isolation des étapes de transformation
- **Gouvernance** : ACLs différentes par couche
- **Performance** : Optimisation par use case
- **Coûts** : Lifecycle policies par couche

#### Performance et Optimisation

**Partitionnement des données :**
- **Stratégie** : Partitionner par colonnes fréquemment filtrées (date, région, type)
- **Granularité** : Éviter trop de petites partitions (< 100 MB)
- **Exemple** : `/data/year=2024/month=10/day=23/`
- **Avantage** : Pruning de partitions, scan réduit

**Formats de fichiers :**
- **Parquet** : Format colonnaire, compression efficace (recommandé)
- **ORC** : Alternative à Parquet, optimisé pour Hive
- **Avro** : Format row-based, schéma évolutif
- **JSON/CSV** : Éviter pour production (performances faibles)

**Taille des fichiers :**
- **Optimal** : 128 MB - 1 GB par fichier
- **Éviter** : Millions de petits fichiers (small files problem)
- **Solution** : Compaction régulière avec Databricks ou ADF

**Indexation et caching :**
- **Delta Lake** : Z-ordering, data skipping
- **Synapse** : Résult sets caching
- **Databricks** : Delta cache, disk cache

#### Lifecycle Management et Coûts

**Access Tiers :**
- **Hot** : Données fréquemment accédées (bronze, silver actif)
- **Cool** : Données occasionnelles (> 30 jours, silver archive)
- **Archive** : Données rarement accédées (> 180 jours, compliance)

**Lifecycle Policies :**
```json
{
  "rules": [
    {
      "name": "MoveToCool",
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["/bronze/"]
        },
        "actions": {
          "baseBlob": {
            "tierToCool": {"daysAfterModificationGreaterThan": 30}
          }
        }
      }
    }
  ]
}
```

**Stratégies d'optimisation des coûts :**
- **Compression** : Parquet avec Snappy/ZSTD
- **Tiering automatique** : Lifecycle policies
- **Cleanup** : Suppression de données obsolètes
- **Monitoring** : Azure Cost Management

#### Erreurs Fréquentes et Pièges

** Erreur 1 : HNS non activé pour Big Data **
- **Symptôme** : Performances dégradées pour analytics
- **Cause** : Compte Blob Storage standard utilisé
- **Solution** : Créer nouveau compte avec HNS activé
- **Prévention** : Toujours activer HNS pour workloads analytics

** Erreur 2 : Confusion RBAC et ACLs **
- **Symptôme** : Utilisateurs ne peuvent pas accéder malgré RBAC
- **Cause** : ACLs POSIX bloquent l'accès
- **Solution** : Vérifier les deux niveaux (RBAC + ACLs)
- **Règle** : Le plus restrictif s'applique

** Erreur 3 : Millions de petits fichiers **
- **Symptôme** : Requêtes extrêmement lentes
- **Cause** : Small files problem (fichiers < 1 MB)
- **Solution** : Compaction avec Delta Lake ou ADF
- **Prévention** : Configurer batch size d'ingestion (128 MB+)

** Erreur 4 : Mauvais partitionnement **
- **Symptôme** : Scans complets malgré filtres
- **Cause** : Partitionnement non aligné avec requêtes
- **Solution** : Re-partitionner selon colonnes filtrées
- **Exemple** : Partitionner par date si filtres par date

** Erreur 5 : Format JSON/CSV en production **
- **Symptôme** : Coûts élevés, performances faibles
- **Cause** : Formats non optimisés pour analytics
- **Solution** : Convertir en Parquet
- **Gain** : 5-10x compression, 10-100x performance

#### Monitoring et Diagnostics

**Métriques Azure Monitor :**
- **Transactions** : Nombre de requêtes
- **Ingress/Egress** : Données entrantes/sortantes
- **Success rate** : Taux de succès des opérations
- **Latency** : E2E latency, server latency

**Diagnostic Logs :**
- **StorageRead** : Opérations de lecture
- **StorageWrite** : Opérations d'écriture
- **StorageDelete** : Opérations de suppression
- **Analyse** : Log Analytics pour requêtes KQL

**Alertes recommandées :**
- **High latency** : Latence > seuil
- **Error rate** : Taux d'erreur élevé
- **Throttling** : Dépassement de limites
- **Cost spike** : Augmentation soudaine des coûts

#### Comparaison avec Autres Solutions

**Data Lake Storage Gen2 vs Gen1**
- **Gen2** : Basé sur Blob Storage, HNS, ACLs POSIX, recommandé
- **Gen1** : Service dédié, deprecated, migration vers Gen2 recommandée
- **Migration** : Outils de migration disponibles (AdlCopy, ADF)

**Data Lake Storage vs Azure Files**
- **Data Lake** : Big Data analytics, PB de données
- **Azure Files** : File shares, remplacement NAS/SAN
- **Protocoles** : ABFS vs SMB/NFS
- **Use case** : Analytics vs partage de fichiers

**Data Lake Storage vs SQL Database**
- **Data Lake** : Données non structurées/semi-structurées, schema-on-read
- **SQL Database** : Données structurées, schema-on-write
- **Scale** : Illimité vs limité (4 TB - 100 TB)
- **Coût** : Très faible vs élevé

#### Points Critiques pour l'Examen AZ-104

✅ **Hierarchical Namespace** : Irréversible, requis pour analytics
✅ **ACLs POSIX** : Permissions granulaires fichier/répertoire
✅ **RBAC + ACLs** : Deux couches de sécurité, plus restrictif s'applique
✅ **ABFS protocol** : Protocole optimisé pour Hadoop/Spark
✅ **Storage Blob Data Owner** : Seul rôle permettant modification ACLs
✅ **Partitionnement** : Clé pour performance analytics
✅ **Parquet format** : Format recommandé pour Big Data
✅ **Lifecycle policies** : Optimisation automatique des coûts
✅ **Integration** : Synapse, Databricks, HDInsight, Data Factory

### 2.5 Data Transfer Solutions (Mise à jour 2024)

#### Azure Import/Export Service
** Destinations supportées identifiées :**
- **Azure Blob Storage**
- **Azure Files** (max 5 TB)
- SQL Database, autres services

**Process :**
1. Préparer les disques (BitLocker pour Windows)
2. Créer le job Import/Export
3. Expédier vers datacenter Azure
4. Azure transfert les données

#### Outils de Transfert (Mise à jour 2024)

**AzCopy v10+ (Recommandé)**
- **Fonctionnalités** : Transfert haute performance, résilience
- **Support** : Blob, Files, Tables, Queues
- **Nouveautés 2024** : Support des identités managées, chiffrement en transit
- **Usage** : Scripts automatisés, migrations à grande échelle

**Azure Storage Explorer**
- **Interface** : Graphique intuitive
- **Fonctionnalités** : Gestion complète des comptes de stockage
- **Nouveautés 2024** : Support des identités managées, Azure AD B2B
- **Usage** : Administration manuelle, exploration des données

**Azure Data Box Family**
- **Data Box** : 40 TB - 80 TB (disques)
- **Data Box Heavy** : 1 PB (appliance)
- **Data Box Edge** : Computing edge + transfert
- **Nouveautés 2024** : Support des identités managées, chiffrement renforcé

**Azure Storage Actions (Nouveauté 2024)**
- **Fonctionnalité** : Automatisation de la gestion des données
- **Support** : Blob Storage, Data Lake Storage
- **Usage** : Workflows automatisés, transformations de données
- **Avantage** : Plateforme entièrement gérée pour l'automatisation

#### Storage Account Roles et Permissions (Mise à jour 2024)

** Rôles de gestion des comptes de stockage :**

**Storage Account Contributor**
- **Gestion complète** des comptes de stockage
- **Accès aux clés** de compte (Shared Key authorization)
- **Permissions** : Lecture, écriture, suppression des comptes de stockage
- **Usage** : Administration des comptes de stockage

**Storage Blob Data Contributor**
- **Permissions sur les données** : Lecture, écriture, suppression
- **Scope** : Containers et blobs Azure Storage
- **Usage** : Accès aux données blob sans gestion du compte

**Reader**
- **Lecture seule** : Visualisation de toutes les ressources
- **Aucune modification** : Pas de changements autorisés
- **Usage** : Monitoring et audit

**Owner**
- **Accès complet** : Gestion de toutes les ressources
- **Délégation d'accès** : Possibilité d'assigner des rôles
- **Usage** : Administration complète

** Différenciation clé identifiée :**
- **Storage Account Contributor** : Gestion du compte + accès aux clés
- **Storage Blob Data Contributor** : Accès aux données uniquement
- **Reader** : Visualisation sans modification
- **Owner** : Contrôle total + délégation d'accès

#### Sécurité et Conformité (Nouveautés 2024)

**Microsoft Defender pour Storage**
- **Protection** : Détection des menaces en temps réel
- **Alertes** : Activités suspectes, accès anormaux
- **Intégration** : Azure Security Center, Log Analytics
- **Usage** : Sécurité proactive des données de stockage

**Chiffrement et Sécurité Renforcée**
- **TLS 1.3** : Support pour Blob Storage (sécurité renforcée)
- **Chiffrement en transit NFS** : Protection des partages NFS 4.1
- **Clés gérées par le client** : Contrôle total du chiffrement
- **Identités managées** : Authentification sans clés partagées

**Conformité et Gouvernance**
- **Frontière européenne des données** : Contrôle de l'emplacement des données
- **Politiques Purview** : Classification et protection des données
- **Audit et monitoring** : Traçabilité complète des accès
- **RGPD compliance** : Conformité réglementaire européenne

**Nouveautés Sécurité 2024**
- **Azure Storage Actions** : Automatisation sécurisée des workflows
- **Sauvegarde archivée** : Protection contre ransomwares (10 ans de rétention)
- **Azure File Sync sécurisé** : Synchronisation avec identités managées
- **Monitoring avancé** : Métriques de sécurité et alertes intelligentes

---

## 3. Deploy and Manage Azure Compute Resources (20-25%)

### 3.1 Virtual Machines

#### VM Sizes et Families
- **General Purpose** (B, D, DC series) : Workloads équilibrés
- **Compute Optimized** (F series) : Applications CPU-intensive
- **Memory Optimized** (E, M, G series) : Bases de données, analytics
- **Storage Optimized** (L series) : Big data, SQL NoSQL
- **GPU** (N series) : IA, machine learning, rendering

#### Disques et Stockage

** Erreur fréquente identifiée :** Disques temporaires vs persistants

**Disque C: (OS Disk)**
- **Type** : Persistant (VHD dans Azure Storage)
- **Contenu** : OS, applications installées
- **Persistance** : Conservé lors redémarrages/arrêts
- **Usage** : Applications, données importantes

**Disque D: (Temporary Disk)**
- **Type** : Temporaire (SSD local hyperviseur)
- **Contenu** : Pagefile.sys par défaut
- **Persistance** : **PERDU** lors maintenances/redéploiements
- **Usage** : Cache, fichiers temporaires, TempDB

**Disques de Données (E:, F:, etc.)**
- **Type** : Persistants (Managed Disks)
- **Usage** : Données applicatives, bases de données

#### Types de Disques
- **Standard HDD** : Workloads peu fréquents, coût minimal
- **Standard SSD** : Workloads modérés, balance performance/coût
- **Premium SSD** : Workloads critiques, haute performance
- **Ultra Disk** : Workloads extrêmes, latence ultra-faible

#### Création de VM avec Data Disks

**⚠️ Erreur Courante QCM : Attacher plusieurs data disks dès la création**

**Scénario :** Créer une VM avec 3 disques de données (data disks) attachés dès la création.

**Solution : Azure CLI (Méthode la plus simple)**

```bash
# Créer une VM avec 3 data disks en une seule commande
az vm create \
  --resource-group myRG \
  --name myVM \
  --image UbuntuLTS \
  --size Standard_D2s_v3 \
  --admin-username azureuser \
  --generate-ssh-keys \
  --data-disk-sizes-gb 128 256 512
  # Créera automatiquement 3 disques: 128GB, 256GB et 512GB
```

**Paramètres clés :**
- `--data-disk-sizes-gb` : Liste des tailles de disques séparées par espaces
- Les disques sont créés automatiquement et attachés
- LUN (Logical Unit Number) assignés automatiquement : 0, 1, 2, etc.

**Concepts Importants :**

**LUN (Logical Unit Number) :**
- **Identifiant unique** pour chaque disque attaché à une VM
- **Numérotation** : Commence à 0 (LUN 0, LUN 1, LUN 2, etc.)
- **Maximum** : Dépend de la taille de VM
  - Standard_D2s_v3 : Max 8 data disks (LUN 0-7)
  - Standard_D16s_v3 : Max 32 data disks (LUN 0-31)

**Persistance des Data Disks :**
- ✅ **PERSISTANTS** : Les données sont conservées lors arrêts/redémarrages
- ✅ **Managed Disks** : Gestion automatique par Azure
- ✅ **Snapshots** : Possibilité de créer des snapshots pour backup
- ❌ **Différent du disque temporaire (D:)** qui est volatile

**Scénarios d'Examen :**

| Question | Réponse | Raison |
|----------|---------|--------|
| Créer VM avec 3 disques de données | `--data-disk-sizes-gb 128 256 512` | Azure CLI option la plus simple |
| Maximum de data disks sur Standard_D2s_v3 | 8 disks | Dépend de la taille de VM |
| Les data disks sont-ils persistants ? | Oui | Contrairement au disque temporaire D: |
| Identifier un data disk dans la VM | Par LUN (0, 1, 2...) | Logical Unit Number |

#### Accès Externe aux VMs

**⚠️ Erreur Courante QCM : Minimize Administrative Effort**

**Scénario :** Une VM est accessible uniquement depuis le réseau interne. Vous devez permettre l'accès depuis Internet avec **effort administratif minimal**.

**❌ FAUX :** Configurer un VPN Site-to-Site
**✅ CORRECT :** **Ajouter une adresse IP publique** à la VM

**Comparaison des Solutions :**

| Solution | Complexité | Coût | Temps | Effort Admin |
|----------|------------|------|-------|--------------|
| **IP Publique** | ✅ Très faible | ~$3/mois | 2 min | ✅ Minimal |
| **Azure Bastion** | Moyenne | ~$150/mois | 15 min | Moyen |
| **VPN Site-to-Site** | ❌ Élevée | ~$25-150/mois | 1-2h | ❌ Élevé |

**Solution Recommandée - Ajouter une IP Publique :**

```bash
# 1. Créer une IP publique
az network public-ip create \
  --resource-group myRG \
  --name myVM-PublicIP \
  --sku Standard \
  --allocation-method Static

# 2. Associer l'IP publique à la NIC de la VM
az network nic ip-config update \
  --resource-group myRG \
  --nic-name myVM-NIC \
  --name ipconfig1 \
  --public-ip-address myVM-PublicIP
```

**⚠️ Sécurité - NSG Recommandé :**

```bash
# Limiter l'accès SSH/RDP à votre IP uniquement
az network nsg rule create \
  --resource-group myRG \
  --nsg-name myVM-NSG \
  --name Allow-SSH-MyIP \
  --priority 100 \
  --source-address-prefixes 203.0.113.50/32 \
  --destination-port-ranges 22 \
  --access Allow \
  --protocol Tcp
```

**Pourquoi PAS VPN Site-to-Site ?**
- ❌ **Complexe** : Local Network Gateway, Virtual Network Gateway, Connection
- ❌ **Temps** : 1-2 heures de configuration
- ❌ **Coût** : Gateway ~$25-150/mois
- ⚠️ **Use case** : Connectivité réseau entier, pas une seule VM

**Scénarios d'Examen :**
- **"Minimal administrative effort"** → Add Public IP
- **"Secure access without exposing ports"** → Azure Bastion
- **"Connect entire on-premises network"** → VPN Site-to-Site

#### High Availability

**Availability Sets**

**Définition :**
- **Groupement logique** de VMs pour assurer la haute disponibilité
- **Protection** contre pannes matérielles ET maintenances planifiées
- **Placement** : VMs distribuées sur plusieurs racks physiques
- **Scope** : Même datacenter (single region, single datacenter)
- **SLA** : 99.95% (au moins 1 VM disponible)
- **Gratuit** : Aucun coût supplémentaire pour l'availability set lui-même

**Composants Clés :**

**1. Fault Domains (FD) - Domaines de Défaillance**
- **Définition** : Représente un rack physique dans le datacenter
- **Contenu** : Serveurs, switch réseau, source d'alimentation partagés
- **Maximum** : 3 Fault Domains par availability set
- **Protection** : Panne matérielle (hardware failure, power outage, network switch failure)
- **Distribution** : Azure répartit automatiquement vos VMs sur les FDs

**Visualisation des Fault Domains :**
```
Datacenter Azure
├── Rack 1 (FD 0) - Alimentation A, Switch A
│   ├── VM1
│   └── VM4
├── Rack 2 (FD 1) - Alimentation B, Switch B
│   ├── VM2
│   └── VM5
└── Rack 3 (FD 2) - Alimentation C, Switch C
    ├── VM3
    └── VM6
```

**Scénario de panne FD :**
- **Problème** : Rack 1 perd l'alimentation électrique
- **Impact** : Seules VM1 et VM4 sont affectées
- **Résultat** : VM2, VM3, VM5, VM6 continuent de fonctionner
- **Disponibilité** : Au moins 2/3 des VMs restent disponibles

**2. Update Domains (UD) - Domaines de Mise à Jour**
- **Définition** : Groupe logique de VMs redémarrées ensemble lors de maintenances
- **Par défaut** : 5 Update Domains (si non spécifié à la création)
- **Configurable** : De 1 à 20 Update Domains maximum
- **Important** : Configuration définie à la création, non modifiable après
- **Protection** : Maintenance planifiée Azure (host OS updates, hardware maintenance)
- **Processus** : Azure redémarre un seul UD à la fois
- **Délai** : 30 minutes entre chaque UD redémarré

**Visualisation des Update Domains :**
```
Update Domain 0: VM1, VM6
Update Domain 1: VM2, VM7
Update Domain 2: VM3, VM8
Update Domain 3: VM4, VM9
Update Domain 4: VM5, VM10

Maintenance planifiée :
Étape 1: Redémarre UD0 → VM1, VM6 offline, autres OK
[Attend 30 min]
Étape 2: Redémarre UD1 → VM2, VM7 offline, autres OK
[Attend 30 min]
Étape 3: Redémarre UD2 → VM3, VM8 offline, autres OK
...
```

**Scénario de maintenance UD :**
- **Événement** : Maintenance planifiée Azure
- **Processus** : Azure redémarre UD par UD (jamais simultanément)
- **Impact** : Maximum 1/5 (20%) des VMs offline à un moment donné
- **Garantie** : Au moins 80% de capacité maintenue

**3. Combinaison FD + UD - Protection Double**

**Matrice de Distribution (Exemple : 6 VMs, 3 FD, 3 UD) :**
```
              FD 0 (Rack 1)  FD 1 (Rack 2)  FD 2 (Rack 3)
UD 0            VM1             VM2             VM3
UD 1            VM4             VM5             VM6
UD 2            VM7             VM8             VM9
```

**Protection simultanée :**
- **Panne FD 1** : Perd VM2, VM5, VM8 → Garde VM1,3,4,6,7,9
- **Maintenance UD 0** : Redémarre VM1, VM2, VM3 → Garde VM4,5,6,7,8,9
- **Combiné** : Jamais plus d'un FD ET un UD affecté en même temps

**Configuration et Contraintes :**

**Création :**
- **Moment** : Doit être configuré AVANT ou PENDANT la création de la VM
- **Modification** : IMPOSSIBLE de changer l'availability set d'une VM existante
- **Solution** : Recréer la VM si changement nécessaire

**Limitations importantes :**
- **Zone géographique** : Limité à un seul datacenter
- **Maximum VMs** : Limite pratique (recommandé : < 200 VMs)
- **Famille de VM** : Toutes les VMs doivent être de tailles compatibles
- **Managed Disks** : Fortement recommandé (Aligned ou non)
- **Incompatibilité** : Ne peut PAS combiner avec Availability Zones

**Configuration PowerShell :**
```powershell
# Créer l'Availability Set
New-AzAvailabilitySet `
  -ResourceGroupName "myRG" `
  -Name "myAvailabilitySet" `
  -Location "East US" `
  -PlatformFaultDomainCount 3 `
  -PlatformUpdateDomainCount 5 `
  -Sku Aligned

# Créer VM dans l'Availability Set
New-AzVM `
  -ResourceGroupName "myRG" `
  -Name "myVM" `
  -AvailabilitySetName "myAvailabilitySet" `
  ...
```

**Configuration Azure CLI :**
```bash
# Créer l'Availability Set
az vm availability-set create \
  --resource-group myRG \
  --name myAvailabilitySet \
  --platform-fault-domain-count 3 \
  --platform-update-domain-count 5

# Créer VM dans l'Availability Set
az vm create \
  --resource-group myRG \
  --name myVM \
  --availability-set myAvailabilitySet \
  ...
```

**Best Practices :**

1. **Minimum 2 VMs par tier** : Pour bénéficier du SLA 99.95%
2. **Utiliser Managed Disks** : Meilleure résilience (Storage Fault Domains)
3. **Load Balancer** : Combiner avec Azure Load Balancer pour distribution trafic
4. **Même fonction** : Regrouper uniquement VMs ayant le même rôle
5. **Planification** : Configurer dès la création, impossible à modifier après

**Cas d'usage typiques :**
- **Web tier** : Serveurs web dans un availability set
- **App tier** : Serveurs applicatifs dans un autre availability set
- **Data tier** : Serveurs de base de données dans un troisième availability set
- **Séparation** : Un availability set différent par tier applicatif

**Comparaison rapide :**
| Caractéristique | Availability Sets | Availability Zones |
|-----------------|-------------------|-------------------|
| **Protection** | Rack failure | Datacenter failure |
| **Fault Domains** | Max 3 | 3 zones |
| **Scope** | Single datacenter | Multiple datacenters |
| **SLA** | 99.95% | 99.99% |
| **Latence** | Minimale | Légèrement plus élevée |
| **Coût** | Gratuit | Frais de transfert inter-zones |

**Availability Zones**
- **Protection** : Datacenters physiquement séparés
- **SLA** : 99.99%
- **Scope** : Région Azure

** Prérequis identifiés pour Availability Zones :**
- **Managed Disks** : Obligatoire
- **Availability Options** : Doit être configuré à la création

#### Accès Externe aux VMs
** Cas d'usage identifié :** Accès externe avec effort administratif minimal
- **Scénario** : VM interne accessible uniquement depuis le réseau interne, besoin d'accès externe
- **Solution optimale** : Ajouter une adresse IP publique à la VM
- **Avantage** : Configuration simple et directe, effort administratif minimal

** Erreur fréquente identifiée :** Complexité inutile pour accès externe simple
- ** Erreur** : Configurer un VPN Site-to-Site pour un accès externe simple
- ** Correct** : Utiliser une adresse IP publique pour minimiser l'effort administratif
- **Raison** : VPN S2S = Configuration complexe (Local Network Gateway, Connection, etc.)
- **Alternative** : IP publique = Configuration simple et directe

### 3.2 Virtual Machine Scale Sets (VMSS)

#### Concepts Clés
- **Scaling automatique** : Basé sur métriques ou planning
- **Load balancing** : Intégré avec Azure Load Balancer
- **Update management** : Rolling updates avec availability

** Cas d'usage identifié :** Maintenance Azure
- Pour garantir **8 VMs minimum** pendant maintenance
- Créer un **VMSS avec 10 instances**
- Azure maintient une partie, les autres restent disponibles

#### Scaling Policies
- **Manual** : Contrôle manuel du nombre d'instances
- **Automatic** : Basé sur CPU, mémoire, métriques custom
- **Scheduled** : Scaling programmé

### 3.3 App Services

#### Service Plans - Tiers Détaillés

**⚠️ Erreur Courante QCM : Capacités de scaling par tier**

**Comparaison Complète des Tiers :**

| Tier | SKUs | Max Instances | Autoscale | Deployment Slots | Custom Domain | SSL | Prix/mois |
|------|------|---------------|-----------|------------------|---------------|-----|-----------|
| **Free** | F1 | 1 (partagé) | ❌ Non | ❌ Non | ❌ Non | ❌ Non | Gratuit |
| **Shared** | D1 | 1 (partagé) | ❌ Non | ❌ Non | ✅ Oui | ❌ Non | ~$10 |
| **Basic** | B1-B3 | 3 | ❌ Non | ❌ Non | ✅ Oui | ✅ Oui | ~$55-220 |
| **Standard** | S1-S3 | 10 | ✅ Oui | ✅ 5 slots | ✅ Oui | ✅ Oui | ~$70-280 |
| **Premium** | P1v2-P3v2 | 30 | ✅ Oui | ✅ 20 slots | ✅ Oui | ✅ Oui | ~$150-600 |
| **PremiumV3** | P1v3-P3v3 | 30 | ✅ Oui | ✅ 20 slots | ✅ Oui | ✅ Oui | ~$200-800 |
| **Isolated** | I1-I3 | 100 | ✅ Oui | ✅ 20 slots | ✅ Oui | ✅ Oui | ~$650+ |

**Caractéristiques par Tier :**

**Free / Shared (Development) :**
- **Infrastructure** : Partagée avec autres clients
- **Limitations** : Quotas CPU (60 min/jour pour Free)
- **Downtime** : Possible si app inactive > 20 min
- **Use case** : Développement, POC uniquement
- ❌ **Production** : Non recommandé

**Basic (Small Production) :**
- **Infrastructure** : VMs dédiées
- **Scaling** : Manuel uniquement (1-3 instances)
- **Limitations** : Pas d'autoscale, pas de slots
- **Use case** : Petites apps production, budget limité
- **SLA** : 99.95%

**Standard (Production) :**
- **Infrastructure** : VMs dédiées
- **Scaling** : Manuel + Autoscale (1-10 instances)
- **Features** : Deployment slots (5), Traffic Manager
- **Use case** : Apps production standard
- **SLA** : 99.95%

**Premium (High Performance) :**
- **Infrastructure** : VMs dédiées plus puissantes
- **Scaling** : Manuel + Autoscale (1-30 instances)
- **Features** : Deployment slots (20), VNet integration
- **Use case** : Apps critiques, haute performance
- **SLA** : 99.95%

**Isolated (Enterprise) :**
- **Infrastructure** : App Service Environment (ASE) dédié
- **Isolation** : Réseau isolé, pas de partage
- **Scaling** : Jusqu'à 100 instances
- **Use case** : Conformité, sécurité maximale
- **SLA** : 99.95%

#### App Service Plan Scaling - Détaillé

**⚠️ Erreur Courante QCM : Scale Up vs Scale Out**

**Types de Scaling Disponibles :**

**1. Scale Up (Vertical Scaling) - Changer de Tier/SKU**

**Définition :** Augmenter les ressources (CPU, RAM, Storage) de chaque instance

**Quand utiliser :**
- Application nécessite plus de CPU/RAM
- Atteinte des limites de l'instance actuelle
- Besoin de nouvelles fonctionnalités (ex: deployment slots)

**Exemple - Azure CLI :**
```bash
# Upgrader de B1 vers S1
az appservice plan update \
  --name myAppServicePlan \
  --resource-group myRG \
  --sku S1

# Upgrader vers Premium
az appservice plan update \
  --name myAppServicePlan \
  --resource-group myRG \
  --sku P1V2
```

**Impact :**
- ⏱️ **Downtime** : Bref redémarrage (quelques secondes)
- 💰 **Coût** : Augmente selon le nouveau tier
- 🎯 **Performance** : Chaque instance plus puissante

**2. Scale Out (Horizontal Scaling) - Ajouter des Instances**

**Définition :** Augmenter le nombre d'instances (VMs) qui exécutent l'application

**Quand utiliser :**
- Trafic élevé, besoin de plus de capacité
- Haute disponibilité requise
- Distribution de charge

**Exemple - Azure CLI :**
```bash
# Scale Out manuel - Ajouter 5 instances
az appservice plan update \
  --name myAppServicePlan \
  --resource-group myRG \
  --number-of-workers 5
```

**Impact :**
- ⏱️ **Downtime** : ✅ Aucun
- 💰 **Coût** : Proportionnel au nombre d'instances
- 🎯 **Performance** : Capacité totale multipliée

**3. Autoscale (Automatic Horizontal Scaling)**

**Disponible à partir de :** Standard (S1) et supérieur

**Configuration Autoscale :**
```bash
# Créer une règle d'autoscale
az monitor autoscale create \
  --resource-group myRG \
  --resource myAppServicePlan \
  --resource-type Microsoft.Web/serverFarms \
  --name myAutoscaleRule \
  --min-count 2 \
  --max-count 10 \
  --count 2

# Règle basée sur CPU (Scale Out)
az monitor autoscale rule create \
  --resource-group myRG \
  --autoscale-name myAutoscaleRule \
  --condition "CpuPercentage > 75 avg 5m" \
  --scale out 1

# Règle basée sur CPU (Scale In)
az monitor autoscale rule create \
  --resource-group myRG \
  --autoscale-name myAutoscaleRule \
  --condition "CpuPercentage < 25 avg 5m" \
  --scale in 1
```

**Métriques disponibles pour Autoscale :**
- **CPU Percentage** : % utilisation CPU
- **Memory Percentage** : % utilisation RAM
- **Disk Queue Length** : File d'attente disque
- **Http Queue Length** : File d'attente requêtes HTTP
- **Data In/Out** : Bande passante réseau
- **Custom metrics** : Application Insights

**Configuration via Portal :**
```
App Service Plan → Scale out (App Service plan) → Autoscale
→ Add a rule → Metric (CPU Percentage > 70)
→ Operation (Increase count by 1)
→ Cool down (5 minutes)
```

**Best Practices Autoscale :**
- ✅ **Minimum 2 instances** pour haute disponibilité
- ✅ **Cool down period** : 5-10 minutes (éviter flapping)
- ✅ **Margins** : Scale out à 70%, scale in à 30%
- ⚠️ **Limites** : Définir max instances pour contrôler coûts

**Matrice de Décision - Scaling :**

| Symptôme | Solution | Commande |
|----------|----------|----------|
| **CPU/RAM saturé** sur instances | Scale Up | Change SKU (B1→S1) |
| **Trafic élevé**, instances saturées | Scale Out | Augmenter instances |
| **Trafic variable** (pics) | Autoscale | Règles basées métriques |
| **Besoin de deployment slots** | Scale Up | Upgrade vers S1+ |
| **Latence élevée** | Scale Out | Plus d'instances |
| **Coûts trop élevés** | Scale Down/In | Réduire tier ou instances |

**Limitations par Tier :**

| Tier | Scale Out Manuel | Autoscale | Max Instances |
|------|-----------------|-----------|---------------|
| Free/Shared | ❌ Non | ❌ Non | 1 (partagé) |
| Basic | ✅ Oui | ❌ Non | 3 |
| Standard | ✅ Oui | ✅ Oui | 10 |
| Premium | ✅ Oui | ✅ Oui | 30 |
| Isolated | ✅ Oui | ✅ Oui | 100 |

**Scénarios d'Examen :**

| Question | Réponse | Raison |
|----------|---------|--------|
| App nécessite plus de RAM | **Scale Up** | Augmenter ressources par instance |
| Trafic web élevé, pic de charge | **Scale Out** | Ajouter instances pour capacité |
| Besoin deployment slots | **Scale Up to Standard** | Slots disponibles à partir de S1 |
| Trafic variable jour/nuit | **Autoscale** | Ajustement automatique |
| Minimiser downtime pendant scaling | **Scale Out** | Pas de redémarrage |

**PowerShell - Gestion Complète :**
```powershell
# Scale Up
Set-AzAppServicePlan `
  -ResourceGroupName "myRG" `
  -Name "myAppServicePlan" `
  -Tier "Standard" `
  -WorkerSize "Medium"

# Scale Out
Set-AzAppServicePlan `
  -ResourceGroupName "myRG" `
  -Name "myAppServicePlan" `
  -NumberofWorkers 5

# Get current scale
Get-AzAppServicePlan `
  -ResourceGroupName "myRG" `
  -Name "myAppServicePlan" | 
  Select-Object Name, Sku, NumberOfWorkers
```

#### Runtime Stacks et OS
** Point clé identifié :** Un App Service = Un runtime
- Chaque App Service configuré pour **un seul runtime**
- Python OU Java OU C# - pas de mix possible
- Un Plan App Service peut héberger plusieurs App Services du même OS

#### Deployment Slots

** Workflow optimal identifié :**
1. **Deploy** → Déployer l'update sur staging slot
2. **Test** → Tester l'application sur staging
3. **Swap** → Basculer staging vers production

**Avantages du Slot Swapping :**
- **Zero downtime** : Aucune interruption
- **Warm-up automatique** : Instances préchauffées
- **Rollback instant** : Re-swap pour annuler
- **Configuration preservation** : Settings spécifiques aux slots

#### Authentication et Authorization
** Configuration identifiée :** Désactiver l'accès anonyme
- Configurer **Authentication** dans App Service
- Ajouter identity providers : Microsoft, Google, Facebook, Twitter
- **Anonymous access** est une méthode d'authentification

### 3.4 Azure Container Instances (ACI)

#### Caractéristiques
- **Serverless containers** : Pas de gestion d'infrastructure
- **Billing per second** : Facturation à la seconde
- **Quick start** : Démarrage en secondes
- **Support** : Linux et Windows containers

#### Use Cases
- **Burst workloads** : Scaling rapide
- **Build agents** : CI/CD pipelines
- **Data processing** : Jobs batch
- **Development/testing** : Environnements temporaires

### 3.5 ARM Templates

#### Structure de Base
```json
{
    "$schema": "...",
    "contentVersion": "1.0.0.0",
    "parameters": {},
    "variables": {},
    "resources": [],
    "outputs": {}
}
```

** Point identifié :** Déploiement depuis template
- **Resource Group** : Seul paramètre configurable lors du déploiement
- Toutes autres configurations définies dans le template
- Template = Infrastructure as Code

---

## 4. Configure and Manage Virtual Networking (25-30%)

### 4.1 Virtual Networks (VNet)

#### Concepts Fondamentaux
- **Address Space** : Plage CIDR privée 10.0.0.0/8 (16M d'adresses), 172.16.0.0/12 (1M d'adresses), 192.168.0.0/16 (65k adresses)
- **Subnets** : Subdivision du VNet
- **Network Security Groups** : Firewalls au niveau subnet/NIC

#### VNet Peering - Configuration Détaillée

**⚠️ Erreur Courante QCM : Configuration bidirectionnelle obligatoire**

**Vue d'ensemble :**
VNet Peering établit une connexion privée entre deux réseaux virtuels Azure, permettant aux ressources de communiquer comme si elles étaient sur le même réseau.

**Types de Peering :**

| Type | Portée | Latence | Coût | Use Case |
|------|--------|---------|------|----------|
| **Regional VNet Peering** | Même région Azure | Ultra-faible | Gratuit (ingress) | Apps multi-tier, partage de ressources |
| **Global VNet Peering** | Régions différentes | Faible | Facturé (ingress + egress) | Multi-région, DR, geo-distribution |

**Caractéristiques Clés :**
- **Traffic** : 100% privé, transit par backbone Azure (pas d'Internet)
- **Bande passante** : Identique à celle d'un VNet unique
- **Latence** : Minimale, équivalente à un VNet local
- **Sécurité** : Trafic chiffré par défaut sur le réseau Azure
- **Gateway Transit** : Partage de VPN/ExpressRoute Gateways possible

**Configuration Obligatoire - Bidirectionnelle**

**⚠️ POINT CRITIQUE pour l'EXAMEN :**
- **Peering = 2 opérations distinctes**
- VNet1 → VNet2 (créer peering depuis VNet1)
- VNet2 → VNet1 (créer peering depuis VNet2)
- **Les deux doivent être configurés** pour établir la communication

**Visualisation :**
```
VNet1 (10.0.0.0/16)          VNet2 (172.16.0.0/16)
       │                              │
       │ ─────── Peering 1 ────────→ │
       │                              │
       │ ←────── Peering 2 ───────── │
       │                              │
    ✅ Communication établie
```

**Sans configuration bidirectionnelle :**
```
VNet1 (10.0.0.0/16)          VNet2 (172.16.0.0/16)
       │                              │
       │ ─────── Peering 1 ────────→ │
       │                              │
       │         (pas de retour)      │
       │                              │
    ❌ Communication IMPOSSIBLE
```

**1. Règle d'Or : Plages d'adresses non-chevauchantes**

**⚠️ Prérequis OBLIGATOIRE :**
- **Principe** : Les plages d'adresses (CIDR) des deux VNets ne peuvent PAS se chevaucher
- **Vérification Azure** : Azure refuse automatiquement le peering si chevauchement détecté

**Exemples de Compatibilité :**

| VNet1 Address Space | VNet2 Address Space | Peering Possible ? | Raison |
|---------------------|---------------------|-------------------|--------|
| 10.0.0.0/16 | 172.16.0.0/16 | ✅ **OUI** | Plages complètement différentes |
| 10.0.0.0/16 | 10.1.0.0/16 | ✅ **OUI** | Pas de chevauchement |
| 192.168.0.0/24 | 192.168.1.0/24 | ✅ **OUI** | Sous-réseaux différents |
| 192.168.0.0/24 | 192.168.0.0/16 | ❌ **NON** | /24 inclus dans /16 |
| 10.0.0.0/16 | 10.0.0.0/24 | ❌ **NON** | /24 inclus dans /16 |
| 172.16.0.0/16 | 172.16.0.0/12 | ❌ **NON** | /16 inclus dans /12 |

**Erreur Fréquente en Examen :**
- **Question** : VNet1 (192.168.0.0/24) peut-il être peeré avec VNet3 (192.168.0.0/16) ?
- **Réponse correcte** : ❌ **NON**
- **Raison** : La plage 192.168.0.0/24 est **entièrement incluse** dans 192.168.0.0/16
- **Solution** : Utiliser des plages complètement distinctes (ex: 10.0.0.0/16 et 172.16.0.0/16)

**2. Configuration via Azure CLI - Étape par Étape**

**Étape 1 : Obtenir les Resource IDs**
```bash
# VNet1 ID
vnet1Id=$(az network vnet show \
  --resource-group RG1 \
  --name VNet1 \
  --query id --out tsv)

# VNet2 ID
vnet2Id=$(az network vnet show \
  --resource-group RG2 \
  --name VNet2 \
  --query id --out tsv)
```

**Étape 2 : Créer Peering VNet1 → VNet2**
```bash
az network vnet peering create \
  --name VNet1-to-VNet2 \
  --resource-group RG1 \
  --vnet-name VNet1 \
  --remote-vnet $vnet2Id \
  --allow-vnet-access
```

**Étape 3 : Créer Peering VNet2 → VNet1 (Obligatoire !)**
```bash
az network vnet peering create \
  --name VNet2-to-VNet1 \
  --resource-group RG2 \
  --vnet-name VNet2 \
  --remote-vnet $vnet1Id \
  --allow-vnet-access
```

**Étape 4 : Vérifier l'état du Peering**
```bash
# Vérifier depuis VNet1
az network vnet peering show \
  --resource-group RG1 \
  --vnet-name VNet1 \
  --name VNet1-to-VNet2 \
  --query peeringState

# Résultat attendu: "Connected"
```

**3. Configuration via PowerShell**

```powershell
# Créer Peering VNet1 → VNet2
Add-AzVirtualNetworkPeering `
  -Name "VNet1-to-VNet2" `
  -VirtualNetwork (Get-AzVirtualNetwork -Name "VNet1" -ResourceGroupName "RG1") `
  -RemoteVirtualNetworkId "/subscriptions/{sub-id}/resourceGroups/RG2/providers/Microsoft.Network/virtualNetworks/VNet2"

# Créer Peering VNet2 → VNet1 (Obligatoire !)
Add-AzVirtualNetworkPeering `
  -Name "VNet2-to-VNet1" `
  -VirtualNetwork (Get-AzVirtualNetwork -Name "VNet2" -ResourceGroupName "RG2") `
  -RemoteVirtualNetworkId "/subscriptions/{sub-id}/resourceGroups/RG1/providers/Microsoft.Network/virtualNetworks/VNet1"

# Vérifier l'état
Get-AzVirtualNetworkPeering `
  -ResourceGroupName "RG1" `
  -VirtualNetworkName "VNet1" | 
  Select-Object Name, PeeringState
```

**4. Options de Configuration Avancées**

**Options Disponibles :**

| Option | Description | Use Case |
|--------|-------------|----------|
| **allow-vnet-access** | Autoriser trafic entre VNets | Par défaut, toujours activé |
| **allow-forwarded-traffic** | Autoriser trafic transitif via NVA | Hub-spoke avec firewall |
| **allow-gateway-transit** | Partager VPN/ExpressRoute Gateway | VNet hub avec gateway |
| **use-remote-gateways** | Utiliser gateway du VNet distant | VNet spoke sans gateway |

**Exemple - Configuration Hub-Spoke :**
```bash
# Hub VNet (avec VPN Gateway)
az network vnet peering create \
  --name Hub-to-Spoke \
  --resource-group HubRG \
  --vnet-name HubVNet \
  --remote-vnet $spokeVnetId \
  --allow-vnet-access \
  --allow-gateway-transit

# Spoke VNet (utilise gateway du Hub)
az network vnet peering create \
  --name Spoke-to-Hub \
  --resource-group SpokeRG \
  --vnet-name SpokeVNet \
  --remote-vnet $hubVnetId \
  --allow-vnet-access \
  --use-remote-gateways
```

**5. Caractéristiques Non-Transitives**

**⚠️ Important pour l'Examen :**
VNet Peering est **NON-TRANSITIF** par défaut

**Scénario :**
```
VNet A ←→ VNet B ←→ VNet C
```
- VNet A peut communiquer avec VNet B ✅
- VNet B peut communiquer avec VNet C ✅
- VNet A **NE PEUT PAS** communiquer avec VNet C ❌

**Solution pour rendre transitif :**
- Créer un peering direct A ←→ C
- OU utiliser une architecture Hub-Spoke avec **allow-forwarded-traffic** et une appliance réseau (NVA) dans le hub

**6. Performance et Latence**
- **Bande passante** : Identique à celle d'un VNet local (pas de limite imposée par le peering)
- **Latence** : Ultra-faible (< 1ms en regional, quelques ms en global)
- **Throughput** : Dépend uniquement des VM sizes
- **Coût Regional** : Ingress gratuit, Egress gratuit dans la même région
- **Coût Global** : Ingress et Egress facturés ($0.01-0.035/GB selon zones)

**7. Troubleshooting VNet Peering**

**Problèmes Courants :**

| Problème | Cause | Solution |
|----------|-------|----------|
| Peering en état "Initiated" | Peering bidirectionnel incomplet | Créer le peering retour |
| Communication impossible | NSG bloque le trafic | Vérifier règles NSG avec IP Flow Verify |
| Peering refusé | Adresses IP chevauchantes | Modifier les plages d'adresses |
| Gateway transit ne fonctionne pas | Options mal configurées | Vérifier allow-gateway-transit et use-remote-gateways |

**Commandes de Diagnostic :**
```bash
# Vérifier l'état du peering
az network vnet peering list \
  --resource-group myRG \
  --vnet-name myVNet \
  --output table

# Vérifier les plages d'adresses
az network vnet show \
  --resource-group myRG \
  --name myVNet \
  --query addressSpace.addressPrefixes

# Tester la connectivité (depuis une VM)
ping <private-ip-remote-vm>
```

**8. Configuration dans le Portail Azure**
- **Navigation** : Virtual Network → Peerings → + Add
- **Bidirectionnel** : Cocher "Configure peering settings on both VNets"
- **Validation automatique** : Azure vérifie la compatibilité des plages
- **État** : Doit afficher "Connected" des deux côtés

**9. Best Practices**

✅ **À FAIRE :**
- Planifier les plages d'adresses dès le début (éviter chevauchements)
- Toujours configurer le peering dans les deux sens
- Utiliser des conventions de nommage claires (VNet1-to-VNet2)
- Documenter la topologie réseau
- Tester la connectivité après création

❌ **À ÉVITER :**
- Utiliser des plages d'adresses chevauchantes
- Oublier le peering bidirectionnel
- Compter sur la transitivité par défaut
- Ignorer les NSG qui peuvent bloquer le trafic

#### DNS Resolution

** Point identifié :** DNS interne Azure
- **Format automatique** : `vm-name.internal.cloudapp.net`
- **Usage** : Résolution entre VMs dans VNet
- **Custom DNS** : Possibilité d'utiliser ses propres serveurs

### 4.1.1 Azure Private DNS Zones

#### Concepts et Fonctionnalités

**Azure Private DNS Zone**
- **Usage** : Résolution DNS privée pour ressources Azure dans un VNet
- **Avantages** : 
  - Noms personnalisés pour ressources internes
  - Résolution entre VNets connectés
  - Pas besoin de serveurs DNS personnalisés
- **Domaines** : Domaines personnalisés (ex: contoso.internal)

#### Virtual Network Links

**Processus d'association identifié :**
Pour associer un VNet à une Private DNS Zone, vous devez :
1. **Créer un Virtual Network Link** dans la zone DNS privée
2. **Sélectionner le VNet** à associer
3. **Configurer l'auto-registration** (optionnel)
4. **Valider** l'association

**Caractéristiques des Virtual Network Links :**
- **Fonction** : Lie un VNet à une zone DNS privée
- **Résolution** : Les VMs du VNet peuvent résoudre les enregistrements de la zone
- **Auto-registration** : Enregistrement automatique des VMs dans la zone DNS
- **Bi-directionnel** : Un VNet peut être lié à plusieurs zones DNS

#### Azure DNS Private Resolver

**Rôle et Utilisation :**
- **Fonction principale** : Proxy pour requêtes DNS entre environnements on-premises et Azure DNS
- **Scénario** : Connectivité hybride (on-premises <--> Azure)
- **Avantages** :
  - Résolution DNS bidirectionnelle
  - Intégration transparente avec Private DNS Zones
  - Pas de gestion de serveurs DNS
- **Architecture** : Service géré déployé dans un VNet

**Comparaison des Solutions DNS :**

**Virtual Network Link (Recommandé pour Azure-only)**
- **Usage** : Association VNet --> Private DNS Zone
- **Scope** : Azure uniquement
- **Configuration** : Simple, intégré
- **Coût** : Inclus avec Private DNS

**Azure DNS Private Resolver (Hybride)**
- **Usage** : Proxy DNS on-premises <--> Azure
- **Scope** : Environnements hybrides
- **Configuration** : Déploiement dans VNet requis
- **Coût** : Service facturé séparément

**Custom DNS Server (Non recommandé pour Private DNS)**
- **Usage** : Serveurs DNS personnalisés (VM ou appliance)
- **Limitation** : Configuration complexe, ne fonctionne pas nativement avec Private DNS Zones
- **Maintenance** : Gestion manuelle requise
- **Coût** : Infrastructure + maintenance

#### User-Defined Routes (UDR)
** Cas d'usage identifié :** Redirection de trafic vers appliances réseau
- **Objectif** : Forcer le trafic à passer par des appliances spécifiques (firewalls, appliances d'inspection)
- **Mécanisme** : Azure crée automatiquement une table de routage pour chaque sous-réseau avec des routes système par défaut
- **Override** : Les UDR permettent de remplacer certaines routes système Azure
- **Application** : Le trafic sortant d'un sous-réseau suit les routes de la table de routage du sous-réseau

** Erreur fréquente identifiée :** Confusion entre routes système et UDR
- ** Erreur** : Essayer de modifier les routes système par défaut
- ** Correct** : Créer des User-Defined Routes pour rediriger le trafic
- **Principe** : Les routes système sont gérées par Azure, les UDR permettent de surcharger le comportement

### 4.2 Network Security Groups (NSG)
Can be used with subnet or NIC

#### Règles de Sécurité
- **Priority** : 100-4096, plus bas = plus prioritaire
- **Direction** : Inbound, Outbound
- **Action** : Allow, Deny

** Optimisation identifiée :**
- **Un NSG peut être associé à plusieurs ressources**
- **5 VMs avec mêmes règles = 5 NICs + 1 NSG**
- Partage possible entre subnets et NICs

**Tips critiques identifiés :**

**1. Système de Priorités NSG**
- **Règle fondamentale** : Plus le numéro de priorité est bas, plus la règle est prioritaire
- **Exemple critique** : Priority 100 > Priority 200 (100 est plus prioritaire)
- **Impact** : Une règle Deny avec priorité élevée (100) bloque une règle Allow avec priorité faible (200)
- **Solution** : Ajuster les priorités ou modifier l'action de la règle

**2. Évaluation en Cascade : Subnet → NIC**
- **Ordre d'évaluation** : NSG du Subnet d'abord, puis NSG de la NIC
- **Principe** : Les deux niveaux doivent autoriser le trafic pour qu'il passe
- **Piège courant** : NSG Subnet Allow + NSG NIC Deny = Trafic bloqué
- **Optimisation** : Un seul NSG Deny à n'importe quel niveau bloque tout le trafic

**3. Stratégies de Résolution de Problèmes**
- **Diagnostic** : Vérifier les NSG aux deux niveaux (Subnet et NIC)
- **Priorités** : Identifier les règles conflictuelles par numéro de priorité
- **Actions** : Modifier la priorité OU changer l'action (Allow/Deny)
- **Test** : Utiliser Network Watcher pour valider les règles

#### Default Rules
**Inbound :**
- Allow VNet traffic
- Allow Azure Load Balancer
- Deny all other traffic

**Outbound :**
- Allow VNet traffic
- Allow Internet traffic
- Deny all other traffic

### 4.3 Load Balancing Solutions

#### Azure Load Balancer (Layer 4) - Guide DevOps Approfondi

**Architecture et Composants :**

**Types de Load Balancer :**
1. **Public Load Balancer**
   - **Frontend** : Adresse IP publique
   - **Usage** : Trafic Internet → VMs backend
   - **Cas d'usage** : Applications web, APIs publiques
   - **NAT** : Traduit IP publique → IP privées backend

2. **Internal Load Balancer (ILB)**
   - **Frontend** : Adresse IP privée du VNet
   - **Usage** : Trafic interne entre tiers applicatifs
   - **Cas d'usage** : App tier → Database tier, microservices
   - **Sécurité** : Jamais exposé à Internet

**SKUs - Différences Critiques :**

| Caractéristique | Basic | Standard |
|-----------------|-------|----------|
| **Backend pool size** | 300 VMs | 1000 VMs |
| **Health probes** | TCP, HTTP | TCP, HTTP, HTTPS |
| **Availability Zones** | ❌ Non | ✅ Oui |
| **SLA** | Aucun | 99.99% |
| **HA Ports** | ❌ Non | ✅ Oui |
| **Outbound rules** | Basiques | Avancées |
| **Sécurité** | Open par défaut | Fermé par défaut |
| **Coût** | Gratuit | ~$18/mois + data |
| **Production** | ⚠️ Non recommandé | ✅ Recommandé |

**⚠️ IMPORTANT DevOps** : Basic SKU sera déprécié en 2025. Toujours utiliser Standard SKU.

**Composants Techniques Détaillés :**

**1. Frontend IP Configuration**
```bash
# Public Load Balancer
az network lb create \
  --resource-group myRG \
  --name myPublicLB \
  --sku Standard \
  --public-ip-address myPublicIP \
  --frontend-ip-name myFrontend

# Internal Load Balancer
az network lb create \
  --resource-group myRG \
  --name myInternalLB \
  --sku Standard \
  --vnet-name myVNet \
  --subnet mySubnet \
  --frontend-ip-name myFrontend \
  --private-ip-address 10.0.1.10
```

**2. Backend Pool - Options de Configuration**

**A. VM-based Backend Pool**
```bash
# Ajouter des VMs au backend pool
az network nic ip-config address-pool add \
  --resource-group myRG \
  --nic-name myNIC \
  --ip-config-name ipconfig1 \
  --lb-name myLB \
  --address-pool myBackendPool
```

**B. IP-based Backend Pool (Standard SKU only)**
```bash
# Créer backend pool avec IP addresses
az network lb address-pool create \
  --resource-group myRG \
  --lb-name myLB \
  --name myBackendPool

# Ajouter une IP au pool
az network lb address-pool address add \
  --resource-group myRG \
  --lb-name myLB \
  --pool-name myBackendPool \
  --name backend1 \
  --ip-address 10.0.1.4
```

**C. VMSS Backend Pool (Auto-scaling)**
```bash
# Associer VMSS au load balancer
az vmss create \
  --resource-group myRG \
  --name myVMSS \
  --image UbuntuLTS \
  --load-balancer myLB \
  --backend-pool-name myBackendPool
```

**3. Health Probes - Configuration Avancée**

**Types de Probes :**

**TCP Probe (Recommandé pour bases de données)**
```bash
az network lb probe create \
  --resource-group myRG \
  --lb-name myLB \
  --name tcpProbe \
  --protocol tcp \
  --port 1433 \
  --interval 15 \
  --threshold 2
```
- **Fonctionnement** : Vérifie si le port répond (TCP handshake)
- **Avantage** : Léger, rapide
- **Inconvénient** : Ne vérifie pas si l'application fonctionne

**HTTP/HTTPS Probe (Recommandé pour web apps)**
```bash
az network lb probe create \
  --resource-group myRG \
  --lb-name myLB \
  --name httpProbe \
  --protocol http \
  --port 80 \
  --path /health \
  --interval 15 \
  --threshold 2
```
- **Fonctionnement** : Attend HTTP 200 OK
- **Avantage** : Vérifie la santé applicative
- **Best practice** : Créer un endpoint `/health` ou `/healthz` dédié

**Endpoint Health Check - Pattern DevOps**
```python
# Flask example
@app.route('/health')
def health_check():
    try:
        # Vérifier DB connectivity
        db.session.execute('SELECT 1')
        # Vérifier dependencies
        redis_client.ping()
        return jsonify({"status": "healthy"}), 200
    except Exception as e:
        return jsonify({"status": "unhealthy", "error": str(e)}), 503
```

**Paramètres de Health Probe - Tuning Performance**
- **Interval** : Fréquence des checks (5-300 sec, défaut: 15)
- **Threshold** : Nombre d'échecs avant marquage unhealthy (défaut: 2)
- **Calcul downtime** : `interval × threshold = temps avant exclusion`
  - Exemple : 15s × 2 = 30 secondes avant que VM soit retirée
- **Trade-off** : Interval court = détection rapide mais plus de charge

**4. Load Balancing Rules - Distribution du Trafic**

**Rule Basique**
```bash
az network lb rule create \
  --resource-group myRG \
  --lb-name myLB \
  --name myHTTPRule \
  --protocol tcp \
  --frontend-port 80 \
  --backend-port 80 \
  --frontend-ip-name myFrontend \
  --backend-pool-name myBackendPool \
  --probe-name httpProbe \
  --disable-outbound-snat false \
  --idle-timeout 15 \
  --enable-tcp-reset true
```

**Distribution Algorithms (Session Affinity)**

**A. 5-Tuple Hash (Défaut) - Aucune session persistence**
```
Hash = (Source IP, Source Port, Dest IP, Dest Port, Protocol)
```
- **Comportement** : Distribution équitable
- **Usage** : Applications stateless
- **Avantage** : Meilleure répartition de charge

**B. 3-Tuple Hash (Source IP affinity)**
```
Hash = (Source IP, Dest IP, Protocol)
```
```bash
az network lb rule update \
  --resource-group myRG \
  --lb-name myLB \
  --name myHTTPRule \
  --load-distribution SourceIP
```
- **Comportement** : Même client → même backend
- **Usage** : Applications avec état (sessions)
- **Durée** : Pendant la durée de la session TCP

**C. 2-Tuple Hash (Source IP + Protocol affinity)**
```
Hash = (Source IP, Protocol)
```
```bash
az network lb rule update \
  --resource-group myRG \
  --lb-name myLB \
  --name myHTTPRule \
  --load-distribution SourceIPProtocol
```
- **Comportement** : Client → toujours même backend pour ce protocole
- **Usage** : FTP, RDP, applications legacy

**Pattern DevOps - Choix de Distribution**
| Application Type | Distribution | Raison |
|------------------|--------------|--------|
| **REST API stateless** | 5-tuple (défaut) | Pas de session, max performance |
| **E-commerce avec panier** | 3-tuple (SourceIP) | Maintenir sessions utilisateur |
| **WebSockets** | 3-tuple (SourceIP) | Connexion persistante |
| **Remote Desktop** | 2-tuple (SourceIPProtocol) | Session RDP stable |
| **Microservices** | 5-tuple (défaut) | Stateless, scalabilité max |

**5. HA Ports (Standard SKU Only) - Pattern Avancé**

**Configuration**
```bash
az network lb rule create \
  --resource-group myRG \
  --lb-name myInternalLB \
  --name HAPortsRule \
  --protocol All \
  --frontend-port 0 \
  --backend-port 0 \
  --frontend-ip-name myFrontend \
  --backend-pool-name myBackendPool
```

**Cas d'usage DevOps**
- **Network Virtual Appliances (NVA)** : Firewalls, IDS/IPS
- **Architecture Hub-Spoke** : Load balancing de tous les flux
- **Port forwarding dynamique** : Pas besoin de règles par port

**6. Outbound Rules - Contrôle du Trafic Sortant**

**Problème** : Par défaut, VMs derrière ILB ne peuvent pas accéder à Internet

**Solution A : Outbound Rule avec Public LB**
```bash
az network lb outbound-rule create \
  --resource-group myRG \
  --lb-name myLB \
  --name myOutboundRule \
  --frontend-ip-configs myFrontend \
  --protocol All \
  --idle-timeout 15 \
  --outbound-ports 10000 \
  --address-pool myBackendPool
```

**Solution B : NAT Gateway (Recommandé pour production)**
```bash
az network nat gateway create \
  --resource-group myRG \
  --name myNATGateway \
  --public-ip-addresses myPublicIP \
  --idle-timeout 10

az network vnet subnet update \
  --resource-group myRG \
  --vnet-name myVNet \
  --name mySubnet \
  --nat-gateway myNATGateway
```

**7. Inbound NAT Rules - Accès Direct aux VMs**

**Use Case** : SSH/RDP vers VMs spécifiques derrière LB
```bash
# Port 2221 → VM1:22
az network lb inbound-nat-rule create \
  --resource-group myRG \
  --lb-name myLB \
  --name SSH-VM1 \
  --protocol tcp \
  --frontend-port 2221 \
  --backend-port 22 \
  --frontend-ip-name myFrontend

# Port 2222 → VM2:22
az network lb inbound-nat-rule create \
  --resource-group myRG \
  --lb-name myLB \
  --name SSH-VM2 \
  --protocol tcp \
  --frontend-port 2222 \
  --backend-port 22 \
  --frontend-ip-name myFrontend

# Associer à la NIC
az network nic ip-config inbound-nat-rule add \
  --resource-group myRG \
  --nic-name myVM1-NIC \
  --ip-config-name ipconfig1 \
  --inbound-nat-rule SSH-VM1
```

**Pattern DevOps - Bastion Alternative**
```
Internet → LB Public IP:2221 → VM1:22
Internet → LB Public IP:2222 → VM2:22
Internet → LB Public IP:2223 → VM3:22
```

#### Troubleshooting Load Balancer - Standard SKU

**Problèmes de connectivité courants et résolutions :**

**1. Health Probe Configuration (Priorité haute)**
- **Symptôme** : VMs ne reçoivent pas de trafic
- **Cause** : Health probes mal configurés ou VMs ne répondent pas
- **Solution** : 
  - Vérifier la configuration des health probes (protocole, port, chemin)
  - S'assurer que les VMs répondent sur le port configuré
  - Tester manuellement le endpoint de health check
- **Impact** : Si les probes échouent, le Load Balancer n'envoie pas de trafic

**2. Port Response Verification (Priorité haute)**
- **Symptôme** : Trafic n'atteint pas les backends
- **Cause** : Application non démarrée ou port non écouté
- **Solution** :
  - Vérifier que l'application écoute sur le port configuré
  - Utiliser `netstat -an` pour confirmer les ports d'écoute
  - Redémarrer les services si nécessaire
- **Impact** : Health probes échouent si le port ne répond pas

**3. NSG Rules Validation (Priorité haute)**
- **Symptôme** : Connectivité bloquée malgré Load Balancer configuré
- **Cause** : NSG bloque le trafic inbound vers les VMs
- **Solution** :
  - Vérifier les règles NSG au niveau subnet ET NIC
  - S'assurer que les règles autorisent le trafic sur les ports requis
  - Autoriser le trafic depuis AzureLoadBalancer service tag
- **Impact** : NSG peut bloquer même si Load Balancer est correct

**Actions NON recommandées (Erreurs courantes) :**
- **Modifier session persistence** : Ne résout pas les problèmes de connectivité de base
- **Augmenter timeout settings** : N'aide pas si le trafic est bloqué
- **Redémarrer les VMs** : Peut masquer temporairement sans résoudre la cause

** Stratégie de troubleshooting identifiée :**
1. **Health probes** : Vérifier configuration et réponses
2. **Port listening** : Confirmer que les applications écoutent
3. **NSG rules** : Valider les autorisations de trafic
4. **Network Watcher** : Utiliser IP Flow Verify pour diagnostic approfondi

** Tips critiques identifiés :**

**1. Session Persistence (Sticky Sessions) - Concept Clé**
- **Problème résolu** : Maintenir l'utilisateur sur le même serveur backend
- **Cas d'usage critique** : Applications avec état (paniers e-commerce, sessions utilisateur)
- **Configuration** : Client IP + Protocol pour une persistance optimale
- **Alternative** : None = distribution aléatoire (pas de persistance)

**2. Différenciation des Options de Load Balancer**
- **Session Persistence** : Contrôle la distribution des sessions utilisateur
- **NAT Rules** : Redirection de trafic spécifique (différent de la session persistence)
- **Health Probes** : Vérification de l'état des backends
- **Load Balancing Rules** : Définition des pools et méthodes de distribution

**3. Stratégies de Configuration**
- **Client IP** : Persistance basée sur l'adresse IP source
- **Protocol** : Persistance basée sur le protocole (HTTP/HTTPS)
- **Combinaison** : Client IP + Protocol pour une persistance maximale
- **Performance** : Équilibrer entre persistance et répartition de charge

#### Application Gateway (Layer 7) - Guide DevOps Approfondi

**Architecture et Composants :**

Application Gateway est un **reverse proxy intelligent** opérant au Layer 7 (HTTP/HTTPS).

**SKUs et Versions :**

| Caractéristique | v1 Standard | v1 WAF | v2 Standard | v2 WAF |
|-----------------|-------------|--------|-------------|--------|
| **Autoscaling** | ❌ Non | ❌ Non | ✅ Oui | ✅ Oui |
| **Availability Zones** | ❌ Non | ❌ Non | ✅ Oui | ✅ Oui |
| **Static VIP** | ❌ Non | ❌ Non | ✅ Oui | ✅ Oui |
| **WAF** | ❌ Non | ✅ Oui | ❌ Non | ✅ Oui |
| **Performance** | 1000 Mbps | 1000 Mbps | Supérieure | Supérieure |
| **Rewrite HTTP** | ❌ Non | ❌ Non | ✅ Oui | ✅ Oui |
| **Custom Error Pages** | ❌ Non | ❌ Non | ✅ Oui | ✅ Oui |
| **Status** | ⚠️ Legacy | ⚠️ Legacy | ✅ Recommandé | ✅ Recommandé |
| **Coût/mois (estimé)** | ~$125 | ~$250 | ~$200+ | ~$300+ |

**⚠️ IMPORTANT DevOps** : v1 est en maintenance. Toujours déployer v2 pour nouvelles installations.

**Composants de l'Application Gateway :**

**1. Frontend IP Configuration**
- **Public IP** : Exposition Internet (obligatoire pour v2)
- **Private IP** : Exposition interne VNet (optionnel)
- **Both** : Combinaison public + private possible

```bash
# Créer Application Gateway v2 avec autoscaling
az network application-gateway create \
  --name myAppGateway \
  --resource-group myRG \
  --sku WAF_v2 \
  --capacity 2 \
  --min-capacity 2 \
  --max-capacity 10 \
  --vnet-name myVNet \
  --subnet appGatewaySubnet \
  --public-ip-address myPublicIP \
  --http-settings-cookie-based-affinity Enabled \
  --http-settings-protocol Http \
  --frontend-port 80 \
  --priority 100
```

**2. Backend Pools - Multi-type Support**

Application Gateway peut cibler :
- **VMs** : Machines virtuelles Azure
- **VMSS** : VM Scale Sets
- **App Services** : Azure Web Apps
- **IP/FQDN** : Serveurs on-premises ou autres clouds
- **Private Link** : Services privés Azure

```bash
# Backend pool avec plusieurs types
az network application-gateway address-pool create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --name myBackendPool \
  --servers 10.0.1.4 10.0.1.5 webapp.azurewebsites.net
```

**3. HTTP Settings - Configuration Backend**

```bash
az network application-gateway http-settings create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --name myHTTPSettings \
  --port 80 \
  --protocol Http \
  --cookie-based-affinity Enabled \
  --timeout 30 \
  --probe myHealthProbe \
  --host-name-from-backend-pool false \
  --host-name api.internal.com
```

**Paramètres clés :**
- **Cookie-based affinity** : Session persistence (cookie ApplicationGatewayAffinity)
- **Connection draining** : Termine proprement les connexions lors de retrait backend
- **Timeout** : 1-86400 secondes (défaut: 30)
- **Override backend hostname** : Réécrit le header Host

**4. Health Probes - Monitoring Avancé**

**Probe Custom HTTP**
```bash
az network application-gateway probe create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --name myHealthProbe \
  --protocol Http \
  --host-name-from-http-settings true \
  --path /api/health \
  --interval 30 \
  --timeout 30 \
  --threshold 3 \
  --match-status-codes 200-399
```

**Pattern DevOps - Health Endpoint Avancé**
```javascript
// Node.js/Express example
app.get('/api/health', async (req, res) => {
  const health = {
    status: 'healthy',
    timestamp: new Date().toISOString(),
    checks: {}
  };
  
  try {
    // Database check
    await db.query('SELECT 1');
    health.checks.database = 'ok';
    
    // Redis check
    await redis.ping();
    health.checks.cache = 'ok';
    
    // External API check
    const apiHealth = await fetch('https://api.external.com/health');
    health.checks.externalAPI = apiHealth.ok ? 'ok' : 'degraded';
    
    res.status(200).json(health);
  } catch (error) {
    health.status = 'unhealthy';
    health.error = error.message;
    res.status(503).json(health);
  }
});
```

**5. Listeners (HTTP/HTTPS)**

**HTTP Listener Basique**
```bash
az network application-gateway http-listener create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --name myHTTPListener \
  --frontend-port appGatewayFrontendPort \
  --frontend-ip appGatewayFrontendIP \
  --host-name www.contoso.com
```

**HTTPS Listener avec SSL Certificate**
```bash
# Upload SSL certificate
az network application-gateway ssl-cert create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --name myCert \
  --cert-file /path/to/cert.pfx \
  --cert-password "P@ssw0rd"

# Créer HTTPS listener
az network application-gateway http-listener create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --name myHTTPSListener \
  --frontend-port 443 \
  --frontend-ip appGatewayFrontendIP \
  --ssl-cert myCert \
  --host-name www.contoso.com
```

**Multi-site Listener (Plusieurs domaines)**
```bash
# Listener pour site 1
az network application-gateway http-listener create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --name site1Listener \
  --frontend-port 443 \
  --ssl-cert cert1 \
  --host-names www.site1.com site1.com

# Listener pour site 2
az network application-gateway http-listener create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --name site2Listener \
  --frontend-port 443 \
  --ssl-cert cert2 \
  --host-names www.site2.com site2.com
```

**6. Routing Rules - Logique de Distribution**

**A. Basic Routing Rule**
```bash
az network application-gateway rule create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --name rule1 \
  --http-listener myHTTPListener \
  --rule-type Basic \
  --address-pool myBackendPool \
  --http-settings myHTTPSettings \
  --priority 100
```

**B. Path-based Routing (URL Routing)**
```bash
# Créer URL path map
az network application-gateway url-path-map create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --name myPathMap \
  --paths /api/* \
  --address-pool apiBackendPool \
  --http-settings apiHTTPSettings \
  --default-address-pool defaultBackendPool \
  --default-http-settings defaultHTTPSettings

# Path rule pour /images
az network application-gateway url-path-map rule create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --path-map-name myPathMap \
  --name imagesRule \
  --paths /images/* \
  --address-pool imagesBackendPool \
  --http-settings imagesHTTPSettings

# Associer à routing rule
az network application-gateway rule create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --name pathRoutingRule \
  --http-listener myListener \
  --rule-type PathBasedRouting \
  --url-path-map myPathMap \
  --priority 200
```

**Architecture Path-based Routing :**
```
www.contoso.com
├── /api/*          → API Backend Pool (port 8080)
├── /images/*       → CDN/Storage Backend Pool
├── /admin/*        → Admin Backend Pool (port 9000)
└── /*              → Web Frontend Backend Pool (port 80)
```

**7. SSL/TLS Configuration - Patterns DevOps**

**A. SSL Termination (Déchiffrement au Gateway)**
```
Client (HTTPS) → App Gateway (déchiffre) → Backend (HTTP)
```
- **Avantage** : Offload SSL processing des backends
- **Usage** : Applications internes, microservices
- **Coût CPU** : Gateway gère le chiffrement

**B. End-to-End SSL (SSL de bout en bout)**
```
Client (HTTPS) → App Gateway (HTTPS) → Backend (HTTPS)
```
```bash
az network application-gateway http-settings update \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --name myHTTPSettings \
  --protocol Https \
  --port 443 \
  --trusted-root-certificates backendCert
```
- **Avantage** : Sécurité maximale
- **Usage** : Conformité, données sensibles
- **Requis** : Certificats SSL sur backends

**C. SSL Policy Configuration**
```bash
# Politique SSL moderne (TLS 1.2+)
az network application-gateway ssl-policy set \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --min-protocol-version TLSv1_2 \
  --cipher-suites TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 \
                 TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
```

**8. Web Application Firewall (WAF) - Sécurité Avancée**

**WAF Modes :**
- **Detection** : Log les attaques, ne bloque pas
- **Prevention** : Bloque les attaques détectées

```bash
# Activer WAF en mode Prevention
az network application-gateway waf-config set \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --enabled true \
  --firewall-mode Prevention \
  --rule-set-type OWASP \
  --rule-set-version 3.2
```

**OWASP Top 10 Protection :**
1. **Injection** (SQL, NoSQL, LDAP)
2. **Broken Authentication**
3. **Sensitive Data Exposure**
4. **XML External Entities (XXE)**
5. **Broken Access Control**
6. **Security Misconfiguration**
7. **Cross-Site Scripting (XSS)**
8. **Insecure Deserialization**
9. **Using Components with Known Vulnerabilities**
10. **Insufficient Logging & Monitoring**

**Custom WAF Rules**
```bash
# Bloquer une IP spécifique
az network application-gateway waf-policy custom-rule create \
  --policy-name myWAFPolicy \
  --resource-group myRG \
  --name blockIP \
  --priority 100 \
  --rule-type MatchRule \
  --action Block \
  --match-conditions RemoteAddr IPMatch 192.168.1.100

# Rate limiting (DDoS protection)
az network application-gateway waf-policy custom-rule create \
  --policy-name myWAFPolicy \
  --resource-group myRG \
  --name rateLimitRule \
  --priority 200 \
  --rule-type RateLimitRule \
  --action Block \
  --rate-limit-duration OneMin \
  --rate-limit-threshold 100
```

**9. Rewrite Rules (v2 only) - Manipulation HTTP**

**A. Rewrite Headers**
```bash
# Rewrite Set
az network application-gateway rewrite-rule set create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --name myRewriteSet

# Ajouter X-Forwarded-For
az network application-gateway rewrite-rule create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --rule-set-name myRewriteSet \
  --name addXForwardedFor \
  --sequence 100 \
  --request-headers X-Forwarded-For="{var_client_ip}"

# Supprimer header sensible
az network application-gateway rewrite-rule create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --rule-set-name myRewriteSet \
  --name removeServerHeader \
  --sequence 200 \
  --response-headers Server=""
```

**B. URL Rewrite**
```bash
# Rediriger /old vers /new
az network application-gateway rewrite-rule create \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --rule-set-name myRewriteSet \
  --name urlRewrite \
  --sequence 300 \
  --request-headers Location="/new" \
  --conditions http_req_url Equal "/old"
```

**Use Cases DevOps :**
- **Ajouter headers de sécurité** : HSTS, X-Frame-Options, CSP
- **Client IP forwarding** : X-Forwarded-For pour les backends
- **Masquer technologies** : Supprimer Server, X-Powered-By
- **URL canonicalization** : Normaliser les URLs

**10. Autoscaling (v2 only)**

```bash
az network application-gateway update \
  --name myAppGateway \
  --resource-group myRG \
  --min-capacity 2 \
  --max-capacity 10
```

**Calcul de capacité :**
- **1 Capacity Unit** = 2500 connexions/sec persistantes
- **Scaling** : Basé sur CPU, connexions, throughput
- **Temps** : ~6-7 minutes pour scale out/in
- **Coût** : Pay-per-capacity-unit-hour

**11. Monitoring et Diagnostics - DevOps Essentials**

**Métriques Clés à Monitorer :**
```bash
# CPU Utilization
az monitor metrics list \
  --resource myAppGateway \
  --resource-group myRG \
  --resource-type Microsoft.Network/applicationGateways \
  --metric "CpuUtilization"

# Response time
az monitor metrics list \
  --resource myAppGateway \
  --resource-group myRG \
  --metric "ApplicationGatewayTotalTime"

# Failed requests
az monitor metrics list \
  --resource myAppGateway \
  --resource-group myRG \
  --metric "FailedRequests"
```

**Logs Diagnostiques :**
```bash
# Activer logs
az monitor diagnostic-settings create \
  --name myDiagnostics \
  --resource myAppGatewayId \
  --logs '[
    {
      "category": "ApplicationGatewayAccessLog",
      "enabled": true
    },
    {
      "category": "ApplicationGatewayPerformanceLog",
      "enabled": true
    },
    {
      "category": "ApplicationGatewayFirewallLog",
      "enabled": true
    }
  ]' \
  --workspace myLogAnalyticsWorkspace
```

**KQL Queries pour Troubleshooting :**
```kql
// Top 10 erreurs 5xx
AzureDiagnostics
| where ResourceType == "APPLICATIONGATEWAYS"
| where httpStatus_d >= 500
| summarize count() by httpStatus_d, requestUri_s
| top 10 by count_

// Requêtes lentes (>3 secondes)
AzureDiagnostics
| where ResourceType == "APPLICATIONGATEWAYS"
| where timeTaken_d > 3000
| project TimeGenerated, requestUri_s, timeTaken_d, clientIP_s
| order by timeTaken_d desc

// WAF blocks
AzureDiagnostics
| where ResourceType == "APPLICATIONGATEWAYS"
| where action_s == "Blocked"
| summarize count() by ruleId_s, Message
| order by count_ desc
```

**12. Architecture Patterns DevOps**

**Pattern A : Multi-Region avec Traffic Manager**
```
Traffic Manager (DNS)
├── Region 1: App Gateway → Backend Pool 1
└── Region 2: App Gateway → Backend Pool 2
```

**Pattern B : Microservices Routing**
```
App Gateway
├── /user-service/*     → User Microservice Pool
├── /order-service/*    → Order Microservice Pool
├── /payment-service/*  → Payment Microservice Pool
└── /static/*           → CDN/Storage
```

**Pattern C : Blue/Green Deployment**
```bash
# Backend pools
- bluePool (v1.0 - production actuelle)
- greenPool (v2.0 - nouvelle version)

# Étape 1: Déployer v2.0 dans greenPool
# Étape 2: Tester greenPool (health probes)
# Étape 3: Switcher routing rule: bluePool → greenPool
# Étape 4: Monitorer
# Étape 5: Rollback si problème (greenPool → bluePool)
```

**Pattern D : Canary Deployment (avec Path Rules)**
```bash
# 95% trafic → stable backend
# 5% trafic → canary backend

# Utiliser weighted routing ou custom headers
X-Canary-User: true → Canary Backend
```

**13. Infrastructure as Code - Terraform Examples**

**A. Load Balancer Standard avec Backend Pool**
```hcl
# Public IP
resource "azurerm_public_ip" "lb" {
  name                = "lb-public-ip"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  allocation_method   = "Static"
  sku                 = "Standard"
}

# Load Balancer
resource "azurerm_lb" "main" {
  name                = "main-lb"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  sku                 = "Standard"

  frontend_ip_configuration {
    name                 = "PublicIPAddress"
    public_ip_address_id = azurerm_public_ip.lb.id
  }
}

# Backend Pool
resource "azurerm_lb_backend_address_pool" "main" {
  loadbalancer_id = azurerm_lb.main.id
  name            = "backend-pool"
}

# Health Probe
resource "azurerm_lb_probe" "http" {
  loadbalancer_id = azurerm_lb.main.id
  name            = "http-probe"
  protocol        = "Http"
  request_path    = "/health"
  port            = 80
  interval_in_seconds = 15
  number_of_probes    = 2
}

# Load Balancing Rule
resource "azurerm_lb_rule" "http" {
  loadbalancer_id                = azurerm_lb.main.id
  name                           = "http-rule"
  protocol                       = "Tcp"
  frontend_port                  = 80
  backend_port                   = 80
  frontend_ip_configuration_name = "PublicIPAddress"
  backend_address_pool_ids       = [azurerm_lb_backend_address_pool.main.id]
  probe_id                       = azurerm_lb_probe.http.id
  load_distribution              = "SourceIP"
  idle_timeout_in_minutes        = 15
  enable_tcp_reset               = true
}

# NAT Rule pour SSH
resource "azurerm_lb_nat_rule" "ssh" {
  resource_group_name            = azurerm_resource_group.main.name
  loadbalancer_id                = azurerm_lb.main.id
  name                           = "ssh-vm1"
  protocol                       = "Tcp"
  frontend_port                  = 2221
  backend_port                   = 22
  frontend_ip_configuration_name = "PublicIPAddress"
}
```

**B. Application Gateway v2 avec WAF**
```hcl
# Public IP
resource "azurerm_public_ip" "appgw" {
  name                = "appgw-public-ip"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  allocation_method   = "Static"
  sku                 = "Standard"
}

# Application Gateway
resource "azurerm_application_gateway" "main" {
  name                = "main-appgateway"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name

  sku {
    name     = "WAF_v2"
    tier     = "WAF_v2"
  }

  autoscale_configuration {
    min_capacity = 2
    max_capacity = 10
  }

  gateway_ip_configuration {
    name      = "gateway-ip-config"
    subnet_id = azurerm_subnet.appgw.id
  }

  frontend_port {
    name = "https-port"
    port = 443
  }

  frontend_port {
    name = "http-port"
    port = 80
  }

  frontend_ip_configuration {
    name                 = "frontend-ip"
    public_ip_address_id = azurerm_public_ip.appgw.id
  }

  backend_address_pool {
    name = "api-backend-pool"
  }

  backend_address_pool {
    name = "web-backend-pool"
  }

  backend_http_settings {
    name                  = "https-settings"
    cookie_based_affinity = "Enabled"
    port                  = 443
    protocol              = "Https"
    request_timeout       = 30
    probe_name            = "api-probe"
    
    connection_draining {
      enabled           = true
      drain_timeout_sec = 60
    }
  }

  http_listener {
    name                           = "https-listener"
    frontend_ip_configuration_name = "frontend-ip"
    frontend_port_name             = "https-port"
    protocol                       = "Https"
    ssl_certificate_name           = "ssl-cert"
    host_name                      = "api.contoso.com"
  }

  # URL Path Map pour microservices
  url_path_map {
    name                               = "api-path-map"
    default_backend_address_pool_name  = "web-backend-pool"
    default_backend_http_settings_name = "https-settings"

    path_rule {
      name                       = "api-rule"
      paths                      = ["/api/*"]
      backend_address_pool_name  = "api-backend-pool"
      backend_http_settings_name = "https-settings"
    }
  }

  request_routing_rule {
    name                       = "api-routing-rule"
    rule_type                  = "PathBasedRouting"
    http_listener_name         = "https-listener"
    url_path_map_name          = "api-path-map"
    priority                   = 100
  }

  ssl_certificate {
    name     = "ssl-cert"
    data     = filebase64("certificate.pfx")
    password = var.ssl_password
  }

  ssl_policy {
    policy_type = "Predefined"
    policy_name = "AppGwSslPolicy20220101"
  }

  # Health Probe
  probe {
    name                = "api-probe"
    protocol            = "Https"
    path                = "/api/health"
    interval            = 30
    timeout             = 30
    unhealthy_threshold = 3
    host                = "api.contoso.com"
    
    match {
      status_code = ["200-399"]
    }
  }

  # WAF Configuration
  waf_configuration {
    enabled          = true
    firewall_mode    = "Prevention"
    rule_set_type    = "OWASP"
    rule_set_version = "3.2"
  }

  # Rewrite rules pour headers de sécurité
  rewrite_rule_set {
    name = "security-headers"
    
    rewrite_rule {
      name          = "add-hsts"
      rule_sequence = 100
      
      response_header_configuration {
        header_name  = "Strict-Transport-Security"
        header_value = "max-age=31536000; includeSubDomains"
      }
    }

    rewrite_rule {
      name          = "remove-server-header"
      rule_sequence = 200
      
      response_header_configuration {
        header_name  = "Server"
        header_value = ""
      }
    }
  }

  tags = {
    Environment = "Production"
    ManagedBy   = "Terraform"
  }
}
```

**C. Monitoring avec Azure Monitor - Terraform**
```hcl
# Log Analytics Workspace
resource "azurerm_log_analytics_workspace" "main" {
  name                = "lb-logs"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  sku                 = "PerGB2018"
  retention_in_days   = 30
}

# Diagnostic Settings pour Load Balancer
resource "azurerm_monitor_diagnostic_setting" "lb" {
  name                       = "lb-diagnostics"
  target_resource_id         = azurerm_lb.main.id
  log_analytics_workspace_id = azurerm_log_analytics_workspace.main.id

  enabled_log {
    category = "LoadBalancerAlertEvent"
  }

  enabled_log {
    category = "LoadBalancerProbeHealthStatus"
  }

  metric {
    category = "AllMetrics"
    enabled  = true
  }
}

# Diagnostic Settings pour Application Gateway
resource "azurerm_monitor_diagnostic_setting" "appgw" {
  name                       = "appgw-diagnostics"
  target_resource_id         = azurerm_application_gateway.main.id
  log_analytics_workspace_id = azurerm_log_analytics_workspace.main.id

  enabled_log {
    category = "ApplicationGatewayAccessLog"
  }

  enabled_log {
    category = "ApplicationGatewayPerformanceLog"
  }

  enabled_log {
    category = "ApplicationGatewayFirewallLog"
  }

  metric {
    category = "AllMetrics"
    enabled  = true
  }
}

# Alert pour Load Balancer unhealthy
resource "azurerm_monitor_metric_alert" "lb_unhealthy" {
  name                = "lb-unhealthy-backends"
  resource_group_name = azurerm_resource_group.main.name
  scopes              = [azurerm_lb.main.id]
  description         = "Alert when backend health drops"
  severity            = 2
  frequency           = "PT1M"
  window_size         = "PT5M"

  criteria {
    metric_namespace = "Microsoft.Network/loadBalancers"
    metric_name      = "DipAvailability"
    aggregation      = "Average"
    operator         = "LessThan"
    threshold        = 50
  }

  action {
    action_group_id = azurerm_monitor_action_group.main.id
  }
}

# Alert pour Application Gateway failed requests
resource "azurerm_monitor_metric_alert" "appgw_failed" {
  name                = "appgw-failed-requests"
  resource_group_name = azurerm_resource_group.main.name
  scopes              = [azurerm_application_gateway.main.id]
  description         = "Alert when failed requests exceed threshold"
  severity            = 1
  frequency           = "PT1M"
  window_size         = "PT5M"

  criteria {
    metric_namespace = "Microsoft.Network/applicationGateways"
    metric_name      = "FailedRequests"
    aggregation      = "Total"
    operator         = "GreaterThan"
    threshold        = 100
  }

  action {
    action_group_id = azurerm_monitor_action_group.main.id
  }
}
```

**14. Best Practices DevOps - Checklist de Production**

**Load Balancer :**
- ✅ **Toujours utiliser Standard SKU** (Basic deprecated)
- ✅ **Configurer health probes avec endpoints dédiés** `/health` ou `/healthz`
- ✅ **Utiliser zones de disponibilité** pour HA multi-datacenter
- ✅ **Activer TCP Reset** pour connexions stale
- ✅ **Monitorer DipAvailability metric** (santé des backends)
- ✅ **Configurer NSG pour autoriser AzureLoadBalancer tag**
- ✅ **Utiliser NAT Gateway pour outbound** (pas outbound rules LB)
- ✅ **Session persistence selon besoin applicatif** (stateful vs stateless)
- ✅ **Logs diagnostiques vers Log Analytics**
- ✅ **Alerts sur health probe failures**

**Application Gateway :**
- ✅ **Toujours déployer v2** (v1 legacy)
- ✅ **Activer autoscaling** min=2, max=10+
- ✅ **WAF en mode Prevention** pour production
- ✅ **End-to-end SSL** pour données sensibles
- ✅ **TLS 1.2+ uniquement** (désactiver TLS 1.0/1.1)
- ✅ **Health probes custom** avec vérifications applicatives
- ✅ **Connection draining** activé (60s minimum)
- ✅ **Rewrite rules** pour headers de sécurité (HSTS, X-Frame-Options)
- ✅ **Dedicated subnet** /24 minimum pour App Gateway
- ✅ **Custom error pages** pour meilleure UX
- ✅ **Logs vers SIEM** (Sentinel, Splunk)
- ✅ **Monitor Capacity Units** pour coûts
- ✅ **Alerts sur 5xx errors** et latence

**Sécurité :**
- ✅ **NSG sur subnet Application Gateway**
- ✅ **WAF custom rules** pour votre application
- ✅ **Rate limiting** pour DDoS protection
- ✅ **IP whitelisting** si applicable
- ✅ **Certificates dans Key Vault** (pas dans code)
- ✅ **Rotation automatique certificats**
- ✅ **RBAC strict** sur ressources load balancing

**Monitoring et Alerting :**
- ✅ **Dashboard Azure Monitor** avec métriques clés
- ✅ **Alerts sur santé backends**
- ✅ **Alerts sur latence élevée**
- ✅ **Alerts sur erreurs 5xx**
- ✅ **Workbooks pour analyse trafic**
- ✅ **Integration avec outils DevOps** (PagerDuty, Slack)

**15. Troubleshooting Avancé - Playbook DevOps**

**Problème : Backends marqués unhealthy**
```bash
# 1. Vérifier status health probes
az network lb show --name myLB --resource-group myRG \
  --query "probes[].{Name:name,Protocol:protocol,Port:port,Path:requestPath}"

# 2. Tester manuellement l'endpoint
curl -v http://10.0.1.4/health

# 3. Vérifier NSG rules
az network nsg show --name myNSG --resource-group myRG \
  --query "securityRules[?destinationPortRange=='80']"

# 4. Vérifier depuis la VM que le service écoute
ssh user@vm1
netstat -tlnp | grep :80

# 5. Vérifier logs applicatifs
tail -f /var/log/app/error.log
```

**Problème : Application Gateway erreurs 502**
```bash
# 1. Vérifier backend health
az network application-gateway show-backend-health \
  --name myAppGateway \
  --resource-group myRG

# 2. Analyser logs
az monitor log-analytics query \
  --workspace myWorkspace \
  --analytics-query "
    AzureDiagnostics
    | where ResourceType == 'APPLICATIONGATEWAYS'
    | where httpStatus_d == 502
    | project TimeGenerated, requestUri_s, backendSettingName_s
    | take 50"

# 3. Vérifier timeout settings
az network application-gateway http-settings show \
  --gateway-name myAppGateway \
  --resource-group myRG \
  --name myHTTPSettings \
  --query "requestTimeout"

# 4. Tester backend directement
curl -H "Host: api.contoso.com" http://10.0.1.4/api/endpoint
```

**Problème : Performance dégradée**
```bash
# Load Balancer - Vérifier SNAT port exhaustion
az monitor metrics list \
  --resource myLB \
  --metric "UsedSNATPorts" \
  --resource-type Microsoft.Network/loadBalancers

# Application Gateway - Vérifier capacity units
az monitor metrics list \
  --resource myAppGateway \
  --metric "CurrentCapacityUnits" \
  --resource-type Microsoft.Network/applicationGateways

# Latency analysis
az monitor metrics list \
  --resource myAppGateway \
  --metric "ApplicationGatewayTotalTime,BackendResponseTime"
```

**16. Coûts et Optimisation**

**Load Balancer Standard - Calcul Coûts**
```
Coût mensuel = Base fee + Data processed

Base fee: ~$18/mois
Data processed: $0.005/GB

Exemple :
- 1TB trafic/mois = 1000 GB × $0.005 = $5
- Total: $18 + $5 = $23/mois
```

**Application Gateway v2 - Calcul Coûts**
```
Coût mensuel = Fixed capacity units + Variable capacity units + Data processed

Fixed (2 CU): ~$145/mois
Variable CU: $0.008/CU-hour
Data processed: $0.008/GB

Exemple avec autoscaling 2-5 CU:
- Base (2 CU): $145
- Variable (3 CU × 730h × $0.008): $17.52
- Data (500 GB × $0.008): $4
- Total: ~$167/mois
```

**Optimisation Coûts :**
- **Load Balancer** : Moins cher pour TCP/UDP simple
- **App Gateway v2** : Autoscaling évite surprovisionnement
- **Zones de disponibilité** : Pas de coût supplémentaire (sauf data transfer)
- **WAF** : +30-40% coût vs Standard (évaluer nécessité)

#### Comparaison Détaillée : Azure Load Balancer vs Application Gateway

**🎯 Différences Critiques pour l'Examen AZ-104**

**1. Couche OSI et Protocoles**

**Azure Load Balancer (Layer 4 - Transport)**
- **Protocoles supportés** : TCP, UDP uniquement
- **Fonctionnement** : Distribution basée sur IP source/destination + port
- **Visibilité** : Ne peut pas voir le contenu des paquets
- **Usage** : Load balancing basique, NAT, HA ports
- **Performance** : Très haute (pas d'inspection de contenu)

**Application Gateway (Layer 7 - Application)**
- **Protocoles supportés** : HTTP, HTTPS uniquement
- **Fonctionnement** : Inspection du contenu HTTP/HTTPS
- **Visibilité** : Peut analyser headers, URLs, cookies, body
- **Usage** : Routage intelligent, WAF, SSL termination
- **Performance** : Plus élevée latence (inspection de contenu)

**2. Fonctionnalités et Capacités**

**Azure Load Balancer - Fonctionnalités Clés**
- **Health Probes** : TCP, HTTP, HTTPS
- **Session Persistence** : Client IP, Client IP + Protocol, None
- **NAT Rules** : Port forwarding, inbound/outbound
- **HA Ports** : Load balancing sur tous les ports
- **Backend Pools** : VMs, VMSS, IP addresses
- **Distribution Methods** : 5-tuple hash, 3-tuple hash, Source IP

**Application Gateway - Fonctionnalités Clés**
- **URL-based Routing** : Routage basé sur le chemin URL
- **Host-based Routing** : Routage basé sur le header Host
- **Path-based Routing** : Routage basé sur le chemin de l'URL
- **Multi-site Hosting** : Plusieurs domaines sur même gateway
- **SSL Termination** : Décryptage SSL côté gateway
- **WAF Integration** : Protection contre OWASP Top 10
- **Cookie-based Affinity** : Session persistence basée sur cookies

**3. Cas d'Usage et Scénarios**

**Utiliser Azure Load Balancer quand :**
- **Applications non-HTTP** : Bases de données, services TCP/UDP
- **Performance maximale** : Latence minimale requise
- **Simplicité** : Load balancing basique sans inspection
- **Coût** : Solution la moins chère
- **Backend hétérogène** : Mélange de services différents

**Utiliser Application Gateway quand :**
- **Applications web** : Sites web, APIs REST
- **Routage intelligent** : Besoin de router selon URL/host
- **Sécurité web** : Protection contre attaques web
- **SSL centralisé** : Gestion centralisée des certificats
- **Multi-tenant** : Plusieurs sites sur même infrastructure

**4. Architecture et Déploiement**

**Azure Load Balancer**
- **Types** : Public, Internal
- **SKUs** : Basic, Standard
- **Backend** : VMs, VMSS, IP addresses
- **Frontend** : Public IP ou Private IP
- **Zones** : Standard SKU supporte Availability Zones

**Application Gateway**
- **Types** : v1, v2 (WAF v2)
- **SKUs** : Standard, WAF, Standard_v2, WAF_v2
- **Backend** : VMs, VMSS, App Services, IP addresses
- **Frontend** : Public IP uniquement
- **Zones** : v2 SKU supporte Availability Zones

**5. Configuration et Gestion**

**Azure Load Balancer - Configuration Type**
```json
{
  "loadBalancingRules": [
    {
      "name": "LBRule",
      "protocol": "Tcp",
      "frontendPort": 80,
      "backendPort": 80,
      "enableFloatingIP": false
    }
  ],
  "probes": [
    {
      "name": "HTTPProbe",
      "protocol": "Http",
      "port": 80,
      "path": "/health"
    }
  ]
}
```

**Application Gateway - Configuration Type**
```json
{
  "routingRules": [
    {
      "name": "Rule1",
      "ruleType": "Basic",
      "httpListener": "Listener1",
      "backendAddressPool": "Pool1",
      "backendHttpSettings": "Settings1"
    }
  ],
  "httpListeners": [
    {
      "name": "Listener1",
      "frontendPort": "Port1",
      "protocol": "Http"
    }
  ]
}
```

**6. Coûts et Facturation**

**Azure Load Balancer**
- **Basic** : Gratuit (limitations)
- **Standard** : ~$18/mois + trafic sortant
- **Facturation** : Par règle + trafic

**Application Gateway**
- **v1** : ~$18/mois + capacité
- **v2** : Pay-per-use + capacité
- **WAF** : Coût supplémentaire
- **Facturation** : Par heure + capacité + données

**7. Limitations et Contraintes**

**Azure Load Balancer**
- **Basic SKU** : Pas de HA ports, pas de zones
- **Backend** : Maximum 1000 instances
- **Rules** : Maximum 150 rules
- **Probes** : Maximum 5 probes

**Application Gateway**
- **Backend** : Maximum 100 instances
- **Rules** : Maximum 100 rules
- **Listeners** : Maximum 40 listeners
- **Certificates** : Maximum 20 certificates

**8. Matrice de Décision Rapide**

| Critère | Azure Load Balancer | Application Gateway |
|---------|-------------------|-------------------|
| **Protocole** | TCP/UDP | HTTP/HTTPS |
| **Couche** | Layer 4 | Layer 7 |
| **Performance** | Très haute | Haute |
| **Coût** | Faible | Élevé |
| **Sécurité** | Basique | Avancée (WAF) |
| **Routage** | Basique | Intelligent |
| **SSL** | Pas de gestion | Termination |
| **Monitoring** | Métriques de base | Métriques avancées |

**9. Scénarios d'Examen Courants**

**Scénario 1 : Application Web avec Routage**
- **Besoin** : Router `/api` vers backend API, `/app` vers frontend
- **Solution** : Application Gateway avec path-based routing
- **Raison** : Load Balancer ne peut pas router selon URL

**Scénario 2 : Base de Données avec HA**
- **Besoin** : Load balancing pour SQL Server
- **Solution** : Azure Load Balancer
- **Raison** : Application Gateway ne supporte que HTTP/HTTPS

**Scénario 3 : Sécurité Web**
- **Besoin** : Protection contre attaques OWASP
- **Solution** : Application Gateway avec WAF
- **Raison** : Load Balancer n'a pas de fonctionnalités de sécurité web

**Scénario 4 : Performance Maximale**
- **Besoin** : Latence minimale pour application critique
- **Solution** : Azure Load Balancer
- **Raison** : Pas d'inspection de contenu = latence minimale

#### Traffic Manager (DNS-based)
- **Global** : Répartition géographique
- **Methods** : Performance, Geographic, Weighted, Priority
- **Health monitoring** : Surveillance des endpoints

### 4.4 Network Watcher - Outils de Diagnostic Réseau

**⚠️ Erreur Courante QCM : Choisir le bon outil Network Watcher**

#### Vue d'ensemble et Outils
**Azure Network Watcher** est un service central de monitoring et diagnostic réseau qui fournit des outils pour :
- **Monitoring** : Surveillance continue des ressources réseau
- **Diagnostics** : Identification et résolution de problèmes de connectivité
- **Métriques** : Visualisation des performances réseau
- **Logging** : Activation/désactivation des logs pour ressources Azure VNet

**Disponibilité :**
- **Activation** : Automatique lors de la création d'un VNet
- **Scope** : Par région (un Network Watcher par région)
- **Gestion** : Network Watcher → Région → Outils disponibles

**Outils Network Watcher - Vue d'ensemble :**

| Outil | Use Case Principal | Sortie | Rapidité | Complexité |
|-------|-------------------|--------|----------|------------|
| **IP Flow Verify** | Vérifier si NSG bloque | Allow/Deny + Règle NSG | Immédiat | Faible |
| **Connection Troubleshoot** | Tester connectivité VM → VM | Reachable/Unreachable | Rapide | Faible |
| **Next Hop** | Vérifier routage | Type de hop suivant | Immédiat | Faible |
| **Topology** | Visualiser architecture | Diagramme réseau | Rapide | Faible |
| **Packet Capture** | Analyser trafic détaillé | Fichier .cap | Long | Élevée |
| **NSG Flow Logs** | Analyser historique trafic | Logs JSON | Délai analyse | Élevée |
| **Connection Monitor** | Surveiller latence continue | Métriques temps réel | Continu | Moyenne |

#### 1. IP Flow Verify - Diagnostic NSG (⭐ Outil Principal pour NSG)

**⚠️ Scénario d'Examen Typique :**
- **Question** : "Une VM ne peut pas se connecter à une autre VM. Quel outil utiliser pour identifier la règle NSG bloquante ?"
- **Réponse correcte** : **IP Flow Verify**

**Fonctionnalités principales :**
- **Spécification complète** : Source/destination IPv4, port, protocole (TCP/UDP), direction (inbound/outbound)
- **Identification précise** : Identifie le **NSG spécifique** et la **règle exacte** qui bloque
- **Résultat immédiat** : Allow ou Deny avec détails de la règle
- **Cas d'usage** : Troubleshooting rapide des problèmes de connectivité NSG

**Configuration via Azure CLI :**
```bash
# Vérifier si le trafic est autorisé
az network watcher test-ip-flow \
  --resource-group myRG \
  --vm myVM \
  --direction Outbound \
  --protocol TCP \
  --local 10.0.0.4:80 \
  --remote 10.1.0.4:3389

# Résultat exemple :
# Access: Deny
# Rule Name: UserRule_DenyRDP
# NSG: myNSG
```

**Configuration via PowerShell :**
```powershell
Test-AzNetworkWatcherIPFlow `
  -NetworkWatcher $nw `
  -TargetVirtualMachineId $vm.Id `
  -Direction Outbound `
  -Protocol TCP `
  -LocalIPAddress "10.0.0.4" `
  -LocalPort "80" `
  -RemoteIPAddress "10.1.0.4" `
  -RemotePort "3389"
```

**Sortie Détaillée :**
```json
{
  "access": "Deny",
  "ruleName": "SecurityRule_DenyAll",
  "networkSecurityGroup": "/subscriptions/.../networkSecurityGroups/myNSG"
}
```

**Avantages :**
- ✅ **Identification directe** du NSG bloquant
- ✅ **Configuration minimale**, test immédiat
- ✅ **Résultat précis** : règle NSG exacte responsable du blocage
- ✅ **Usage** : Diagnostic rapide et précis

#### 2. Connection Troubleshoot - Test de Connectivité End-to-End

**Use Case :**
- Tester la connectivité entre deux ressources Azure
- Identifier si la communication est possible (reachable/unreachable)
- Détecter les problèmes de routage, NSG, firewall

**Sources supportées :**
- Virtual Machines
- VM Scale Sets
- Application Gateway
- Azure Bastion

**Destinations supportées :**
- VM (IP privée)
- URI (HTTP/HTTPS)
- FQDN (nom de domaine)
- IPv4 publique ou privée

**Configuration via Azure CLI :**
```bash
# Tester connectivité VM → VM
az network watcher test-connectivity \
  --resource-group myRG \
  --source-resource myVM1 \
  --dest-resource myVM2 \
  --protocol TCP \
  --dest-port 443

# Tester connectivité VM → URL
az network watcher test-connectivity \
  --resource-group myRG \
  --source-resource myVM1 \
  --dest-address www.microsoft.com \
  --protocol TCP \
  --dest-port 443
```

**Sortie Exemple :**
```json
{
  "connectionStatus": "Reachable",
  "avgLatencyInMs": 4,
  "minLatencyInMs": 2,
  "maxLatencyInMs": 8,
  "probesSent": 10,
  "probesFailed": 0
}
```

**Cas d'échec - Diagnostics :**
```json
{
  "connectionStatus": "Unreachable",
  "hops": [
    {
      "type": "Source",
      "id": "myVM1",
      "issues": []
    },
    {
      "type": "VirtualNetwork",
      "issues": []
    },
    {
      "type": "NetworkSecurityGroup",
      "id": "myNSG",
      "issues": [
        {
          "type": "NetworkSecurityRule",
          "context": ["Rule 'DenyHTTPS' blocked the connection"]
        }
      ]
    }
  ]
}
```

#### 3. Next Hop - Vérification du Routage

**Use Case :**
- Déterminer comment le trafic est routé depuis une VM
- Identifier le prochain saut (next hop) pour une destination
- Diagnostiquer problèmes de routage (UDR, routes système)

**Types de Next Hop :**
- **Internet** : Trafic vers Internet
- **VirtualAppliance** : Via une appliance réseau (NVA)
- **VirtualNetworkGateway** : Via VPN ou ExpressRoute Gateway
- **VnetLocal** : Destination dans le même VNet
- **VNetPeering** : Destination via VNet Peering
- **None** : Aucun routage (trafic bloqué)

**Configuration via Azure CLI :**
```bash
az network watcher show-next-hop \
  --resource-group myRG \
  --vm myVM \
  --source-ip 10.0.0.4 \
  --dest-ip 10.1.0.4

# Résultat :
# NextHopType: VNetPeering
# NextHopIpAddress: 10.1.0.4
# RouteTableId: System Route
```

**Scénario d'Examen :**
| Question | Next Hop Type | Raison |
|----------|---------------|--------|
| Trafic vers 8.8.8.8 | **Internet** | Destination publique |
| Trafic vers 10.1.0.4 (VNet peeré) | **VNetPeering** | Via peering |
| Trafic via NVA Firewall | **VirtualAppliance** | UDR configuré |
| Trafic vers on-premises via VPN | **VirtualNetworkGateway** | Via VPN Gateway |

#### 4. Topology - Visualisation du Réseau

**Use Case :**
- Visualiser l'architecture réseau d'un Resource Group
- Comprendre les interconnexions entre ressources
- Identifier rapidement la topologie globale

**Éléments affichés :**
- VNets et Subnets
- VMs et NICs
- Load Balancers
- Application Gateways
- VNet Peerings
- VPN Gateways
- NSGs associés

**Configuration via Azure CLI :**
```bash
az network watcher show-topology \
  --resource-group myRG \
  --output json > topology.json
```

**Utilisation Portal :**
```
Network Watcher → Topology → Sélectionner Resource Group
```

#### 5. Packet Capture - Analyse de Paquets Réseau

**Use Case :**
- Capture détaillée du trafic réseau pour analyse approfondie
- Debugging de problèmes applicatifs (HTTP, SQL, etc.)
- Analyse de sécurité (détection d'intrusions)

**Configuration :**
```bash
# Démarrer une capture
az network watcher packet-capture create \
  --resource-group myRG \
  --vm myVM \
  --name myCapture \
  --storage-account mystorageaccount \
  --time-limit 60

# Arrêter la capture
az network watcher packet-capture stop \
  --resource-group myRG \
  --name myCapture \
  --location eastus

# Télécharger et analyser avec Wireshark
```

**Limitations :**
- Nécessite l'extension VM Network Watcher
- Capture limitée en durée (max 5 heures)
- Impact potentiel sur les performances VM
- Fichiers .cap volumineux

**⚠️ Important :** Ne convient PAS pour identifier rapidement un NSG bloquant (utiliser IP Flow Verify)

#### 6. NSG Flow Logs - Analyse Historique du Trafic

**Use Case :**
- Analyser le trafic IP à travers les NSG sur une période prolongée
- Auditer et tracer les connexions réseau
- Détecter anomalies et patterns de trafic
- Conformité et forensic

**Configuration :**
```bash
# Activer NSG Flow Logs
az network watcher flow-log create \
  --resource-group myRG \
  --nsg myNSG \
  --storage-account mystorageaccount \
  --enabled true \
  --retention 90 \
  --format JSON \
  --log-version 2

# Activer Traffic Analytics (optionnel)
az network watcher flow-log configure \
  --nsg myNSG \
  --enabled true \
  --storage-account mystorageaccount \
  --workspace myLogAnalyticsWorkspace \
  --traffic-analytics true
```

**Format des Logs (Version 2) :**
```json
{
  "time": "2025-10-27T10:00:00Z",
  "systemId": "xxx",
  "macAddress": "00-0D-3A-...",
  "category": "NetworkSecurityGroupFlowEvent",
  "resourceId": "/subscriptions/.../networkSecurityGroups/myNSG",
  "operationName": "NetworkSecurityGroupFlowEvents",
  "properties": {
    "Version": 2,
    "flows": [
      {
        "rule": "Allow-HTTP",
        "flows": [
          {
            "mac": "00-0D-3A-...",
            "flowTuples": [
              "1698393600,10.0.0.4,10.1.0.4,80,3389,T,I,A,C,1024,512,10,5"
            ]
          }
        ]
      }
    ]
  }
}
```

**Tuple Format :**
```
timestamp,source_ip,dest_ip,source_port,dest_port,protocol,flow_direction,flow_state,encryption,bytes_sent,bytes_received,packets_sent,packets_received
```

**Traffic Analytics :**
- **Visualisation** : Dashboards dans Azure Monitor
- **Insights** : Top talkers, blocked traffic, geo-distribution
- **Alerting** : Alertes sur anomalies détectées

**Limitations :**
- ❌ Configuration complexe (Storage Account + Log Analytics requis)
- ❌ Analyse manuelle des logs JSON
- ❌ Délai d'ingestion (5-10 minutes)
- ❌ Ne pointe pas directement le NSG problématique en temps réel

#### 7. Connection Monitor - Surveillance Continue de Latence

**Use Case :**
- Mesurer la latence (RTT - Round Trip Time) entre VMs en continu
- Surveiller la disponibilité des endpoints
- Détecter dégradations de performance réseau
- Monitoring proactif

**Caractéristiques :**
- **Granularité** : Métriques par minute
- **Targets** : VM, FQDN, URI, IPv4 privée/publique
- **Protocols** : TCP, HTTP, HTTPS, ICMP
- **Monitoring** : 24/7 avec historique

**Configuration via Azure CLI :**
```bash
# Créer un Connection Monitor
az network watcher connection-monitor create \
  --name myConnectionMonitor \
  --location eastus \
  --endpoint-source-name VM1 \
  --endpoint-source-resource-id $vm1Id \
  --endpoint-dest-name VM2 \
  --endpoint-dest-resource-id $vm2Id \
  --test-config-name HTTPTest \
  --protocol HTTP \
  --http-port 80 \
  --test-frequency 60

# Voir les résultats
az network watcher connection-monitor show \
  --name myConnectionMonitor \
  --location eastus
```

**Métriques collectées :**
- **RTT (Round Trip Time)** : Latence moyenne, min, max
- **Probes Sent/Failed** : Taux de succès
- **Checks Failed Percent** : Pourcentage d'échec

**Alertes :**
```bash
# Créer une alerte si RTT > 100ms
az monitor metrics alert create \
  --name HighLatencyAlert \
  --resource-group myRG \
  --scopes $connectionMonitorId \
  --condition "avg RoundTripTimeMs > 100" \
  --description "Alert when RTT exceeds 100ms"
```

#### Matrice de Décision - Quel Outil Utiliser ?

| Symptôme / Question | Outil Recommandé | Raison |
|---------------------|-----------------|--------|
| **VM ne peut pas communiquer avec une autre VM** | IP Flow Verify | Identifie NSG bloquant + règle exacte |
| **Identifier quelle règle NSG bloque le trafic** | IP Flow Verify | Diagnostic NSG en temps réel |
| **Tester si une VM peut atteindre une URL** | Connection Troubleshoot | Test end-to-end avec diagnostics |
| **Comprendre le routage du trafic** | Next Hop | Montre le prochain saut |
| **Visualiser l'architecture réseau** | Topology | Diagramme de toutes les ressources |
| **Analyser trafic détaillé (debugging app)** | Packet Capture | Capture complète pour Wireshark |
| **Auditer historique de trafic NSG** | NSG Flow Logs | Logs détaillés sur période longue |
| **Surveiller latence en continu** | Connection Monitor | Monitoring proactif 24/7 |

#### Comparaison Détaillée - Scénarios d'Examen

**Scénario 1 : Communication VM bloquée - Diagnostic rapide**
- **Besoin** : Identifier pourquoi une VM ne peut pas communiquer
- **Solution** : **IP Flow Verify** ✅
- **Raison** : Identification immédiate du NSG et de la règle bloquante
- **Alternative (mauvaise)** : NSG Flow Logs ❌ (trop long, analyse manuelle)

**Scénario 2 : Vérifier connectivité à un service externe**
- **Besoin** : Tester si une VM peut atteindre https://api.example.com
- **Solution** : **Connection Troubleshoot** ✅
- **Raison** : Test direct avec indication reachable/unreachable

**Scénario 3 : Comprendre pourquoi le trafic passe par un NVA**
- **Besoin** : Déterminer le routage du trafic
- **Solution** : **Next Hop** ✅
- **Raison** : Montre le type de routage (VirtualAppliance) et l'IP du NVA

**Scénario 4 : Analyser un problème applicatif HTTP complexe**
- **Besoin** : Voir les requêtes/réponses HTTP détaillées
- **Solution** : **Packet Capture** ✅
- **Raison** : Capture complète pour analyse Wireshark

**Scénario 5 : Conformité - Audit des connexions sur 6 mois**
- **Besoin** : Tracer toutes les connexions pour audit
- **Solution** : **NSG Flow Logs** + Traffic Analytics ✅
- **Raison** : Historique complet avec rétention longue durée

#### Best Practices Network Watcher

✅ **À FAIRE :**
- Utiliser **IP Flow Verify** en premier pour problèmes NSG
- Activer **NSG Flow Logs** pour audit et conformité
- Configurer **Connection Monitor** pour services critiques
- Utiliser **Topology** pour comprendre l'architecture

❌ **À ÉVITER :**
- Utiliser Packet Capture pour identifier un NSG bloquant (trop complexe)
- Analyser manuellement NSG Flow Logs sans Traffic Analytics
- Oublier d'activer Network Watcher dans les nouvelles régions

** Stratégie de diagnostic identifiée :**
1. **Premier choix** : IP Flow Verify pour identifier rapidement le NSG
2. **Deuxième choix** : NSG Flow Logs pour analyse historique
3. **Dernier recours** : Packet Capture pour investigation approfondie

** Tips critiques identifiés :**

**1. Commandes de Diagnostic Spécialisées**
- **`netstat -an`** : Diagnostic des ports d'écoute (essentiel pour troubleshooting)
- **`Test-NetConnection`** : Tests de connectivité modernes (remplace ping)
- **`nbtstat -c`** : Diagnostic NetBIOS (legacy, moins fréquent)
- **`Get-AzVirtualNetworkUsageList`** : PowerShell Azure (pas de diagnostic réseau)

**2. Stratégie de Diagnostic par Couche**
- **Couche Application** : `netstat -an` pour ports d'écoute
- **Couche Transport** : `Test-NetConnection` pour tests TCP/UDP
- **Couche Réseau** : `ping` ou `Test-NetConnection` pour ICMP
- **Couche Application** : `nslookup` pour résolution DNS

**3. Outils Azure vs Outils Système**
- **Azure PowerShell** : `Get-Az*` pour gestion des ressources Azure
- **Outils Windows** : `netstat`, `Test-NetConnection` pour diagnostic réseau
- **Outils Legacy** : `nbtstat`, `ping` pour compatibilité
- **Règle** : Diagnostic réseau = outils système, pas PowerShell Azure

#### Traffic Analytics
** Ressources requises identifiées :**
1. **Log Analytics Workspace** : Analyse et stockage
2. **Storage Account** : Stockage NSG Flow Logs
3. **NSG Flow Logs** : Source de données activée

**Prérequis :** Même région pour tous les composants

#### Other Features
- **IP Flow Verify** : Tester règles NSG
- **Next Hop** : Routing troubleshooting
- **Packet Capture** : Capture de paquets sur VMs

### 4.5 VPN et ExpressRoute

#### Site-to-Site VPN
- **Virtual Network Gateway** : Passerelle Azure
- **Local Network Gateway** : Représentation on-premises
- **Connection** : Lien entre les gateways
- **Protocols** : IKEv1, IKEv2, SSTP

** Cas d'usage identifié :** Connexions chiffrées on-premises
- **Scénario** : Activer la connectivité VNet vers ressources on-premises avec connexion chiffrée
- **Solution** : Configurer une Virtual Network Gateway (VPN Gateway)
- **Mécanisme** : Envoie du trafic chiffré entre un réseau virtuel et un emplacement on-premises via connexion publique
- **Configuration** : Dépend de plusieurs ressources avec paramètres configurables

** Erreur fréquente identifiée :** Confusion entre Private Endpoints et VPN Gateways
- ** Erreur** : Utiliser des Private Endpoints pour la connectivité on-premises
- ** Correct** : Utiliser des Virtual Network Gateways pour les connexions chiffrées
- **Différenciation** : Private Endpoints = Accès privé aux services Azure
- **Usage** : VPN Gateways = Connexions chiffrées vers on-premises

#### Point-to-Site VPN
- **Client certificates** : Authentification par certificat
- **Azure AD authentication** : Authentification moderne
- **RADIUS** : Authentification externe

#### ExpressRoute
- **Private connectivity** : Connexion privée dédiée
- **No Internet** : Pas de transit par Internet
- **Higher bandwidth** : Jusqu'à 100 Gbps
- **Lower latency** : Latence prévisible

### 4.6 Azure Bastion

#### Vue d'ensemble
**Azure Bastion** est un service PaaS géré qui fournit une connectivité sécurisée aux VMs

**Caractéristiques principales :**
- **Connexion via navigateur** : RDP/SSH directement depuis le portail Azure
- **Sécurité renforcée** : Pas d'exposition des ports RDP (3389) et SSH (22)
- **Pas d'IP publique requise** : Les VMs restent complètement privées
- **Protection DDoS** : Intégré avec Azure DDoS Protection
- **Protocoles** : RDP et SSH via SSL (port 443)

#### Comparaison des Solutions d'Accès aux VMs

**Azure Bastion (Recommandé)**
- **Avantage** : Sécurité maximale, pas d'exposition de ports
- **Accès** : Via navigateur et portail Azure
- **Configuration** : Service géré, simple à déployer
- **Coût** : Service facturé par heure
- **Usage** : Production, environnements sécurisés

**Remote Desktop (RDP Direct)**
- **Type** : Fonctionnalité du système d'exploitation Windows
- **Exposition** : Port RDP (3389) exposé sur Internet
- **Risque** : Vulnérable aux attaques brute force
- **Configuration** : Nécessite IP publique sur la VM
- **Usage** : Environnements de développement uniquement

**Azure Monitor**
- **Fonction** : Monitoring et diagnostics
- **Limitation** : N'offre PAS de connectivité aux VMs
- **Usage** : Surveillance des performances

**Azure Network Watcher**
- **Fonction** : Diagnostic réseau
- **Limitation** : N'offre PAS de connectivité RDP/SSH
- **Usage** : Troubleshooting connectivité

** Point clé identifié :** Sécurité vs Accessibilité
- **Azure Bastion** : Meilleure pratique pour accès sécurisé sans exposition de ports
- **RDP Direct** : Solution simple mais dangereuse, exposée aux attaques
- **Règle** : Toujours préférer Azure Bastion en production

#### Références Microsoft
- Quickstart - Create an Azure private DNS zone using the Azure portal | Microsoft Learn
- Configure Azure DNS - Training | Microsoft Learn
- Monitor and maintain Azure resources
- Azure Network Watcher | Microsoft Learn

---

## 5. Monitor and Backup Azure Resources (10-15%)

### 5.1 Azure Monitor

#### Architecture et Fonctionnalités
**Azure Monitor** aide à maximiser la disponibilité et les performances des applications et services

- **Data Collection** : Métriques, logs, traces
- **Storage** : Metrics store, Log Analytics workspace
- **Analysis** : KQL queries, workbooks, dashboards
- **Actions** : Alerts, autoscale, automation

**Différenciation des services de monitoring Azure :**

**Azure Monitor**
- **Fonction** : Maximiser disponibilité et performance des applications
- **Capacités** : Collecte de métriques, logs, alertes, dashboards
- **Usage** : Monitoring complet des ressources Azure
- **API** : HTTP Data Collector API pour envoyer logs vers Log Analytics

**Azure Network Watcher**
- **Fonction** : Monitoring et diagnostic réseau
- **Capacités** : IP Flow Verify, NSG Flow Logs, Packet Capture, Connection Monitor
- **Usage** : Troubleshooting connectivité et diagnostic réseau
- **Scope** : Ressources réseau uniquement (VNets, NSGs, VMs)

**Azure Resource Manager**
- **Fonction** : Service de déploiement et gestion Azure
- **Capacités** : Déploiement, gestion du cycle de vie, RBAC
- **Usage** : Infrastructure as Code, gestion des ressources
- **Scope** : Pas un outil de monitoring

**Network Security Groups (NSG)**
- **Fonction** : Sécurité réseau (firewall)
- **Capacités** : Règles de filtrage de trafic
- **Usage** : Contrôle d'accès réseau
- **Limitation** : Sécurité uniquement, pas de monitoring

#### Cost Management et Budgets

** Configuration des budgets et actions automatiques :**

**Processus d'édition de budget :**
1. **Cost Management + Billing** → **Budgets**
2. **Éditer le budget** associé aux ressources du groupe de ressources
3. **Créer un nouveau Action Group** de type **Runbook**
4. **Choisir "Stop VM"** comme action

** Points clés identifiés :**
- **Cost analysis** : Ne peut pas arrêter automatiquement les VMs
- **Scale Up VM action group** : Non requis pour arrêter les VMs
- **Runbook type** : Obligatoire pour actions d'automatisation
- **Stop VM action** : Action spécifique pour arrêter les machines virtuelles

** Azure Advisor - Cost Optimization :**
- **Cost blade** : Optimisation et réduction des dépenses Azure
- **Identification** : VMs sous-utilisées
- **Performance blade** : Amélioration de la vitesse des applications
- **High availability** : Non disponible via Azure Advisor
- **Operational Excellence** : Efficacité des processus et workflows, gestion des ressources, meilleures pratiques de déploiement

**🎯 Concept clé identifié :** Target Resource pour alertes
- **VM Events/Syslog** → Target = **Log Analytics Workspace**
- **VM Metrics** → Target = **Virtual Machine**
- Les VMs envoient logs vers Log Analytics pour analyse

#### Diagnostic Settings - Collecte de Données de Diagnostic

**⚠️ Erreur Courante QCM : Destinations disponibles pour Diagnostic Settings**

**Vue d'ensemble :**
Les Diagnostic Settings permettent de collecter les **Platform Logs** et **Platform Metrics** des ressources Azure et de les envoyer vers différentes destinations pour analyse, archivage ou intégration.

**Types de données collectées :**
- **Activity Logs** : Opérations sur les ressources (création, suppression, modification)
- **Resource Logs** : Logs internes des ressources (ex: SQL queries, Storage operations)
- **Metrics** : Métriques de performance (CPU, Memory, Network, Storage)

**Destinations Disponibles - Vue Complète :**

| Destination | Use Case | Rétention | Coût | Requiert |
|-------------|----------|-----------|------|----------|
| **Log Analytics Workspace** | Analyse et requêtes KQL | Configurable (30-730 jours) | Ingestion + Rétention | Log Analytics Workspace |
| **Storage Account** | Archivage long terme | Illimitée | Stockage Blob | Storage Account |
| **Event Hub** | Streaming vers outils externes | Temps réel | Throughput | Event Hub Namespace |
| **Partner Solutions** | SIEM tiers (Splunk, Datadog) | Selon partenaire | Selon partenaire | Integration configurée |

**1. Log Analytics Workspace - Analyse et Alertes**

**⚠️ Destination PRINCIPALE pour l'examen AZ-104**

**Use Cases :**
- **Analyse avec KQL** : Requêtes complexes sur logs
- **Alertes** : Créer alertes basées sur logs
- **Dashboards** : Visualisations Azure Monitor
- **Workbooks** : Rapports interactifs
- **Insights** : VM Insights, Container Insights, Application Insights

**Configuration via Azure CLI :**
```bash
# Créer un Log Analytics Workspace
az monitor log-analytics workspace create \
  --resource-group myRG \
  --workspace-name myWorkspace \
  --location eastus \
  --retention-time 90

# Configurer Diagnostic Settings pour une VM
az monitor diagnostic-settings create \
  --name VMDiagnostics \
  --resource /subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Compute/virtualMachines/myVM \
  --workspace /subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.OperationalInsights/workspaces/myWorkspace \
  --logs '[{"category":"Administrative","enabled":true},{"category":"Security","enabled":true}]' \
  --metrics '[{"category":"AllMetrics","enabled":true}]'
```

**Configuration via PowerShell :**
```powershell
# Créer Log Analytics Workspace
New-AzOperationalInsightsWorkspace `
  -ResourceGroupName "myRG" `
  -Name "myWorkspace" `
  -Location "East US" `
  -Sku "PerGB2018" `
  -RetentionInDays 90

# Configurer Diagnostic Settings pour Storage Account
$storageAccount = Get-AzStorageAccount -ResourceGroupName "myRG" -Name "mystorageaccount"
$workspace = Get-AzOperationalInsightsWorkspace -ResourceGroupName "myRG" -Name "myWorkspace"

Set-AzDiagnosticSetting `
  -ResourceId $storageAccount.Id `
  -Name "StorageDiagnostics" `
  -WorkspaceId $workspace.ResourceId `
  -Enabled $true `
  -Category @("StorageWrite", "StorageRead", "StorageDelete")
```

**Requêtes KQL Utiles :**
```kusto
// Toutes les opérations de write sur Storage Account
StorageBlobLogs
| where OperationName == "PutBlob"
| project TimeGenerated, AccountName, Uri, StatusCode, CallerIpAddress

// VMs avec CPU > 80%
Perf
| where ObjectName == "Processor" and CounterName == "% Processor Time"
| where CounterValue > 80
| summarize avg(CounterValue) by Computer, bin(TimeGenerated, 5m)

// Activité administrative
AzureActivity
| where OperationNameValue contains "Microsoft.Compute/virtualMachines"
| where ActivityStatusValue == "Success"
| project TimeGenerated, Caller, OperationNameValue, ResourceGroup
```

**Limites et Coûts :**
- **Ingestion** : ~$2.50/GB (peut varier selon engagement)
- **Rétention** : Incluse jusqu'à 31 jours, puis ~$0.12/GB/mois
- **Requêtes** : Gratuites (incluses)
- **Alertes** : Coût selon fréquence d'évaluation

**2. Storage Account - Archivage Long Terme**

**Use Cases :**
- **Archivage** : Logs pour conformité (1-10 ans)
- **Audit** : Historique complet pour forensic
- **Coût réduit** : Alternative économique à Log Analytics
- **Backup logs** : Logs de sauvegarde et restauration

**Configuration via Azure CLI :**
```bash
# Configurer Diagnostic Settings vers Storage Account
az monitor diagnostic-settings create \
  --name VMDiagToStorage \
  --resource /subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Compute/virtualMachines/myVM \
  --storage-account /subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/mystorageaccount \
  --logs '[{"category":"Administrative","enabled":true,"retentionPolicy":{"enabled":true,"days":365}}]' \
  --metrics '[{"category":"AllMetrics","enabled":true,"retentionPolicy":{"enabled":true,"days":90}}]'
```

**Structure de Stockage :**
```
mystorageaccount
├── insights-logs-administrative/
│   ├── resourceId=/SUBSCRIPTIONS/{sub-id}/RESOURCEGROUPS/{rg}/PROVIDERS/{provider}/{resource}
│   │   ├── y=2025/m=10/d=27/h=12/m=00/
│   │   │   └── PT1H.json
```

**Format des Logs (JSON) :**
```json
{
  "time": "2025-10-27T12:00:00.000Z",
  "resourceId": "/subscriptions/.../resourceGroups/myRG/providers/Microsoft.Compute/virtualMachines/myVM",
  "category": "Administrative",
  "operationName": "Microsoft.Compute/virtualMachines/write",
  "resultType": "Success",
  "callerIpAddress": "203.0.113.50",
  "identity": {
    "authorization": {
      "action": "Microsoft.Compute/virtualMachines/write",
      "scope": "/subscriptions/.../resourceGroups/myRG"
    }
  }
}
```

**Lifecycle Management :**
```bash
# Créer lifecycle policy pour archiver après 90 jours
az storage account management-policy create \
  --account-name mystorageaccount \
  --resource-group myRG \
  --policy '{
    "rules": [{
      "name": "ArchiveLogs",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["insights-logs-"]
        },
        "actions": {
          "baseBlob": {
            "tierToCool": {"daysAfterModificationGreaterThan": 30},
            "tierToArchive": {"daysAfterModificationGreaterThan": 90}
          }
        }
      }
    }]
  }'
```

**Limites :**
- **Analyse** : Pas de requêtes KQL (nécessite téléchargement)
- **Alertes** : Pas d'alertes en temps réel
- **Coût** : ~$0.02/GB/mois (Hot), ~$0.01/GB/mois (Cool), ~$0.002/GB/mois (Archive)

**3. Event Hub - Streaming en Temps Réel**

**Use Cases :**
- **SIEM Integration** : Splunk, QRadar, ArcSight
- **Custom Processing** : Azure Functions, Stream Analytics
- **Multi-destination** : Plusieurs consommateurs en parallèle
- **Near real-time** : Latence < 1 minute

**Configuration via Azure CLI :**
```bash
# Créer Event Hub Namespace
az eventhubs namespace create \
  --resource-group myRG \
  --name myEventHubNS \
  --location eastus \
  --sku Standard

# Créer Event Hub
az eventhubs eventhub create \
  --resource-group myRG \
  --namespace-name myEventHubNS \
  --name diagnostics-hub \
  --partition-count 4 \
  --message-retention 7

# Configurer Diagnostic Settings vers Event Hub
az monitor diagnostic-settings create \
  --name VMDiagToEventHub \
  --resource /subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Compute/virtualMachines/myVM \
  --event-hub myEventHubNS \
  --event-hub-rule /subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.EventHub/namespaces/myEventHubNS/authorizationRules/RootManageSharedAccessKey \
  --logs '[{"category":"Administrative","enabled":true}]' \
  --metrics '[{"category":"AllMetrics","enabled":true}]'
```

**Consumer Example (Azure Function) :**
```csharp
[FunctionName("ProcessDiagnosticLogs")]
public static async Task Run(
    [EventHubTrigger("diagnostics-hub", Connection = "EventHubConnection")] EventData[] events,
    ILogger log)
{
    foreach (var eventData in events)
    {
        string messageBody = Encoding.UTF8.GetString(eventData.Body.Array);
        var logEntry = JsonConvert.DeserializeObject<DiagnosticLog>(messageBody);
        
        // Custom processing
        if (logEntry.Category == "Security" && logEntry.ResultType == "Failure")
        {
            await SendAlertToSlack(logEntry);
        }
    }
}
```

**Limites et Coûts :**
- **Throughput** : 1 MB/s par partition (Standard), 20 MB/s (Premium)
- **Rétention** : 1-7 jours (Standard), jusqu'à 90 jours (Premium)
- **Coût** : ~$0.028/million events + $11/throughput unit/mois

**4. Partner Solutions - SIEM Tiers**

**Partenaires Disponibles :**
- **Datadog** : APM et infrastructure monitoring
- **Elastic (Elasticsearch)** : Search, analytics, visualization
- **LogRhythm** : SIEM et threat detection
- **Splunk** : Enterprise SIEM
- **Sumo Logic** : Cloud-native SIEM

**Configuration :**
```bash
# Liste des partenaires disponibles
az monitor diagnostic-settings subscription list-categories

# Configuration vers Partner Solution (exemple Datadog)
az monitor diagnostic-settings create \
  --name VMDiagToDatadog \
  --resource /subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Compute/virtualMachines/myVM \
  --marketplace-partner-id /subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Datadog/monitors/myDatadogMonitor \
  --logs '[{"category":"Administrative","enabled":true}]' \
  --metrics '[{"category":"AllMetrics","enabled":true}]'
```

**Avantages :**
- **Corrélation multi-cloud** : Logs Azure + AWS + GCP
- **Threat intelligence** : Détection avancée de menaces
- **Compliance** : Rapports de conformité pré-configurés
- **Expertise** : Support spécialisé SIEM

**Configuration Multi-Destinations**

**⚠️ Point Important pour l'Examen :**
Une ressource peut avoir **plusieurs Diagnostic Settings** envoyant vers **différentes destinations simultanément**.

**Exemple - Architecture Complète :**
```bash
# Diagnostic Setting 1 : Analyse temps réel
az monitor diagnostic-settings create \
  --name RealTimeAnalytics \
  --resource $vmId \
  --workspace $logAnalyticsId \
  --logs '[{"category":"Administrative","enabled":true}]'

# Diagnostic Setting 2 : Archivage long terme
az monitor diagnostic-settings create \
  --name LongTermArchive \
  --resource $vmId \
  --storage-account $storageAccountId \
  --logs '[{"category":"Administrative","enabled":true,"retentionPolicy":{"enabled":true,"days":2555}}]'

# Diagnostic Setting 3 : Streaming vers SIEM
az monitor diagnostic-settings create \
  --name SIEMIntegration \
  --resource $vmId \
  --event-hub $eventHubNSId \
  --logs '[{"category":"Security","enabled":true}]'
```

**Architecture Typique - DevOps :**
```
Azure Resource (VM, Storage, NSG, etc.)
├── Diagnostic Setting 1 → Log Analytics Workspace
│   ├── Requêtes KQL
│   ├── Alertes temps réel
│   └── Dashboards Azure Monitor
│
├── Diagnostic Setting 2 → Storage Account
│   ├── Archivage 7 ans (conformité)
│   ├── Lifecycle policy (Cool → Archive)
│   └── Backup des logs
│
├── Diagnostic Setting 3 → Event Hub
│   ├── Azure Function (processing custom)
│   ├── Stream Analytics
│   └── Splunk/Datadog Integration
│
└── Diagnostic Setting 4 → Partner Solution (Datadog)
    ├── APM
    ├── Infrastructure monitoring
    └── Multi-cloud correlation
```

**Scénarios d'Examen - Diagnostic Settings**

| Besoin | Destination | Raison |
|--------|-------------|--------|
| **Créer alertes sur logs VM** | Log Analytics Workspace | Permet requêtes KQL et alertes |
| **Archiver logs 5 ans pour audit** | Storage Account | Coût faible, rétention illimitée |
| **Envoyer logs vers Splunk** | Event Hub OU Partner Solution | Streaming temps réel ou intégration native |
| **Analyser activité administrative** | Log Analytics Workspace | Requêtes et dashboards |
| **Conformité RGPD (7 ans logs)** | Storage Account | Archivage long terme économique |
| **Multi-cloud SIEM** | Partner Solution (Datadog, Splunk) | Corrélation Azure + AWS + on-prem |

**Catégories de Logs Disponibles par Type de Ressource**

**Virtual Machines :**
- Performance counters (métriques CPU, Memory, Disk, Network)
- Event logs (Windows) / Syslog (Linux)
- IIS logs (si IIS installé)
- Nécessite **VM agent** ou **Azure Monitor Agent**

**Storage Account :**
- StorageRead, StorageWrite, StorageDelete
- Transaction logs (blob, file, queue, table)
- Metrics (availability, latency, capacity)

**Network Security Group (NSG) :**
- NSG Flow Logs (version 1 ou 2)
- Network Watcher requis

**Azure SQL Database :**
- SQLInsights, QueryStoreRuntimeStatistics
- Errors, Timeouts, Deadlocks
- Automatic tuning, Intelligent insights

**App Service :**
- AppServiceHTTPLogs, AppServiceConsoleLogs
- AppServiceAppLogs, AppServiceAuditLogs
- AppServicePlatformLogs

**Best Practices - Diagnostic Settings**

✅ **À FAIRE :**
- **Log Analytics** pour toutes les ressources critiques (analyse + alertes)
- **Storage Account** pour archivage conformité (LRS ou GRS selon criticité)
- **Rétention appropriée** : 90 jours Log Analytics, 7 ans Storage
- **Multi-destinations** : Log Analytics (analyse) + Storage (archive)
- **Catégories sélectives** : N'activer que logs nécessaires (coûts)
- **Lifecycle policies** : Archiver logs anciens (Cool/Archive tier)
- **RBAC strict** : Limiter accès aux logs sensibles

❌ **À ÉVITER :**
- Activer tous les logs sans distinction (coûts élevés)
- Oublier la rétention (défaut = 30 jours Log Analytics)
- Storage Account sans lifecycle policy (coûts croissants)
- Event Hub pour archivage (rétention limitée)
- Multiple Diagnostic Settings avec mêmes logs (duplication coûts)

**PowerShell - Gestion Complète :**
```powershell
# Lister Diagnostic Settings d'une ressource
Get-AzDiagnosticSetting -ResourceId $vmId

# Supprimer un Diagnostic Setting
Remove-AzDiagnosticSetting `
  -ResourceId $vmId `
  -Name "VMDiagnostics"

# Exporter configuration Diagnostic Settings
Get-AzDiagnosticSetting -ResourceId $vmId |
  ConvertTo-Json -Depth 10 |
  Out-File "diagnostic-settings-backup.json"
```

**Troubleshooting Diagnostic Settings**

| Problème | Cause Possible | Solution |
|----------|----------------|----------|
| Logs n'arrivent pas dans Log Analytics | Agent non installé (VM) | Installer Azure Monitor Agent |
| Storage Account ne reçoit pas de logs | Permissions insuffisantes | Vérifier RBAC (Monitoring Contributor) |
| Event Hub sans données | Authorization rule incorrecte | Utiliser RootManageSharedAccessKey |
| Coûts élevés | Tous les logs activés | Sélectionner catégories pertinentes uniquement |
| Rétention trop courte | Défaut 30 jours | Configurer rétention 90-730 jours |

#### Types d'Alertes

**Metric Alerts**
- **Seuils numériques** : CPU > 80%
- **Near real-time** : Évaluation fréquente
- **Multiple dimensions** : Filtrage par propriétés

**Log Alerts**
- **KQL queries** : Requêtes sur logs
- **Example** : `Event | search "error"`
- **Flexibility** : Logique complexe possible

**Activity Log Alerts**
- **Administrative events** : Création/suppression ressources
- **Service Health** : Incidents Azure
- **Resource Health** : État des ressources

#### Requêtes KQL Essentielles
```kusto
// Rechercher dans une table spécifique
Event | search "error"

// Compter par computer
Event | summarize count() by Computer

// Filtrer par niveau
Event | where EventLevelName == "Error"

// Timeline des événements
Event | where TimeGenerated > ago(1h) | sort by TimeGenerated desc
```

### 5.2 Azure Backup

#### Recovery Services Vault

** Règles critiques identifiées :**

**Localisation :**
- Vault et ressources **dans la même région**
- Example : Vault2 (West US) peut sauvegarder Storage1 (West US)
- Share1 sauvegardable car dans Storage1 même région

**Suppression du Vault :**
1. **D'abord** : Arrêter backup de tous les éléments protégés
2. **Ensuite** : Supprimer le vault
3. **Erreur** : Vault ne peut pas être supprimé avec éléments protégés

#### Backup Policies

** Limites par type de ressource :**
- **VMs** : Maximum 100 VMs par policy
- **SQL Databases** : Policy séparée requise
- **File Shares** : Policy séparée requise

**Example :** 100 VMs + 20 SQL + 50 Files = **3 policies minimum**

#### Changement de Vault

**Pour Backup :**
1. Arrêter backup dans vault actuel (RSV1)
2. Configurer backup dans nouveau vault (RSV2)

**Pour Site Recovery :**
- VM → Disaster Recovery → Replication Settings → Nouveau vault

#### Types de Backup
- **Azure VMs** : Snapshots dans région source
- **On-premises** : MARS agent ou DPM/MABS
- **SQL in Azure VM** : Application-consistent backups
- **Azure Files** : Snapshots au niveau share

### 5.3 Azure Site Recovery

#### Supported Scenarios
- **Azure to Azure** : VMs entre régions Azure
- **VMware to Azure** : VMs VMware vers Azure
- **Hyper-V to Azure** : VMs Hyper-V vers Azure
- **Physical to Azure** : Serveurs physiques vers Azure

#### Components
- **Recovery Services Vault** : Orchestration et management
- **Configuration Server** : Pour VMware (on-premises)
- **Process Server** : Réplication data processing
- **Master Target** : Receive replication data

#### Recovery Plans
- **Grouping** : VMs dans groupes logiques
- **Sequencing** : Ordre de démarrage
- **Scripts** : Automation pendant failover
- **Manual actions** : Actions manuelles requises

---

## Tips Pratiques d'Examen - Insights des Questions Réelles

### 4.5 Pièges Fréquents et Solutions

#### VNet Peering - Erreurs de Plages d'Adresses
** Piège identifié :** Confusion entre plages chevauchantes et non-chevauchantes
- **Erreur courante** : Essayer de peerer 192.168.0.0/24 avec 192.168.0.0/16
- **Raison** : /24 est inclus dans /16 → chevauchement détecté par Azure
- **Solution** : Utiliser des plages complètement différentes (10.x.x.x vs 172.x.x.x)
- **Validation** : Azure bloque automatiquement les peerings avec chevauchement

#### NSG - Priorités et Évaluation en Cascade
** Piège identifié :** Oublier l'évaluation en cascade Subnet → NIC
- **Erreur courante** : NSG Subnet Allow + NSG NIC Deny = Trafic bloqué
- **Raison** : Les deux niveaux doivent autoriser le trafic
- **Solution** : Vérifier les NSG aux deux niveaux lors du troubleshooting
- **Optimisation** : Un seul NSG Deny à n'importe quel niveau bloque tout

#### Load Balancer - Session Persistence vs NAT Rules
** Piège identifié :** Confusion entre session persistence et NAT rules
- **Erreur courante** : Utiliser NAT rules pour maintenir les sessions utilisateur
- **Raison** : NAT rules = redirection de trafic, Session persistence = maintien de session
- **Solution** : Client IP + Protocol pour les applications avec état
- **Cas d'usage** : E-commerce, applications avec paniers, sessions utilisateur

#### Diagnostic Réseau - Outils Spécialisés
** Piège identifié :** Utiliser les mauvais outils pour le diagnostic
- **Erreur courante** : `Get-AzVirtualNetworkUsageList` pour diagnostic de ports
- **Raison** : PowerShell Azure ≠ outils de diagnostic réseau
- **Solution** : `netstat -an` pour ports d'écoute, `Test-NetConnection` pour connectivité
- **Règle** : Diagnostic réseau = outils système Windows, pas cmdlets Azure

### 4.6 Matrice de Décision Rapide

| Problème | Diagnostic | Solution | Outil/Commande |
|----------|------------|----------|----------------|
| VNets ne communiquent pas | Vérifier plages d'adresses | VNet Peering | Portail Azure → Peerings |
| Trafic bloqué | Vérifier NSG Subnet + NIC | Ajuster priorités | NSG → Rules → Priority |
| Sessions perdues | Vérifier session persistence | Client IP + Protocol | Load Balancer → Settings |
| Ports d'écoute | Diagnostic réseau | Vérifier services | `netstat -an` |
| Connectivité Internet | Test de connectivité | Vérifier NSG outbound | `Test-NetConnection` |

---

## Points Critiques Basés sur Vos Erreurs

### 1. Log Analytics = Hub Central pour Monitoring
** Erreur courante :** Choisir la VM comme target resource
** Correct :** Log Analytics Workspace pour toutes les alertes de logs
- Windows Event Logs → Log Analytics
- Linux Syslog → Log Analytics
- VM metrics → VM directement

### 2. Règle de Même Région
** Erreur :** Vault et Storage dans régions différentes
** Correct :** Toujours même région pour :
- Recovery Services Vault + Storage Account
- Traffic Analytics components
- Backup sources et destinations

### 3. Disque D: = Temporaire et Volatil
** Erreur :** Stocker des données importantes sur D:
** Correct :** D: pour cache/temp uniquement
- C: = Persistant (OS, apps)
- D: = Temporaire (perdu lors maintenance)
- E:, F: = Persistants (données)

### 4. Storage Account Types et Limitations
** Erreur :** Premium File Shares sur StorageV2
** Correct :** FileStorage accounts uniquement
- StorageV2 = Standard files uniquement
- BlobStorage = Pas de files du tout

### 5. NSG Sharing et Optimisation
** Erreur :** Un NSG par VM
** Correct :** Un NSG partagé si mêmes règles
- 5 VMs = 5 NICs + 1 NSG (optimal)

### 6. Recovery Services Vault Management
** Erreur :** Essayer supprimer vault avec backups actifs
** Correct :** Toujours arrêter backups d'abord
- Stop backup → Delete vault
- Change vault = Stop + Start elsewhere

### 7. App Service Deployment Workflow
** Erreur :** Deploy direct en production
** Correct :** Deploy → Test → Swap
- Staging slot pour tests
- Production swap pour zero downtime

### 8. Root Management Group Access
** Erreur :** Accès direct au root MG
** Correct :** Global Admin + elevation required
- Aucun accès par défaut
- Global Admin doit s'élever

### 9. Azure Bastion vs Remote Desktop
** Erreur :** Exposer les ports RDP/SSH sur Internet
** Correct :** Utiliser Azure Bastion
- Azure Bastion = Accès via navigateur, pas d'exposition de ports
- Remote Desktop = Ports exposés, vulnérable aux attaques
- Azure Monitor et Network Watcher = Pas de connectivité aux VMs

### 10. Private DNS Zone Association
** Erreur :** Essayer d'utiliser directement une Private DNS Zone
** Correct :** Créer un Virtual Network Link
- Virtual Network Link = Associe VNet à Private DNS Zone
- DNS Private Resolver = Proxy pour on-premises ↔ Azure
- Custom DNS Server = Complexe et ne fonctionne pas avec Private DNS Zones

### 11. Network Watcher IP Flow Verify
** Erreur :** Utiliser NSG Flow Logs ou Packet Capture d'abord
** Correct :** IP Flow Verify pour diagnostic rapide
- IP Flow Verify = Identifie le NSG bloquant directement
- NSG Flow Logs = Configuration complexe, analyse manuelle
- Packet Capture = Détails mais pas d'identification NSG

### 12. Load Balancer Troubleshooting
** Erreur :** Modifier session persistence ou timeout pour résoudre connectivité
** Correct :** Vérifier Health Probes, Ports, NSG rules
- Health Probes = Configuration et réponses des VMs
- Port Listening = Applications écoutent sur les bons ports
- NSG Rules = Autoriser trafic au niveau subnet ET NIC
- Session persistence/timeout = Ne résolvent pas les problèmes de base

---

## Checklist Final d'Examen

### Identities and Governance
- [ ] Dynamic group rules syntax : `(user.property -eq "value")`
- [ ] Custom domain DNS records : TXT ou MX
- [ ] Root MG access : Global Admin + elevation
- [ ] RBAC scopes : MG → Subscription → RG → Resource
- [ ] **User Administrator** : Gestion utilisateurs et groupes (moindre privilège)
- [ ] **Global Administrator** : Toutes permissions (à utiliser avec parcimonie)

### Storage
- [ ] FileStorage pour Premium files uniquement
- [ ] Import/Export destinations : Blob + Files (5TB max)
- [ ] Replication types par account type
- [ ] Port 445 pour Azure Files SMB
- [ ] **Data Lake Storage Gen2** : Hierarchical Namespace irréversible
- [ ] **ACLs POSIX** : Permissions granulaires + RBAC (le plus restrictif s'applique)
- [ ] **ABFS protocol** : abfs:// ou abfss:// pour Hadoop/Spark
- [ ] **Storage Blob Data Owner** : Seul rôle pour modifier ACLs
- [ ] **Parquet format** : Format recommandé pour Big Data analytics

### Compute
- [ ] Disque D: temporaire, C: persistant
- [ ] VMSS pour high availability pendant maintenance
- [ ] Template deployment : Resource Group configurable
- [ ] Availability Zones : Managed disks requis

### Networking
- [ ] NSG sharing entre ressources
- [ ] DNS interne : vm-name.internal.cloudapp.net
- [ ] Traffic Analytics : Log Analytics + Storage Account
- [ ] Connection Monitor pour RTT measurements
- [ ] **VNet Peering** : Plages d'adresses non-chevauchantes obligatoires
- [ ] **NSG Priorities** : Plus bas = plus prioritaire (100 < 200)
- [ ] **Session Persistence** : Client IP + Protocol pour sticky sessions
- [ ] **Commandes réseau** : `netstat -an` pour ports d'écoute
- [ ] **Azure Bastion** : Pas d'exposition RDP/SSH, accès via navigateur
- [ ] **Private DNS Zone** : Virtual Network Link pour associer un VNet
- [ ] **DNS Private Resolver** : Proxy DNS entre on-premises et Azure
- [ ] **IP Flow Verify** : Identifier le NSG bloquant la communication
- [ ] **Load Balancer Troubleshooting** : Health probes, ports, NSG rules

### Monitoring & Backup
- [ ] Log Analytics Workspace comme target pour VM alerts
- [ ] Backup policies : 100 VMs max par policy
- [ ] Recovery Services Vault : même région
- [ ] Stop backup avant delete vault
- [ ] KQL syntax : `Table | search "term"`
- [ ] **Azure Monitor** : Maximiser disponibilité et performance des applications
- [ ] **Network Watcher** : Monitoring réseau uniquement (pas connectivité VMs)
- [ ] **NSG vs Monitoring** : NSG = sécurité, pas monitoring

---

##  Ressources d'Étude Recommandées

### Documentation Microsoft
- Azure Architecture Center
- Azure Well-Architected Framework
- Azure Best Practices

### Labs Pratiques
- Microsoft Learn modules
- Azure free account (12 mois)
- Hands-on labs

### Examens Blancs
- MeasureUp practice tests
- Whizlabs Azure exams
- Tutorials Dojo practice tests

**Temps de préparation recommandé :** 40-60 heures d'étude + practice labs
