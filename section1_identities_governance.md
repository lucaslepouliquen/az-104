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
  - [Conditional Access](#conditional-access)
- [1.2 Managed Identities](#12-managed-identities)
  - [System-Assigned Managed Identity](#system-assigned-managed-identity)
  - [User-Assigned Managed Identity](#user-assigned-managed-identity)
  - [Comparaison et Use Cases](#comparaison-et-use-cases)
- [1.3 Role-Based Access Control (RBAC)](#13-role-based-access-control-rbac)
  - [Rôles Built-in Essentiels](#rôles-built-in-essentiels)
  - [Rôles Administratifs Azure AD](#rôles-administratifs-azure-ad)
  - [Rôles Spécialisés](#rôles-spécialisés)
  - [Scopes d'assignation RBAC - Détaillé](#scopes-dassignation-rbac---détaillé)
- [1.4 Azure Policy](#14-azure-policy)
  - [Concepts Clés](#concepts-clés)
  - [Effects Principaux - Détaillé](#effects-principaux---détaillé)
  - [Built-in Policies Courantes](#built-in-policies-courantes)
- [1.5 Management Groups](#15-management-groups)
  - [Hiérarchie](#hiérarchie)
  - [Limites](#limites)
  - [Resource Locks](#resource-locks)
- [1.6 Azure Blueprints](#16-azure-blueprints)
  - [Concepts et Architecture](#concepts-et-architecture)
  - [Composants d'un Blueprint](#composants-dun-blueprint)
  - [Lifecycle et Versioning](#lifecycle-et-versioning)
  - [Blueprints vs ARM Templates vs Policy](#blueprints-vs-arm-templates-vs-policy)

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

### Conditional Access

**⚠️ Concept Clé pour AZ-104 : Conditional Access permet de contrôler l'accès aux ressources cloud en fonction de conditions spécifiques**

**Définition :**
- **Conditional Access** : Outil de sécurité Azure AD qui évalue les signaux (utilisateur, localisation, appareil) pour prendre des décisions d'accès automatisées
- **Fonctionnalité** : Azure AD Premium P1 ou P2 requis
- **Application** : Appliqué AVANT l'accès aux applications cloud

**Architecture des Politiques Conditional Access :**

```
Signal (IF) → Decision (THEN) → Enforcement
   ↓              ↓                ↓
Qui/Où/Quoi → Évaluation → Bloquer/MFA/Autoriser
```

**Composants d'une Politique Conditional Access :**

**1. Assignments (Qui et Où) - Conditions "IF"**

**Users and Groups :**
- ✅ Utilisateurs spécifiques
- ✅ Groupes (Security Groups)
- ✅ Rôles d'annuaire (Directory roles)
- ✅ Guest users
- ❌ **Exclude** : Comptes d'urgence (break-glass accounts)

**Cloud apps or actions :**
- ✅ Toutes les applications cloud
- ✅ Applications spécifiques (Office 365, Azure Portal, etc.)
- ✅ Actions utilisateur (Register security info)

**Conditions :**
- **Sign-in risk** : Basé sur Azure AD Identity Protection (P2)
- **Device platforms** : Windows, iOS, Android, macOS
- **Locations** : Pays, régions, adresses IP
- **Client apps** : Browser, mobile apps, desktop clients
- **Device state** : Compliant, Hybrid Azure AD joined

**2. Access Controls (Que faire) - Actions "THEN"**

**Grant Controls (Accorder l'accès) :**
- ✅ **Require multi-factor authentication** : MFA obligatoire
- ✅ **Require device to be marked as compliant** : Intune compliance
- ✅ **Require Hybrid Azure AD joined device** : Appareil joint au domaine
- ✅ **Require approved client app** : Applications mobiles approuvées
- ✅ **Require app protection policy** : Intune App Protection
- ❌ **Block access** : Bloquer complètement

**Session Controls :**
- **Sign-in frequency** : Forcer réauthentification (ex: toutes les 4h)
- **Persistent browser session** : Rester connecté
- **App enforced restrictions** : Limiter fonctionnalités (ex: SharePoint read-only)
- **Conditional Access App Control** : Microsoft Cloud App Security monitoring

**Scénarios Pratiques - Pour l'Examen :**

**Scénario 1 - Bloquer accès hors du réseau d'entreprise :**
```
IF:
- Users: All users
- Cloud apps: Office 365
- Locations: Any location EXCEPT Corporate Network

THEN:
- Grant: Block access
```

**Scénario 2 - Exiger MFA pour administrateurs :**
```
IF:
- Users: Directory role = Global Administrator
- Cloud apps: All cloud apps
- Locations: Any location

THEN:
- Grant: Require multi-factor authentication
```

**Scénario 3 - Exiger appareil compliant pour accès mobile :**
```
IF:
- Users: All users
- Cloud apps: Office 365
- Device platforms: iOS, Android

THEN:
- Grant: Require device to be marked as compliant
```

**Scénario 4 - Bloquer accès depuis certains pays :**
```
IF:
- Users: All users
- Cloud apps: Azure Portal
- Locations: High-risk countries (ex: pays X, Y, Z)

THEN:
- Grant: Block access
```

**⚠️ Configuration via Azure Portal :**

```bash
# Navigation dans le portail
Azure AD → Security → Conditional Access → New policy

# Ou via PowerShell (Azure AD Premium P1/P2 requis)
# Note: Conditional Access nécessite le module AzureAD
Connect-AzureAD

# Créer une politique (exemple conceptuel - syntaxe simplifiée)
New-AzureADMSConditionalAccessPolicy -DisplayName "Require MFA for Admins" `
  -State "Enabled" `
  -Conditions @{
    Users = @{
      IncludeRoles = @("62e90394-69f5-4237-9190-012177145e10") # Global Admin
    }
  } `
  -GrantControls @{
    BuiltInControls = @("mfa")
  }
```

**⚠️ Best Practices Conditional Access :**

**1. Report-Only Mode (Recommandé pour test) :**
- **Tester AVANT** de mettre en production
- **Éviter** de bloquer accidentellement les utilisateurs
- **Analyser** les logs avant activation

```
Policy State Options:
- Report-only: Log uniquement, pas d'enforcement
- On: Actif, enforcement complet
- Off: Désactivé
```

**2. Exclude Break-Glass Accounts :**
- **Toujours exclure** au moins 2 comptes d'urgence
- **Éviter** de se verrouiller hors du tenant
- **Utiliser** des comptes séparés avec MFA alternatif

**3. Require Multiple Controls :**
```
Grant Controls Options:
- Require ALL the selected controls (AND logic)
- Require ONE of the selected controls (OR logic)
```

**⚠️ Erreurs Courantes QCM :**

| Question | Réponse Incorrecte ❌ | Réponse Correcte ✅ |
|----------|----------------------|---------------------|
| **"Comment bloquer accès hors du bureau ?"** | "Créer une policy avec Grant: MFA" | "Créer une policy avec Locations + Block access" |
| **"Conditional Access est gratuit ?"** | "Oui, inclus dans Azure AD Free" | "Non, nécessite Azure AD Premium P1" |
| **"Peut-on utiliser distribution groups ?"** | "Oui, tous les types de groupes" | "Non, uniquement Security Groups et M365 Groups" |
| **"MFA suffit pour appareil non compliant ?"** | "Oui, MFA = sécurité suffisante" | "Non, utiliser 'Require compliant device'" |

**Prérequis Techniques :**

| Fonctionnalité | License Requise | Notes |
|---------------|----------------|-------|
| **Conditional Access basics** | Azure AD Premium P1 | Policies, MFA, device compliance |
| **Sign-in risk-based policies** | Azure AD Premium P2 | Identity Protection requis |
| **User risk-based policies** | Azure AD Premium P2 | Identity Protection requis |
| **Report-only mode** | Azure AD Premium P1 | Test sans enforcement |

**Monitoring et Troubleshooting :**

```bash
# Vérifier les sign-ins avec Conditional Access
# Azure Portal → Azure AD → Sign-in logs → Filter by Conditional Access

# Colonnes importantes :
# - Conditional Access: Success/Failure/Not Applied
# - Applied policies: Liste des policies évaluées
# - Result: Grant/Block/Require MFA
```

**Intégration avec d'autres services :**
- **Azure AD Identity Protection** : Risk-based policies (P2)
- **Microsoft Intune** : Device compliance
- **Microsoft Defender for Cloud Apps** : Session monitoring
- **Azure AD Privileged Identity Management (PIM)** : Protection des rôles privilégiés

**⚠️ Points Clés pour l'Examen :**
- ✅ Conditional Access = **"IF-THEN" policies**
- ✅ Nécessite **Azure AD Premium P1 minimum**
- ✅ Toujours utiliser **Report-only mode** avant activation
- ✅ **Exclure break-glass accounts** de toutes les policies
- ✅ **Locations** peuvent être Named Locations (IP ranges)
- ✅ **MFA** est un Grant Control, pas un Session Control
- ✅ **Block access** empêche complètement l'accès

## 1.2 Managed Identities

**⚠️ Concept Clé pour AZ-104 : Managed Identities permettent aux ressources Azure de s'authentifier auprès d'autres services sans gérer de credentials**

**Définition :**
- **Managed Identity** : Identité Azure AD automatiquement gérée par Azure
- **Avantage Principal** : Pas de credentials dans le code, rotation automatique des secrets
- **Cas d'usage** : VMs, App Services, Functions accédant à Key Vault, Storage, SQL, etc.

**Principe de Fonctionnement :**

```
Application avec Managed Identity
    ↓ (demande token via IMDS/Azure Instance Metadata Service)
Azure AD
    ↓ (fournit token d'accès)
Azure Resource (Key Vault, Storage, SQL)
    ↓ (valide token et autorise accès via RBAC)
Accès accordé
```

### System-Assigned Managed Identity

**Caractéristiques :**
- **Lifecycle** : Lié à la ressource Azure (VM, App Service, etc.)
- **Création** : Automatique lors de l'activation sur la ressource
- **Suppression** : Automatique lors de la suppression de la ressource
- **1:1 Mapping** : Une identité par ressource uniquement
- **Scope** : Limitée à une seule ressource

**Activation - System-Assigned :**

```bash
# Via Azure CLI - Activer sur une VM
az vm identity assign \
  --resource-group myResourceGroup \
  --name myVM

# Output:
# {
#   "systemAssignedIdentity": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
#   "type": "SystemAssigned"
# }

# Via Azure CLI - Activer sur App Service
az webapp identity assign \
  --resource-group myResourceGroup \
  --name myWebApp

# Via PowerShell - Activer sur une VM
Update-AzVM -ResourceGroupName myResourceGroup \
  -VM (Get-AzVM -ResourceGroupName myResourceGroup -Name myVM) \
  -IdentityType SystemAssigned
```

**Utilisation dans le code - System-Assigned :**

```python
# Python - Accéder à Key Vault avec System-Assigned MI
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

# DefaultAzureCredential détecte automatiquement la Managed Identity
credential = DefaultAzureCredential()
client = SecretClient(vault_url="https://myvault.vault.azure.net", credential=credential)

secret = client.get_secret("mySecret")
print(secret.value)
```

```csharp
// C# - Accéder à Azure SQL avec System-Assigned MI
using Azure.Identity;
using Microsoft.Data.SqlClient;

var credential = new DefaultAzureCredential();
var token = await credential.GetTokenAsync(
    new Azure.Core.TokenRequestContext(new[] { "https://database.windows.net/.default" }));

using var connection = new SqlConnection("Server=myserver.database.windows.net;Database=mydb;");
connection.AccessToken = token.Token;
await connection.OpenAsync();
```

**Assigner des permissions RBAC - System-Assigned :**

```bash
# 1. Récupérer le Principal ID de la VM
principalId=$(az vm identity show \
  --resource-group myResourceGroup \
  --name myVM \
  --query principalId --output tsv)

# 2. Assigner le rôle "Key Vault Secrets User" à la Managed Identity
az role assignment create \
  --assignee $principalId \
  --role "Key Vault Secrets User" \
  --scope "/subscriptions/xxx/resourceGroups/myRG/providers/Microsoft.KeyVault/vaults/myVault"

# 3. Assigner accès à un Storage Account
az role assignment create \
  --assignee $principalId \
  --role "Storage Blob Data Contributor" \
  --scope "/subscriptions/xxx/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/myStorage"
```

**⚠️ Avantages System-Assigned :**
- ✅ **Simple** : Activer en un clic/commande
- ✅ **Lifecycle automatique** : Créé/supprimé avec la ressource
- ✅ **Sécurisé** : Pas de credentials à gérer
- ✅ **Audit** : Identity intégrée à la ressource

**❌ Limitations System-Assigned :**
- ❌ **1:1 uniquement** : Ne peut pas être partagée entre ressources
- ❌ **Suppression** : Identity supprimée si ressource supprimée
- ❌ **Réutilisation impossible** : Nouvelle identity à chaque recréation

### User-Assigned Managed Identity

**Caractéristiques :**
- **Lifecycle** : Indépendant des ressources Azure
- **Création** : Ressource Azure standalone (comme une VM ou Storage Account)
- **Suppression** : Manuelle, survit à la suppression des ressources
- **1:N Mapping** : Peut être partagée entre plusieurs ressources
- **Scope** : Peut être utilisée par plusieurs VMs, App Services, etc.

**Création - User-Assigned :**

```bash
# 1. Créer une User-Assigned Managed Identity
az identity create \
  --resource-group myResourceGroup \
  --name myUserAssignedIdentity

# Output: Noter le 'id' et 'principalId'
# {
#   "id": "/subscriptions/xxx/resourceGroups/myRG/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myUserAssignedIdentity",
#   "principalId": "yyyyyyyy-yyyy-yyyy-yyyy-yyyyyyyyyyyy",
#   "clientId": "zzzzzzzz-zzzz-zzzz-zzzz-zzzzzzzzzzzz"
# }

# 2. Récupérer l'ID de la Managed Identity
identityId=$(az identity show \
  --resource-group myResourceGroup \
  --name myUserAssignedIdentity \
  --query id --output tsv)
```

**Assignation à des ressources - User-Assigned :**

```bash
# Assigner à une VM
az vm identity assign \
  --resource-group myResourceGroup \
  --name myVM \
  --identities $identityId

# Assigner à une autre VM (réutilisation de la même identity)
az vm identity assign \
  --resource-group myResourceGroup \
  --name myVM2 \
  --identities $identityId

# Assigner à un App Service
az webapp identity assign \
  --resource-group myResourceGroup \
  --name myWebApp \
  --identities $identityId

# Assigner à Azure Container Instances
az container create \
  --resource-group myResourceGroup \
  --name myContainer \
  --image myimage:latest \
  --assign-identity $identityId
```

**Assigner des permissions RBAC - User-Assigned :**

```bash
# Récupérer le Principal ID de la User-Assigned MI
principalId=$(az identity show \
  --resource-group myResourceGroup \
  --name myUserAssignedIdentity \
  --query principalId --output tsv)

# Assigner rôle Key Vault
az role assignment create \
  --assignee $principalId \
  --role "Key Vault Secrets User" \
  --scope "/subscriptions/xxx/resourceGroups/myRG/providers/Microsoft.KeyVault/vaults/myVault"

# Assigner rôle Storage Blob
az role assignment create \
  --assignee $principalId \
  --role "Storage Blob Data Reader" \
  --scope "/subscriptions/xxx/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/myStorage"
```

**Utilisation dans le code - User-Assigned :**

```python
# Python - Spécifier explicitement User-Assigned MI par Client ID
from azure.identity import ManagedIdentityCredential
from azure.keyvault.secrets import SecretClient

# Utiliser le Client ID de la User-Assigned MI
credential = ManagedIdentityCredential(client_id="zzzzzzzz-zzzz-zzzz-zzzz-zzzzzzzzzzzz")
client = SecretClient(vault_url="https://myvault.vault.azure.net", credential=credential)

secret = client.get_secret("mySecret")
```

```csharp
// C# - Spécifier User-Assigned MI
using Azure.Identity;

var credential = new ManagedIdentityCredential(clientId: "zzzzzzzz-zzzz-zzzz-zzzz-zzzzzzzzzzzz");
var client = new SecretClient(new Uri("https://myvault.vault.azure.net"), credential);
```

**⚠️ Avantages User-Assigned :**
- ✅ **Réutilisable** : Partagée entre plusieurs ressources (VMs, App Services, Functions)
- ✅ **Lifecycle indépendant** : Survit à la suppression des ressources
- ✅ **Centralisation** : Une seule identity avec permissions centralisées
- ✅ **Flexibilité** : Peut être détachée/réassignée

**❌ Limitations User-Assigned :**
- ❌ **Gestion manuelle** : Doit être créée et supprimée manuellement
- ❌ **Complexité** : Nécessite de gérer le lifecycle séparément
- ❌ **Client ID requis** : Doit être spécifié dans le code si plusieurs identities

### Comparaison et Use Cases

**⚠️ Tableau Comparatif - System vs User-Assigned :**

| Critère | System-Assigned | User-Assigned |
|---------|----------------|---------------|
| **Lifecycle** | ❌ Lié à la ressource | ✅ Indépendant |
| **Partage** | ❌ 1:1 (une ressource) | ✅ 1:N (plusieurs ressources) |
| **Création** | ✅ Automatique | ❌ Manuelle |
| **Suppression** | ✅ Automatique | ❌ Manuelle |
| **Gestion** | ✅ Simple | ❌ Plus complexe |
| **Coût** | ✅ Gratuit | ✅ Gratuit |
| **Use Case** | Single resource, simple | Shared across multiple resources |

**Scénarios d'Examen - Quand utiliser quoi ?**

**Utiliser System-Assigned quand :**
- ✅ **Une seule ressource** : VM unique, App Service unique
- ✅ **Lifecycle couplé** : Identity doit être supprimée avec la ressource
- ✅ **Simplicité** : Pas besoin de partage d'identity
- ✅ **Prototype/Dev** : Configuration rapide

**Exemple :** Une VM qui accède à Key Vault pour récupérer ses propres secrets

**Utiliser User-Assigned quand :**
- ✅ **Plusieurs ressources** : 10 VMs accédant au même Key Vault
- ✅ **Lifecycle indépendant** : Resources recréées fréquemment (scaling)
- ✅ **Centralisation** : Gestion des permissions centralisée
- ✅ **Infrastructure as Code** : Terraform, Bicep avec réutilisation

**Exemple :** Fleet de VMs dans un VMSS accédant au même Storage Account

**Scénarios Pratiques - Pour l'Examen :**

**Scénario 1 - VM accédant à Key Vault :**
```
Requirement: Une VM doit lire un secret dans Key Vault
Solution: 
1. Activer System-Assigned MI sur la VM
2. Assigner le rôle "Key Vault Secrets User" à la MI
3. Utiliser DefaultAzureCredential dans le code
```

**Scénario 2 - App Service accédant à SQL Database :**
```
Requirement: App Service doit se connecter à Azure SQL
Solution:
1. Activer System-Assigned MI sur App Service
2. Créer un SQL User pour la Managed Identity
3. Utiliser connection string avec "Authentication=Active Directory Managed Identity"

# SQL Command pour créer l'utilisateur
CREATE USER [myAppService] FROM EXTERNAL PROVIDER;
ALTER ROLE db_datareader ADD MEMBER [myAppService];
ALTER ROLE db_datawriter ADD MEMBER [myAppService];
```

**Scénario 3 - Multiple VMs accédant au même Storage :**
```
Requirement: 20 VMs doivent écrire dans le même Storage Account
Solution:
1. Créer User-Assigned MI
2. Assigner rôle "Storage Blob Data Contributor" à la MI
3. Assigner la MI aux 20 VMs
4. Utiliser ManagedIdentityCredential(client_id=...) dans le code

Avantage: Un seul role assignment au lieu de 20
```

**Scénario 4 - Container Instances avec secrets :**
```
Requirement: ACI doit récupérer secrets au démarrage
Solution:
1. Créer User-Assigned MI (car ACI peut être recréé)
2. Assigner rôle Key Vault à la MI
3. Assigner MI au container
4. Fetch secrets via REST API ou SDK

# Via Azure CLI
az container create \
  --resource-group myRG \
  --name myContainer \
  --image myimage:latest \
  --assign-identity $userAssignedIdentityId \
  --environment-variables 'KEY_VAULT_URL=https://myvault.vault.azure.net'
```

**⚠️ Services Azure Supportant Managed Identities :**

**✅ Services avec Support Complet (System + User-Assigned) :**
- Virtual Machines
- Virtual Machine Scale Sets
- App Service / Functions
- Azure Container Instances
- Azure Kubernetes Service (AKS)
- Logic Apps
- Data Factory
- API Management
- Azure Container Registry Tasks

**✅ Services avec Support Partiel (System-Assigned uniquement) :**
- Azure SQL Managed Instance
- Azure Databricks
- Azure Cognitive Services

**❌ Services SANS Support Managed Identity :**
- Classic VMs (ARM requis)
- Azure AD B2C

**⚠️ Erreurs Courantes QCM :**

| Question | Réponse Incorrecte ❌ | Réponse Correcte ✅ |
|----------|----------------------|---------------------|
| **"10 VMs doivent accéder à Key Vault, quelle MI ?"** | "System-Assigned pour chaque VM" | "User-Assigned partagée entre les 10 VMs" |
| **"Managed Identity coûte combien ?"** | "5€ par mois par identity" | "Gratuit, aucun coût" |
| **"Peut-on assigner 2 System-Assigned MI à une VM ?"** | "Oui, jusqu'à 10" | "Non, une seule System-Assigned par ressource" |
| **"User-Assigned MI est supprimée si VM supprimée ?"** | "Oui, lifecycle lié" | "Non, lifecycle indépendant" |

**Monitoring et Troubleshooting :**

```bash
# Vérifier Managed Identity sur une VM
az vm identity show \
  --resource-group myResourceGroup \
  --name myVM

# Vérifier les role assignments d'une Managed Identity
principalId=$(az identity show --resource-group myRG --name myUserMI --query principalId -o tsv)
az role assignment list --assignee $principalId --output table

# Tester l'accès depuis la VM (via SSH/RDP)
# Obtenir un token depuis la VM
curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://vault.azure.net' \
  -H Metadata:true

# Logs d'authentification
# Azure Portal → Azure AD → Sign-in logs → Filter by "Service Principal Sign-ins"
```

**⚠️ Points Clés pour l'Examen :**
- ✅ Managed Identity = **Pas de credentials** dans le code
- ✅ **System-Assigned** : 1:1, lifecycle couplé à la ressource
- ✅ **User-Assigned** : 1:N, lifecycle indépendant, réutilisable
- ✅ **RBAC requis** : Assigner rôles appropriés (Key Vault Secrets User, Storage Blob Contributor, etc.)
- ✅ **DefaultAzureCredential** : Détecte automatiquement Managed Identity
- ✅ **Gratuit** : Aucun coût pour l'utilisation
- ✅ **IMDS Endpoint** : `http://169.254.169.254/metadata/identity/oauth2/token`
- ✅ **Multiple User-Assigned** : Une VM peut avoir plusieurs User-Assigned MI

## 1.3 Role-Based Access Control (RBAC)

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

## 1.5 Management Groups

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


## 1.6 Azure Blueprints

**⚠️ Concept Clé pour AZ-104 : Azure Blueprints permet de définir un ensemble répétable de ressources Azure conforme aux standards et patterns de l'organisation**

**Définition :**
- **Azure Blueprints** : Service de gouvernance qui orchestre le déploiement de plusieurs modèles de ressources et autres artefacts (RBAC, Policies, ARM templates)
- **Objectif** : Déploiement standardisé et reproductible d'environnements Azure conformes
- **Différence clé** : Contrairement à ARM templates seuls, Blueprints maintient une relation vivante entre la définition et le déploiement

### Concepts et Architecture

**Architecture d'un Blueprint :**

```
Blueprint Definition (Reusable Template)
    ├── Artifacts (Components)
    │   ├── Role Assignments (RBAC)
    │   ├── Policy Assignments
    │   ├── ARM Templates (Resources)
    │   └── Resource Groups
    ↓
Blueprint Assignment (Deployment to Subscription/MG)
    ├── Tracking
    ├── Versioning
    └── Auditing
```

**Composants Principaux :**

**1. Blueprint Definition**
- **Template réutilisable** : Contient tous les artefacts à déployer
- **Stockage** : Management Group ou Subscription
- **Versioning** : Support des versions (v1.0, v2.0, etc.)
- **Publication** : Doit être publié avant assignation

**2. Blueprint Assignment**
- **Déploiement actif** : Blueprint appliqué à une subscription ou management group
- **Parameters** : Valeurs spécifiques pour le déploiement
- **Lock Modes** : Protection des ressources déployées
- **Tracking** : Relation maintenue entre blueprint et ressources

**3. Artifacts (Artefacts)**
- **Role Assignments** : Assignations RBAC
- **Policy Assignments** : Policies Azure
- **ARM Templates** : Déploiement de ressources (VMs, Storage, VNets, etc.)
- **Resource Groups** : Création de Resource Groups

### Composants d'un Blueprint

**⚠️ Types d'Artefacts Supportés :**

**1. Role Assignments**
- **Usage** : Assigner des rôles RBAC aux identités
- **Scope** : Subscription ou Resource Group level
- **Exemple** : Assigner "Contributor" à un groupe sur la subscription

```json
{
  "type": "Microsoft.Blueprint/blueprints/artifacts",
  "kind": "roleAssignment",
  "properties": {
    "roleDefinitionId": "/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c",
    "principalIds": "[parameters('contributors')]"
  }
}
```

**2. Policy Assignments**
- **Usage** : Appliquer Azure Policies pour conformité
- **Scope** : Subscription ou Resource Group level
- **Exemple** : Exiger tags sur toutes les ressources

```json
{
  "type": "Microsoft.Blueprint/blueprints/artifacts",
  "kind": "policyAssignment",
  "properties": {
    "policyDefinitionId": "/providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62",
    "parameters": {
      "tagName": {
        "value": "Environment"
      }
    }
  }
}
```

**3. ARM Templates**
- **Usage** : Déployer ressources Azure (VMs, Storage, Networks)
- **Flexibilité** : Support de tous les types de ressources ARM
- **Exemple** : Déployer VNet avec NSG et subnets

```json
{
  "type": "Microsoft.Blueprint/blueprints/artifacts",
  "kind": "template",
  "properties": {
    "template": {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "resources": [
        {
          "type": "Microsoft.Network/virtualNetworks",
          "apiVersion": "2021-02-01",
          "name": "[parameters('vnetName')]",
          "location": "[parameters('location')]",
          "properties": {
            "addressSpace": {
              "addressPrefixes": ["10.0.0.0/16"]
            }
          }
        }
      ]
    },
    "parameters": {
      "vnetName": {
        "value": "[parameters('vnetName')]"
      }
    }
  }
}
```

**4. Resource Groups**
- **Usage** : Définir Resource Groups à créer
- **Placeholders** : Peuvent être référencés par d'autres artefacts
- **Exemple** : Créer "NetworkingRG" et "StorageRG"

```json
{
  "type": "Microsoft.Blueprint/blueprints/artifacts",
  "kind": "resourceGroup",
  "name": "NetworkingRG",
  "properties": {
    "displayName": "Networking Resource Group"
  }
}
```

**⚠️ Ordre de Déploiement des Artefacts :**

Les artefacts sont déployés dans un ordre spécifique :

```
1. Resource Groups
   ↓
2. Role Assignments (au niveau Subscription)
   ↓
3. Policy Assignments (au niveau Subscription)
   ↓
4. ARM Templates déployant ressources dans RGs
   ↓
5. Role Assignments (au niveau Resource Group)
   ↓
6. Policy Assignments (au niveau Resource Group)
```

**Dépendances Explicites :**

```json
{
  "type": "Microsoft.Blueprint/blueprints/artifacts",
  "kind": "template",
  "properties": {
    "dependsOn": ["NetworkingRG", "nsgTemplate"],
    "template": { }
  }
}
```

### Lifecycle et Versioning

**⚠️ Cycle de Vie d'un Blueprint :**

**1. Création (Draft)**

```bash
# Via Azure CLI
az blueprint create \
  --name "CorporateStandard" \
  --description "Corporate standard environment" \
  --target-subscription "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

# Via PowerShell
New-AzBlueprint -Name "CorporateStandard" `
  -SubscriptionId "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" `
  -Description "Corporate standard environment"
```

**2. Ajout d'Artefacts**

```bash
# Ajouter un artifact Policy
az blueprint artifact policy create \
  --blueprint-name "CorporateStandard" \
  --artifact-name "requireTags" \
  --policy-definition-id "/providers/Microsoft.Authorization/policyDefinitions/xxx" \
  --parameters '{"tagName": {"value": "Environment"}}'

# Ajouter un artifact ARM Template
az blueprint artifact template create \
  --blueprint-name "CorporateStandard" \
  --artifact-name "vnetTemplate" \
  --template @vnet-template.json \
  --parameters @vnet-parameters.json
```

**3. Publication**

```bash
# Publier une version
az blueprint publish \
  --blueprint-name "CorporateStandard" \
  --version "1.0" \
  --change-notes "Initial release"

# Via PowerShell
Publish-AzBlueprint -Blueprint $blueprint -Version "1.0"
```

**⚠️ Important :** Un Blueprint doit être **publié** avant de pouvoir être assigné.

**4. Assignation (Déploiement)**

```bash
# Assigner le blueprint à une subscription
az blueprint assignment create \
  --name "ProdEnvironment" \
  --location "eastus" \
  --identity-type "SystemAssigned" \
  --blueprint-version "/providers/Microsoft.Management/managementGroups/ContosoRoot/providers/Microsoft.Blueprint/blueprints/CorporateStandard/versions/1.0" \
  --resource-group-value artifact_name=NetworkingRG name=Networking-RG location=eastus \
  --parameters @assignment-parameters.json

# Via PowerShell
New-AzBlueprintAssignment -Name "ProdEnvironment" `
  -Blueprint $blueprint `
  -Location "eastus" `
  -SubscriptionId "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx" `
  -Lock AllResourcesDoNotDelete
```

**5. Mise à Jour**

```bash
# Modifier et publier nouvelle version
az blueprint publish \
  --blueprint-name "CorporateStandard" \
  --version "2.0" \
  --change-notes "Added NSG rules"

# Mettre à jour l'assignation existante
az blueprint assignment update \
  --name "ProdEnvironment" \
  --blueprint-version "/providers/Microsoft.Management/managementGroups/ContosoRoot/providers/Microsoft.Blueprint/blueprints/CorporateStandard/versions/2.0"
```

**⚠️ Versioning - Concepts Clés :**

**Versions de Blueprint :**
- **Draft** : Version non publiée, modifiable
- **Published** : Version immuable, assignable
- **Format** : Sémantique recommandée (v1.0, v1.1, v2.0)
- **Change Notes** : Documentation des modifications

**Gestion des Versions :**

| État | Modifiable | Assignable | Use Case |
|------|-----------|-----------|----------|
| **Draft** | ✅ Oui | ❌ Non | Développement, tests |
| **Published v1.0** | ❌ Non | ✅ Oui | Production, stable |
| **Published v2.0** | ❌ Non | ✅ Oui | Nouvelle version |

**Stratégies de Mise à Jour :**

```bash
# Stratégie 1: Mise à jour en place (risqué)
az blueprint assignment update --name "ProdEnv" --blueprint-version "2.0"

# Stratégie 2: Blue-Green (recommandé)
# 1. Assigner v2.0 à nouvelle subscription de test
az blueprint assignment create --name "ProdEnv-v2-Test" --blueprint-version "2.0"
# 2. Valider
# 3. Mettre à jour production
az blueprint assignment update --name "ProdEnv" --blueprint-version "2.0"
```

**⚠️ Lock Modes (Protection des Ressources) :**

Les blueprints peuvent protéger les ressources déployées contre les modifications :

**1. None (Par défaut)**
- **Protection** : Aucune
- **Modifications** : Autorisées
- **Use Case** : Environnements de développement

**2. DoNotDelete**
- **Protection** : Empêche suppression
- **Modifications** : Autorisées
- **Use Case** : Ressources critiques mais configurables

**3. AllResourcesReadOnly**
- **Protection** : Lecture seule complète
- **Modifications** : Bloquées (sauf via blueprint update)
- **Use Case** : Conformité stricte, audit

**4. AllResourcesDoNotDelete**
- **Protection** : Empêche suppression de toutes les ressources
- **Modifications** : Autorisées
- **Use Case** : Production avec flexibilité

```bash
# Assigner avec lock mode
az blueprint assignment create \
  --name "ProdEnv" \
  --lock-mode "AllResourcesDoNotDelete" \
  --blueprint-version "1.0"
```

**⚠️ Important :** Les locks de blueprint sont **plus forts** que les resource locks standards et peuvent uniquement être modifiés via mise à jour du blueprint.

### Blueprints vs ARM Templates vs Policy

**⚠️ Comparaison Détaillée - Pour l'Examen :**

| Critère | ARM Templates | Azure Policy | Azure Blueprints |
|---------|--------------|--------------|------------------|
| **Objectif** | Déployer ressources | Conformité governance | Orchestration complète |
| **Contenu** | Ressources uniquement | Règles de conformité | RBAC + Policies + Templates + RGs |
| **Tracking** | ❌ Aucun lien après déploiement | ✅ Évaluation continue | ✅ Relation vivante maintenue |
| **Versioning** | ❌ Manuelle (Git, etc.) | ❌ Non supporté | ✅ Natif (v1.0, v2.0) |
| **RBAC** | ❌ Séparé | ❌ Séparé | ✅ Inclus dans artifacts |
| **Policies** | ❌ Séparé | ✅ Core functionality | ✅ Inclus dans artifacts |
| **Auditing** | ❌ Limité | ✅ Compliance reports | ✅ Complet (qui, quand, quelle version) |
| **Scope** | Resource Group ou Subscription | MG, Subscription, RG, Resource | Subscription ou MG assignment |
| **Update Management** | ❌ Redéploiement manuel | ✅ Remediation tasks | ✅ Blueprint update propagation |

**Quand utiliser quoi ?**

**Utiliser ARM Templates quand :**
- ✅ **Déploiement simple** : Ressources uniquement, pas de governance
- ✅ **Flexibilité maximale** : Besoin de contrôle total sur le déploiement
- ✅ **CI/CD pipeline** : Intégration DevOps standard
- ✅ **One-time deployment** : Pas besoin de tracking

**Exemple :** Déployer une application web avec base de données

**Utiliser Azure Policy quand :**
- ✅ **Conformité uniquement** : Pas de déploiement de ressources
- ✅ **Audit et enforcement** : Vérifier configurations existantes
- ✅ **Standards organisationnels** : Appliquer règles (tags, régions, SKUs)
- ✅ **Remediation** : Corriger automatiquement non-conformités

**Exemple :** Exiger encryption sur tous les Storage Accounts

**Utiliser Azure Blueprints quand :**
- ✅ **Déploiement complet d'environnements** : Ressources + Governance
- ✅ **Standardisation** : Templates réutilisables pour subscriptions
- ✅ **Tracking requis** : Besoin de savoir ce qui a été déployé et par qui
- ✅ **Conformité stricte** : Combinaison RBAC + Policies + Resources
- ✅ **Versioning important** : Gestion de versions multiples

**Exemple :** Déployer un environnement de production standardisé avec networking, RBAC, policies de sécurité, et ressources compute

**Scénarios Pratiques - Pour l'Examen :**

**Scénario 1 - Environnement de Production Standardisé :**

```
Requirement: Déployer 10 subscriptions identiques pour différentes BU
- VNet avec subnets standardisés
- NSG avec règles de sécurité
- Storage Account avec encryption obligatoire
- RBAC: Contributor pour équipe BU, Reader pour security team
- Policies: Require tags, allowed locations, allowed VM SKUs

Solution: Azure Blueprints
1. Créer blueprint avec artifacts:
   - ARM template pour VNet et NSG
   - ARM template pour Storage
   - Policy assignments (tags, locations, SKUs)
   - Role assignments (Contributor, Reader)
2. Publier version 1.0
3. Assigner à chaque subscription avec lock mode DoNotDelete
4. Tracking automatique, versioning, audit complet
```

**Scénario 2 - Landing Zone Azure :**

```
Requirement: Créer landing zone pour nouvelles subscriptions
- Management Groups hierarchy
- Logging et monitoring (Log Analytics)
- Security baseline (Microsoft Defender for Cloud)
- Networking hub-and-spoke
- RBAC standardisé

Solution: Azure Blueprints (CAF Landing Zone)
- Utiliser Microsoft Cloud Adoption Framework (CAF) blueprints
- Personnaliser artifacts pour organisation
- Déployer via blueprint assignment
```

**Scénario 3 - Conformité ISO 27001 :**

```
Requirement: Subscription conforme ISO 27001
- 50+ Azure Policies pour conformité
- RBAC avec least privilege
- Logging et auditing activés
- Encryption partout

Solution: Azure Blueprints (ISO 27001 sample)
- Utiliser blueprint sample Microsoft
- Assigner à subscription avec lock mode ReadOnly
- Automatic compliance reporting
```

**⚠️ Blueprint Samples Microsoft :**

Microsoft fournit des blueprints prêts à l'emploi :

**Compliance :**
- **ISO 27001** : Conformité ISO 27001
- **NIST SP 800-53** : Standards fédéraux US
- **PCI-DSS v3.2.1** : Payment Card Industry
- **HIPAA HITRUST** : Healthcare compliance
- **UK OFFICIAL / UK NHS** : UK Government standards
- **Canada Federal PBMM** : Canadian government

**Foundation :**
- **CAF Foundation** : Cloud Adoption Framework base
- **CAF Migration Landing Zone** : Migration-ready environment

**Création et Gestion via Portal :**

```
Azure Portal Navigation:
1. Search "Blueprints"
2. Blueprint definitions → Create
3. Choisir template (blank ou sample)
4. Ajouter artifacts
5. Publish
6. Assignments → Create assignment
```

**⚠️ Permissions Requises :**

**Pour créer/modifier blueprints :**
- **Owner** sur Management Group ou Subscription (stockage du blueprint)
- Ou rôle custom avec : `Microsoft.Blueprint/blueprints/write`

**Pour assigner blueprints :**
- **Owner** sur target subscription
- **Blueprint Operator** (rôle built-in) : Peut assigner blueprints existants
- ⚠️ Note : Blueprint assignment utilise System-Assigned Managed Identity qui nécessite permissions sur target resources

**Assigner permissions à Managed Identity du Blueprint :**

```bash
# Le blueprint assignment crée automatiquement une System-Assigned MI
# Cette MI nécessite des permissions pour déployer les ressources

# Exemple: Si blueprint déploie VMs et Storage
# La MI a besoin de "Contributor" sur la subscription

# Cela est géré automatiquement si vous avez Owner
# Sinon, assigner manuellement:
principalId=$(az blueprint assignment show --name "ProdEnv" --query identity.principalId -o tsv)
az role assignment create \
  --assignee $principalId \
  --role "Contributor" \
  --scope "/subscriptions/xxx"
```

**⚠️ Erreurs Courantes QCM :**

| Question | Réponse Incorrecte ❌ | Réponse Correcte ✅ |
|----------|----------------------|---------------------|
| **"Différence Blueprint vs ARM template ?"** | "Blueprint est plus rapide" | "Blueprint maintient relation + versioning + includes RBAC/Policies" |
| **"Peut-on modifier un blueprint publié ?"** | "Oui, à tout moment" | "Non, doit créer nouvelle version" |
| **"Blueprint peut déployer sur Management Group ?"** | "Oui, déploiement direct" | "Non, assignation à subscription uniquement" |
| **"Lock mode ReadOnly empêche blueprint update ?"** | "Oui, aucune modification possible" | "Non, blueprint update contourne le lock" |
| **"ARM template suffit pour RBAC + Policies ?"** | "Oui, tout dans ARM" | "Non, utiliser Blueprint pour orchestration complète" |

**Monitoring et Troubleshooting :**

```bash
# Lister les blueprints
az blueprint list --subscription "xxx"

# Voir détails d'un blueprint
az blueprint show --name "CorporateStandard"

# Lister les assignments
az blueprint assignment list

# Voir statut d'un assignment
az blueprint assignment show --name "ProdEnv"

# Voir l'historique de déploiement
az blueprint assignment operation list --name "ProdEnv"

# Logs dans Azure Portal
# Blueprints → Assigned blueprints → Select assignment → Deployment history
```

**⚠️ Points Clés pour l'Examen :**
- ✅ Blueprint = **Orchestration** de RBAC + Policies + ARM Templates + Resource Groups
- ✅ **Tracking vivant** : Relation maintenue entre definition et deployment
- ✅ **Versioning natif** : Published versions immuables
- ✅ **Lock modes** : Protection des ressources déployées
- ✅ **Doit être publié** avant assignation
- ✅ **Managed Identity** : System-Assigned MI créée automatiquement pour deployment
- ✅ **Artifacts** : 4 types (Role, Policy, Template, Resource Group)
- ✅ **Scope** : Blueprint definition au MG/Subscription, assignment à Subscription uniquement
- ✅ **Use Case Principal** : Déploiements standardisés d'environnements conformes
- ✅ **Samples** : Microsoft fournit blueprints ISO, NIST, PCI-DSS, CAF

