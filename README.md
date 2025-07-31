# Complete AZ-104 Guide: Microsoft Azure Administrator

## 📋 Certification Overview

The AZ-104 exam validates your skills as an Azure Administrator. It covers subscription management, security, storage, networking, and compute resources.

**Duration:** 150 minutes  
**Questions:** 40-60 questions  
**Passing Score:** 700/1000  
**Cost:** $165 USD

---

## 🎯 Exam Domains (Weight Distribution)

### 1. Manage Azure identities and governance (15-20%)
### 2. Implement and manage storage (15-20%)
### 3. Deploy and manage Azure compute resources (20-25%)
### 4. Configure and manage virtual networking (25-30%)
### 5. Monitor and back up Azure resources (10-15%)

---

## 🔐 1. Azure Identities and Governance

### Azure Active Directory (Azure AD / Entra ID)

#### Migration des Commandes PowerShell

**⚠️ Important : Commandes Dépréciées**
Microsoft a déprécié le module `AzureAD` PowerShell. Utilisez maintenant le module `Az` avec les commandes `AzAD*` :

**Commandes Dépréciées → Modernes :**
- `Connect-AzureAD` → `Connect-AzAccount`
- `Get-AzureADUser` → `Get-AzADUser`
- `New-AzureADUser` → `New-AzADUser`
- `Set-AzureADUser` → `Update-AzADUser`
- `Remove-AzureADUser` → `Remove-AzADUser`
- `Get-AzureADGroup` → `Get-AzADGroup`
- `New-AzureADGroup` → `New-AzADGroup`
- `Add-AzureADGroupMember` → `Add-AzADGroupMember`
- `Remove-AzureADGroupMember` → `Remove-AzADGroupMember`

**Installation du Module Az :**
```powershell
# Installer le module Az
Install-Module -Name Az -AllowClobber -Force

# Importer le module
Import-Module Az

# Se connecter à Azure
Connect-AzAccount
```

**Permissions Requises :**
Pour utiliser les commandes `Get-AzADUser`, vous devez avoir les permissions appropriées :
- **Global Administrator** ou
- **User Administrator** ou
- **Directory Readers** (pour la lecture uniquement)

**Vérification des Permissions :**
```powershell
# Vérifier votre rôle actuel
Get-AzRoleAssignment -SignInName (Get-AzContext).Account.Id

# Vérifier si vous pouvez lister les utilisateurs
Get-AzADUser -First 1
```

#### Aide et Documentation des Commandes

**PowerShell - Get-Help :**
```powershell
# Aide générale sur une commande
Get-Help Get-AzADUser

# Aide détaillée avec exemples
Get-Help Get-AzADUser -Detailed

# Aide complète avec tous les paramètres
Get-Help Get-AzADUser -Full

# Aide avec exemples uniquement
Get-Help Get-AzADUser -Examples

# Rechercher des commandes contenant un mot
Get-Help *ADUser*

# Aide sur un paramètre spécifique
Get-Help Get-AzADUser -Parameter UserPrincipalName
```

**Azure CLI - --help :**
```bash
# Aide générale sur une commande
az ad user --help

# Aide sur une sous-commande
az ad user list --help

# Aide détaillée avec exemples
az ad user create --help

# Rechercher des commandes
az find "user"

# Aide sur un groupe de commandes
az ad --help

# Aide sur les paramètres globaux
az --help
```

**Conseils d'Utilisation :**
- **PowerShell** : `Get-Help` est plus verbeux et structuré
- **Azure CLI** : `--help` est plus concis et orienté action
- **Recherche** : Utilisez `Get-Help *mot*` ou `az find "mot"`
- **Exemples** : Les deux fournissent des exemples pratiques
- **Paramètres** : Vérifiez toujours les paramètres requis vs optionnels

**Découverte des Commandes :**
```powershell
# PowerShell - Lister toutes les commandes Az
Get-Command -Module Az

# PowerShell - Rechercher des commandes spécifiques
Get-Command *ADUser*
Get-Command *Storage*
Get-Command *VM*
Get-Command *Network*
Get-Command *Monitor*
```

```bash
# Azure CLI - Lister tous les groupes de commandes
az --help

# Azure CLI - Explorer les commandes disponibles
az ad --help
az storage --help
az vm --help
az network --help
az monitor --help

# Azure CLI - Rechercher des commandes
az find "user"
az find "storage"
az find "vm"
az find "network"
az find "monitor"
```

#### Concepts Fondamentaux

**Qu'est-ce qu'Azure AD ?**
Azure Active Directory (maintenant appelé Microsoft Entra ID) est le service d'identité et d'accès basé sur le cloud de Microsoft. Il permet aux utilisateurs de se connecter et d'accéder aux ressources internes et externes.

**Tenants et Directories**
- **Tenant** : Une instance dédiée d'Azure AD qui représente une organisation
- **Directory** : Le répertoire d'identités dans un tenant
- Chaque tenant a un domaine principal (ex: contoso.onmicrosoft.com)

**Types d'Utilisateurs**
- **Utilisateurs Cloud** : Créés directement dans Azure AD
- **Utilisateurs Synchronisés** : Synchronisés depuis Active Directory local
- **Utilisateurs Invités** : Utilisateurs externes invités via B2B

**Groupes et Unités Administratives**
- **Groupes de Sécurité** : Pour gérer les permissions
- **Groupes Microsoft 365** : Pour la collaboration
- **Unités Administratives** : Pour déléguer l'administration

#### Key Concepts
- Tenants and directories
- Users, groups, and administrative units
- Built-in vs custom roles
- Multi-factor authentication (MFA)
- Conditional Access

#### User Management Commands

**Azure CLI:**
```bash
# Create a user
az ad user create --display-name "John Doe" --password "TempPassword123!" --user-principal-name "john.doe@yourdomain.com"

# List all users
az ad user list --output table

# Get user details
az ad user show --id "john.doe@yourdomain.com"

# Update user
az ad user update --id "john.doe@yourdomain.com" --display-name "John Smith"

# Delete user
az ad user delete --id "john.doe@yourdomain.com"

# Reset password
az ad user update --id "john.doe@yourdomain.com" --password "NewPassword123!" --force-change-password-next-sign-in true
```

**PowerShell:**
```powershell
# Connect to Azure AD
Connect-AzAccount

# Create a user
$passwordProfile = New-Object -TypeName Microsoft.Azure.PowerShell.Cmdlets.Resources.MSGraph.Models.ApiV10.IMicrosoftGraphPasswordProfile
$passwordProfile.Password = "TempPassword123!"
$passwordProfile.ForceChangePasswordNextSignIn = $true
New-AzADUser -DisplayName "John Doe" -PasswordProfile $passwordProfile -UserPrincipalName "john.doe@yourdomain.com" -AccountEnabled $true -MailNickName "johndoe"

# List all users
Get-AzADUser

# Get user details
Get-AzADUser -UserPrincipalName "john.doe@yourdomain.com"

# Update user
Update-AzADUser -UserPrincipalName "john.doe@yourdomain.com" -DisplayName "John Smith"

# Delete user
Remove-AzADUser -UserPrincipalName "john.doe@yourdomain.com"
```

#### Group Management Commands

**Azure CLI:**
```bash
# Create a security group
az ad group create --display-name "IT Department" --mail-nickname "ITDept"

# List all groups
az ad group list --output table

# Add user to group
az ad group member add --group "IT Department" --member-id "john.doe@yourdomain.com"

# Remove user from group
az ad group member remove --group "IT Department" --member-id "john.doe@yourdomain.com"

# List group members
az ad group member list --group "IT Department" --output table
```

**PowerShell:**
```powershell
# Create a security group
New-AzADGroup -DisplayName "IT Department" -MailEnabled $false -SecurityEnabled $true -MailNickName "ITDept"

# List all groups
Get-AzADGroup

# Add user to group
Add-AzADGroupMember -MemberObjectId "user-object-id" -TargetGroupObjectId "group-object-id"

# Remove user from group
Remove-AzADGroupMember -MemberObjectId "user-object-id" -GroupObjectId "group-object-id"

# List group members
Get-AzADGroupMember -GroupObjectId "group-object-id"
```

### Role-Based Access Control (RBAC)

#### Aide RBAC

**PowerShell - Aide RBAC :**
```powershell
# Aide sur les commandes RBAC
Get-Help *Role*

# Aide spécifique sur les rôles
Get-Help Get-AzRoleDefinition -Detailed

# Aide sur les assignations
Get-Help New-AzRoleAssignment -Examples

# Rechercher des commandes RBAC
Get-Help *RoleAssignment*
```

**Azure CLI - Aide RBAC :**
```bash
# Aide sur les rôles
az role definition --help

# Aide sur les assignations
az role assignment --help

# Aide sur une sous-commande spécifique
az role assignment create --help

# Rechercher des commandes RBAC
az find "role"
```

**Découverte des Commandes RBAC :**
```powershell
# PowerShell - Commandes RBAC disponibles
Get-Command *Role*
Get-Command *RoleDefinition*
Get-Command *RoleAssignment*
```

```bash
# Azure CLI - Explorer les commandes RBAC
az role --help
az role definition --help
az role assignment --help
```

#### Concepts RBAC

**Qu'est-ce que RBAC ?**
Le contrôle d'accès basé sur les rôles (RBAC) est un système d'autorisation qui permet de gérer l'accès aux ressources Azure en attribuant des rôles appropriés aux utilisateurs, groupes et services.

**Principes RBAC**
- **Principe du moindre privilège** : Accorder uniquement les permissions nécessaires
- **Séparation des responsabilités** : Répartir les tâches administratives
- **Accès juste à temps** : Accorder l'accès temporairement si nécessaire

**Hiérarchie des Scopes**
1. **Management Group** : Plus haut niveau, affecte plusieurs abonnements
2. **Subscription** : Niveau d'abonnement
3. **Resource Group** : Groupe de ressources
4. **Resource** : Ressource individuelle

**Types de Rôles**
- **Built-in Roles** : Rôles prédéfinis par Microsoft
- **Custom Roles** : Rôles personnalisés créés par l'organisation

#### Built-in Roles
- **Owner:** Full access including access management
- **Contributor:** Full access except access management
- **Reader:** View only access
- **User Access Administrator:** Manage user access only

#### RBAC Commands

**Azure CLI:**
```bash
# List all role definitions
az role definition list --output table

# Get specific role definition
az role definition show --name "Virtual Machine Contributor"

# Create custom role
az role definition create --role-definition @customrole.json

# List role assignments
az role assignment list --output table

# Assign role to user
az role assignment create --assignee "user@domain.com" --role "Virtual Machine Contributor" --scope "/subscriptions/subscription-id/resourceGroups/myResourceGroup"

# Assign role to group
az role assignment create --assignee-object-id "group-object-id" --role "Contributor" --scope "/subscriptions/subscription-id"

# Remove role assignment
az role assignment delete --assignee "user@domain.com" --role "Virtual Machine Contributor" --scope "/subscriptions/subscription-id/resourceGroups/myResourceGroup"
```

**PowerShell:**
```powershell
# List all role definitions
Get-AzRoleDefinition

# Get specific role definition
Get-AzRoleDefinition -Name "Virtual Machine Contributor"

# Create custom role
New-AzRoleDefinition -InputFile "customrole.json"

# List role assignments
Get-AzRoleAssignment

# Assign role to user
New-AzRoleAssignment -SignInName "user@domain.com" -RoleDefinitionName "Virtual Machine Contributor" -ResourceGroupName "myResourceGroup"

# Assign role to group
New-AzRoleAssignment -ObjectId "group-object-id" -RoleDefinitionName "Contributor" -Scope "/subscriptions/subscription-id"

# Remove role assignment
Remove-AzRoleAssignment -SignInName "user@domain.com" -RoleDefinitionName "Virtual Machine Contributor" -ResourceGroupName "myResourceGroup"
```

### Azure Policy

#### Découverte des Commandes Policy

**PowerShell - Commandes Policy disponibles :**
```powershell
# PowerShell - Commandes Policy disponibles
Get-Command *Policy*
Get-Command *PolicyDefinition*
Get-Command *PolicyAssignment*
Get-Command *PolicyState*
```

**Azure CLI - Explorer les commandes Policy :**
```bash
# Azure CLI - Explorer les commandes Policy
az policy --help
az policy definition --help
az policy assignment --help
az policy state --help
```

#### Concepts Azure Policy

**Qu'est-ce qu'Azure Policy ?**
Azure Policy est un service qui vous permet de créer, assigner et gérer des politiques pour contrôler et organiser vos ressources Azure. Il garantit que vos ressources restent conformes aux normes de votre entreprise.

**Types de Politiques**
- **Built-in Policies** : Politiques prédéfinies par Microsoft
- **Custom Policies** : Politiques personnalisées créées par l'organisation
- **Initiative Policies** : Groupes de politiques liées

**Effets des Politiques**
- **Deny** : Empêche la création/modification de ressources non conformes
- **Audit** : Permet l'action mais enregistre la non-conformité
- **Append** : Ajoute des propriétés aux ressources
- **DeployIfNotExists** : Déploie des ressources si elles n'existent pas
- **Modify** : Modifie les propriétés des ressources existantes

**Évaluation des Politiques**
- **Automatique** : Évaluation lors de la création/modification
- **Manuelle** : Déclenchement manuel d'une évaluation
- **Compliance** : Suivi de la conformité des ressources

#### Policy Commands

**Azure CLI:**
```bash
# List policy definitions
az policy definition list --output table

# Get policy definition
az policy definition show --name "policy-definition-name"

# Create policy definition
az policy definition create --name "my-policy" --display-name "My Policy" --description "Policy description" --rules @policy-rules.json --params @policy-params.json

# List policy assignments
az policy assignment list --output table

# Assign policy
az policy assignment create --name "my-assignment" --display-name "My Assignment" --policy "policy-definition-id" --scope "/subscriptions/subscription-id/resourceGroups/myResourceGroup"

# List policy compliance
az policy state list --output table

# Trigger policy evaluation
az policy state trigger-scan --resource-group "myResourceGroup"
```

**PowerShell:**
```powershell
# List policy definitions
Get-AzPolicyDefinition

# Get policy definition
Get-AzPolicyDefinition -Name "policy-definition-name"

# Create policy definition
New-AzPolicyDefinition -Name "my-policy" -DisplayName "My Policy" -Description "Policy description" -Policy "policy-rules.json" -Parameter "policy-params.json"

# List policy assignments
Get-AzPolicyAssignment

# Assign policy
New-AzPolicyAssignment -Name "my-assignment" -DisplayName "My Assignment" -PolicyDefinition $policyDef -Scope "/subscriptions/subscription-id/resourceGroups/myResourceGroup"

# Get policy compliance
Get-AzPolicyState
```

### Subscription Management

#### Concepts de Gestion des Abonnements

**Qu'est-ce qu'un Abonnement Azure ?**
Un abonnement Azure est un accord avec Microsoft pour utiliser des services cloud Azure. Il définit comment les ressources Azure sont facturées et gérées.

**Types d'Abonnements**
- **Free Account** : Compte gratuit avec crédits limités
- **Pay-As-You-Go** : Paiement à l'usage
- **Enterprise Agreement** : Accord d'entreprise avec remises
- **Cloud Solution Provider** : Via un partenaire

**Management Groups**
- **Hiérarchie** : Organisation logique des abonnements
- **Héritage** : Les politiques et rôles sont hérités
- **Gouvernance** : Centralisation de la gestion

**Ressources et Facturation**
- **Resource Groups** : Groupement logique des ressources
- **Tags** : Métadonnées pour l'organisation et la facturation
- **Cost Management** : Suivi et optimisation des coûts

**Azure CLI:**
```bash
# List subscriptions
az account list --output table

# Set active subscription
az account set --subscription "subscription-id"

# Show current subscription
az account show

# List locations
az account list-locations --output table

# Create management group
az account management-group create --name "my-mg" --display-name "My Management Group"

# List management groups
az account management-group list --output table
```

**PowerShell:**
```powershell
# List subscriptions
Get-AzSubscription

# Set active subscription
Set-AzContext -SubscriptionId "subscription-id"

# Show current subscription
Get-AzContext

# List locations
Get-AzLocation

# Create management group
New-AzManagementGroup -GroupName "my-mg" -DisplayName "My Management Group"

# List management groups
Get-AzManagementGroup
```

---

## 🏗️ Infrastructure as Code (ARM Templates & Bicep)

### Concepts d'Infrastructure as Code

**Qu'est-ce que l'Infrastructure as Code (IaC) ?**
L'Infrastructure as Code est une approche de gestion de l'infrastructure qui traite la configuration et le déploiement d'infrastructure comme du code logiciel, permettant l'automatisation, la reproductibilité et la versioning.

**Avantages de l'IaC :**
- **Reproductibilité** : Déploiements identiques et prévisibles
- **Versioning** : Contrôle de version de l'infrastructure
- **Automation** : Déploiements automatisés et CI/CD
- **Consistency** : Cohérence entre environnements
- **Documentation** : Infrastructure documentée dans le code
- **Collaboration** : Travail d'équipe sur l'infrastructure

### Azure Resource Manager (ARM) Templates

#### Concepts ARM Templates

**Qu'est-ce qu'un ARM Template ?**
Un ARM Template est un fichier JSON qui définit l'infrastructure et la configuration de vos ressources Azure. Il utilise le service Azure Resource Manager pour déployer et gérer vos ressources.

**Structure d'un ARM Template :**
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    // Paramètres d'entrée
  },
  "variables": {
    // Variables internes
  },
  "resources": [
    // Définition des ressources
  ],
  "outputs": {
    // Valeurs de sortie
  }
}
```

**Composants ARM Template :**
- **Parameters** : Valeurs d'entrée configurables
- **Variables** : Valeurs calculées ou réutilisables
- **Resources** : Définition des ressources Azure
- **Outputs** : Valeurs retournées après déploiement
- **Functions** : Fonctions intégrées pour la logique

#### ARM Template Commands

**Azure CLI:**
```bash
# Deploy ARM template
az deployment group create --resource-group "myResourceGroup" --template-file "template.json" --parameters "parameters.json"

# Deploy ARM template with inline parameters
az deployment group create --resource-group "myResourceGroup" --template-file "template.json" --parameters "param1=value1" "param2=value2"

# Validate ARM template
az deployment group validate --resource-group "myResourceGroup" --template-file "template.json" --parameters "parameters.json"

# Export ARM template from existing resources
az group export --resource-group "myResourceGroup" --include-parameter-default-value

# What-if deployment (preview changes)
az deployment group what-if --resource-group "myResourceGroup" --template-file "template.json" --parameters "parameters.json"

# Deploy ARM template at subscription level
az deployment sub create --location "eastus" --template-file "template.json" --parameters "parameters.json"

# Deploy ARM template at management group level
az deployment mg create --location "eastus" --template-file "template.json" --parameters "parameters.json"

# Deploy ARM template at tenant level
az deployment tenant create --location "eastus" --template-file "template.json" --parameters "parameters.json"
```

**PowerShell:**
```powershell
# Deploy ARM template
New-AzResourceGroupDeployment -ResourceGroupName "myResourceGroup" -TemplateFile "template.json" -TemplateParameterFile "parameters.json"

# Deploy ARM template with inline parameters
New-AzResourceGroupDeployment -ResourceGroupName "myResourceGroup" -TemplateFile "template.json" -param1 "value1" -param2 "value2"

# Validate ARM template
Test-AzResourceGroupDeployment -ResourceGroupName "myResourceGroup" -TemplateFile "template.json" -TemplateParameterFile "parameters.json"

# Export ARM template from existing resources
Export-AzResourceGroup -ResourceGroupName "myResourceGroup" -IncludeParameterDefaultValue

# What-if deployment (preview changes)
New-AzResourceGroupDeployment -ResourceGroupName "myResourceGroup" -TemplateFile "template.json" -WhatIf

# Deploy ARM template at subscription level
New-AzDeployment -Location "East US" -TemplateFile "template.json" -TemplateParameterFile "parameters.json"

# Deploy ARM template at management group level
New-AzManagementGroupDeployment -ManagementGroupId "myManagementGroup" -Location "East US" -TemplateFile "template.json" -TemplateParameterFile "parameters.json"

# Deploy ARM template at tenant level
New-AzTenantDeployment -Location "East US" -TemplateFile "template.json" -TemplateParameterFile "parameters.json"
```

#### Exemple ARM Template

**Template de base pour un compte de stockage :**
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountName": {
      "type": "string",
      "metadata": {
        "description": "Name of the storage account"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for all resources"
      }
    },
    "sku": {
      "type": "string",
      "defaultValue": "Standard_LRS",
      "allowedValues": [
        "Standard_LRS",
        "Standard_GRS",
        "Standard_ZRS"
      ],
      "metadata": {
        "description": "Storage account SKU"
      }
    }
  },
  "variables": {
    "storageAccountName": "[toLower(parameters('storageAccountName'))]"
  },
  "resources": [
    {
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2021-09-01",
      "name": "[variables('storageAccountName')]",
      "location": "[parameters('location')]",
      "sku": {
        "name": "[parameters('sku')]"
      },
      "kind": "StorageV2",
      "properties": {
        "supportsHttpsTrafficOnly": true,
        "minimumTlsVersion": "TLS1_2"
      }
    }
  ],
  "outputs": {
    "storageAccountName": {
      "type": "string",
      "value": "[variables('storageAccountName')]"
    },
    "storageAccountId": {
      "type": "string",
      "value": "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
    }
  }
}
```

**Fichier de paramètres :**
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageAccountName": {
      "value": "mystorageaccount123"
    },
    "location": {
      "value": "East US"
    },
    "sku": {
      "value": "Standard_GRS"
    }
  }
}
```

### Azure Bicep

#### Concepts Azure Bicep

**Qu'est-ce qu'Azure Bicep ?**
Azure Bicep est un langage de domaine spécifique (DSL) qui simplifie l'écriture d'ARM templates. Il offre une syntaxe plus claire et plus concise que JSON, tout en générant des ARM templates valides.

**Avantages de Bicep :**
- **Syntaxe simplifiée** : Plus lisible que JSON
- **IntelliSense** : Support complet dans VS Code
- **Modularité** : Réutilisation de modules
- **Validation** : Validation en temps réel
- **Compilation** : Génération automatique d'ARM templates
- **Versioning** : Compatible avec Git

**Structure d'un fichier Bicep :**
```bicep
@description('Description du paramètre')
param parameterName string = 'defaultValue'

var variableName = 'computedValue'

resource resourceName 'Microsoft.Provider/resourceType@apiVersion' = {
  name: 'resourceName'
  location: location
  properties: {
    // Propriétés de la ressource
  }
}

output outputName string = resourceName.properties.outputProperty
```

#### Bicep Commands

**Azure CLI:**
```bash
# Deploy Bicep file
az deployment group create --resource-group "myResourceGroup" --template-file "template.bicep" --parameters "parameters.json"

# Deploy Bicep file with inline parameters
az deployment group create --resource-group "myResourceGroup" --template-file "template.bicep" --parameters "param1=value1" "param2=value2"

# Validate Bicep file
az deployment group validate --resource-group "myResourceGroup" --template-file "template.bicep" --parameters "parameters.json"

# Build Bicep to ARM template
az bicep build --file "template.bicep"

# Decompile ARM template to Bicep
az bicep decompile --file "template.json"

# Install Bicep CLI
az bicep install

# Update Bicep CLI
az bicep upgrade

# Check Bicep version
az bicep version

# Lint Bicep file
az bicep build --file "template.bicep" --stdout

# Deploy Bicep at subscription level
az deployment sub create --location "eastus" --template-file "template.bicep" --parameters "parameters.json"

# Deploy Bicep at management group level
az deployment mg create --location "eastus" --template-file "template.bicep" --parameters "parameters.json"
```

**PowerShell:**
```powershell
# Deploy Bicep file
New-AzResourceGroupDeployment -ResourceGroupName "myResourceGroup" -TemplateFile "template.bicep" -TemplateParameterFile "parameters.json"

# Deploy Bicep file with inline parameters
New-AzResourceGroupDeployment -ResourceGroupName "myResourceGroup" -TemplateFile "template.bicep" -param1 "value1" -param2 "value2"

# Validate Bicep file
Test-AzResourceGroupDeployment -ResourceGroupName "myResourceGroup" -TemplateFile "template.bicep" -TemplateParameterFile "parameters.json"

# Build Bicep to ARM template
bicep build template.bicep

# Decompile ARM template to Bicep
bicep decompile template.json

# Deploy Bicep at subscription level
New-AzDeployment -Location "East US" -TemplateFile "template.bicep" -TemplateParameterFile "parameters.json"

# Deploy Bicep at management group level
New-AzManagementGroupDeployment -ManagementGroupId "myManagementGroup" -Location "East US" -TemplateFile "template.bicep" -TemplateParameterFile "parameters.json"
```

#### Exemple Bicep

**Template Bicep équivalent au JSON précédent :**
```bicep
@description('Name of the storage account')
param storageAccountName string

@description('Location for all resources')
param location string = resourceGroup().location

@description('Storage account SKU')
@allowed([
  'Standard_LRS'
  'Standard_GRS'
  'Standard_ZRS'
])
param sku string = 'Standard_LRS'

var storageAccountNameLower = toLower(storageAccountName)

resource storageAccount 'Microsoft.Storage/storageAccounts@2021-09-01' = {
  name: storageAccountNameLower
  location: location
  sku: {
    name: sku
  }
  kind: 'StorageV2'
  properties: {
    supportsHttpsTrafficOnly: true
    minimumTlsVersion: 'TLS1_2'
  }
}

output storageAccountName string = storageAccountNameLower
output storageAccountId string = storageAccount.id
```

#### Fonctionnalités Avancées Bicep

**Modules Bicep :**
```bicep
// main.bicep
param storageAccountName string
param location string = resourceGroup().location

module storageModule 'modules/storage.bicep' = {
  name: 'storageDeployment'
  params: {
    storageAccountName: storageAccountName
    location: location
  }
}

output storageAccountId string = storageModule.outputs.storageAccountId
```

```bicep
// modules/storage.bicep
param storageAccountName string
param location string

resource storageAccount 'Microsoft.Storage/storageAccounts@2021-09-01' = {
  name: storageAccountName
  location: location
  sku: {
    name: 'Standard_LRS'
  }
  kind: 'StorageV2'
}

output storageAccountId string = storageAccount.id
```

**Conditions et Boucles :**
```bicep
param environment string = 'dev'
param vmCount int = 1

// Condition
resource storageAccount 'Microsoft.Storage/storageAccounts@2021-09-01' = if (environment == 'prod') {
  name: 'prodstorageaccount'
  location: resourceGroup().location
  sku: {
    name: 'Standard_GRS'
  }
  kind: 'StorageV2'
}

// Boucle
resource vm 'Microsoft.Compute/virtualMachines@2021-11-01' = [for i in range(0, vmCount): {
  name: 'vm-${i}'
  location: resourceGroup().location
  properties: {
    // Propriétés de la VM
  }
}]
```

**Fonctions et Expressions :**
```bicep
// Utilisation de fonctions
var uniqueString = uniqueString(resourceGroup().id)
var concatName = concat('storage', uniqueString)

// Expressions conditionnelles
var skuName = environment == 'prod' ? 'Standard_GRS' : 'Standard_LRS'

// Références entre ressources
resource storageAccount 'Microsoft.Storage/storageAccounts@2021-09-01' = {
  name: concatName
  location: resourceGroup().location
  sku: {
    name: skuName
  }
  kind: 'StorageV2'
}

resource container 'Microsoft.Storage/storageAccounts/blobServices/containers@2021-09-01' = {
  parent: storageAccount
  name: 'default/mycontainer'
}
```

### Bonnes Pratiques ARM/Bicep

**Organisation et Structure :**
- **Modularité** : Diviser les templates en modules réutilisables
- **Paramétrage** : Utiliser des paramètres pour la flexibilité
- **Validation** : Valider les templates avant déploiement
- **Versioning** : Utiliser Git pour le contrôle de version
- **Documentation** : Documenter les paramètres et modules

**Sécurité :**
- **Secrets** : Utiliser Key Vault pour les secrets
- **RBAC** : Définir les rôles appropriés
- **Chiffrement** : Activer le chiffrement par défaut
- **NSG** : Configurer les groupes de sécurité réseau

**Performance :**
- **Dépendances** : Optimiser les dépendances entre ressources
- **Parallélisation** : Utiliser dependsOn pour le parallélisme
- **Taille** : Limiter la taille des templates
- **Caching** : Utiliser les outputs pour éviter les recalculs

**Maintenance :**
- **Naming** : Conventions de nommage cohérentes
- **Tags** : Utiliser des tags pour l'organisation
- **Testing** : Tests automatisés des templates
- **Monitoring** : Surveiller les déploiements

---

## 💾 2. Azure Storage

### Concepts de Stockage Azure

**Qu'est-ce qu'Azure Storage ?**
Azure Storage est le service de stockage cloud de Microsoft qui offre une solution de stockage hautement disponible, évolutive, durable et sécurisée pour les données.

**Types de Stockage**
- **Blob Storage** : Stockage d'objets pour fichiers non structurés
- **File Storage** : Partages de fichiers SMB pour applications
- **Queue Storage** : Stockage de messages pour communication asynchrone
- **Table Storage** : Base de données NoSQL pour données structurées
- **Disk Storage** : Disques managés pour machines virtuelles

**Niveaux de Performance**
- **Standard** : Stockage HDD pour charges de travail générales
- **Premium** : Stockage SSD pour charges de travail intensives

**Redondance et Disponibilité**

**Types de Redondance de Stockage Azure :**

1. **LRS (Locally Redundant Storage)**
   - **Disponibilité** : 99.9% (99% pour les comptes GPv1)
   - **Copies** : 3 copies synchrones dans un datacenter
   - **Région** : Une seule région
   - **Coût** : ~0.0184€/GB/mois (le moins cher)
   - **Cas d'usage** : Développement/test, données non critiques

2. **ZRS (Zone-Redundant Storage)**
   - **Disponibilité** : 99.9%
   - **Copies** : 3 copies synchrones dans 3 zones de disponibilité
   - **Région** : Une seule région, plusieurs zones
   - **Coût** : ~0.0202€/GB/mois (+10% vs LRS)
   - **Cas d'usage** : Applications critiques nécessitant haute disponibilité

3. **GRS (Geo-Redundant Storage)**
   - **Disponibilité** : 99.9%
   - **Copies** : 6 copies (3 dans région primaire + 3 dans région secondaire)
   - **Région** : Deux régions (primaire + secondaire)
   - **Coût** : ~0.0368€/GB/mois (+100% vs LRS)
   - **Cas d'usage** : Protection contre les sinistres régionaux

4. **RA-GRS (Read-Access Geo-Redundant Storage)**
   - **Disponibilité** : 99.9%
   - **Copies** : 6 copies (comme GRS)
   - **Accès** : Lecture possible sur la région secondaire
   - **Région** : Deux régions avec accès en lecture
   - **Coût** : ~0.0368€/GB/mois (même prix que GRS)
   - **Cas d'usage** : Applications nécessitant accès en lecture sur région secondaire

5. **GZRS (Geo-Zone-Redundant Storage)**
   - **Disponibilité** : 99.9%
   - **Copies** : 6 copies (3 dans 3 zones de la région primaire + 3 dans région secondaire)
   - **Région** : Deux régions avec ZRS dans la primaire
   - **Coût** : ~0.0404€/GB/mois (+120% vs LRS)
   - **Cas d'usage** : Applications critiques nécessitant protection maximale

6. **RA-GZRS (Read-Access Geo-Zone-Redundant Storage)**
   - **Disponibilité** : 99.9%
   - **Copies** : 6 copies (comme GZRS)
   - **Accès** : Lecture possible sur la région secondaire
   - **Région** : Deux régions avec accès en lecture
   - **Coût** : ~0.0404€/GB/mois (même prix que GZRS)
   - **Cas d'usage** : Applications critiques avec accès en lecture sur région secondaire

**Comparaison Détaillée des Coûts :**

| Type | Coût relatif | Coût estimé | Facteur multiplicateur | Avantage principal |
|------|-------------|-------------|----------------------|-------------------|
| LRS | 1x | ~0.0184€/GB/mois | Base | Économique |
| ZRS | 1.1x | ~0.0202€/GB/mois | +10% | Haute disponibilité |
| GRS | 2x | ~0.0368€/GB/mois | +100% | Protection géographique |
| RA-GRS | 2x | ~0.0368€/GB/mois | +100% | Protection + accès lecture |
| GZRS | 2.2x | ~0.0404€/GB/mois | +120% | Protection maximale |
| RA-GZRS | 2.2x | ~0.0404€/GB/mois | +120% | Protection max + accès lecture |

**Notes sur les coûts :**
- Les prix sont approximatifs et peuvent varier selon la région
- Les coûts incluent le stockage et la bande passante sortante
- Les opérations (lecture/écriture) ont des coûts supplémentaires
- Les prix sont basés sur le stockage GPv2 Standard
- Les réductions de volume peuvent s'appliquer pour de grandes quantités

### Storage Account Management

#### Aide Storage

**PowerShell - Aide Storage :**
```powershell
# Aide sur les commandes de stockage
Get-Help *Storage*

# Aide spécifique sur les comptes de stockage
Get-Help New-AzStorageAccount -Detailed

# Aide sur les blobs
Get-Help *StorageBlob* -Examples

# Aide sur les partages de fichiers
Get-Help *StorageShare* -Full

# Rechercher des commandes de stockage
Get-Help *StorageAccount*
```

**Azure CLI - Aide Storage :**
```bash
# Aide sur les comptes de stockage
az storage account --help

# Aide sur les blobs
az storage blob --help

# Aide sur les partages de fichiers
az storage share --help

# Aide sur une sous-commande spécifique
az storage account create --help

# Rechercher des commandes de stockage
az find "storage"
```

**Découverte des Commandes Storage :**
```powershell
# PowerShell - Commandes Storage disponibles
Get-Command *Storage*
Get-Command *StorageAccount*
Get-Command *StorageBlob*
Get-Command *StorageShare*
Get-Command *StorageContainer*
```

```bash
# Azure CLI - Explorer les commandes Storage
az storage --help
az storage account --help
az storage blob --help
az storage share --help
az storage container --help
```

#### Concepts des Comptes de Stockage

**Qu'est-ce qu'un Compte de Stockage ?**
Un compte de stockage Azure contient tous vos objets de données Azure Storage : blobs, fichiers, queues, tables et disques.

**Types de Comptes**
- **General Purpose v2** : Recommandé pour la plupart des scénarios
- **General Purpose v1** : Hérité, à éviter pour les nouveaux déploiements
- **Blob Storage** : Spécialisé pour le stockage d'objets uniquement
- **Premium Storage** : Pour les charges de travail intensives

#### Types Détaillés de Comptes de Stockage Azure

**1. General Purpose v2 (GPv2) - Recommandé**

**Caractéristiques :**
- **Performance** : Standard (HDD) et Premium (SSD) disponibles
- **Redondance** : Tous les types de redondance supportés
- **Services** : Blobs, Files, Queues, Tables, Disques
- **API** : REST API complète
- **Coût** : Tarification à la demande optimisée

**Avantages :**
- ✅ Fonctionnalités les plus récentes
- ✅ Tarification la plus économique
- ✅ Support de tous les services de stockage
- ✅ Hiérarchisation automatique des blobs
- ✅ Accès en lecture géo-redondant (RA-GRS/RA-GZRS)
- ✅ Azure Data Lake Storage Gen2

**Cas d'usage recommandés :**
- Nouvelles applications
- Migrations depuis GPv1
- Scénarios nécessitant toutes les fonctionnalités
- Optimisation des coûts

**2. General Purpose v1 (GPv1) - Hérité**

**Caractéristiques :**
- **Performance** : Standard uniquement (HDD)
- **Redondance** : LRS et GRS uniquement
- **Services** : Blobs, Files, Queues, Tables, Disques
- **API** : REST API complète
- **Coût** : Tarification classique

**Limitations :**
- ❌ Pas de support Premium (SSD)
- ❌ Pas de ZRS, GZRS, RA-GZRS
- ❌ Pas de hiérarchisation automatique
- ❌ Pas d'accès en lecture géo-redondant
- ❌ Pas d'Azure Data Lake Storage Gen2
- ❌ Moins de fonctionnalités de sécurité

**Cas d'usage (migration recommandée) :**
- Applications existantes (migrer vers GPv2)
- Scénarios simples sans besoins avancés

**3. Blob Storage - Spécialisé**

**Caractéristiques :**
- **Performance** : Standard (HDD) et Premium (SSD)
- **Redondance** : Tous les types supportés
- **Services** : Blobs uniquement
- **API** : REST API pour blobs
- **Coût** : Tarification optimisée pour blobs

**Avantages :**
- ✅ Optimisé pour le stockage d'objets
- ✅ Hiérarchisation automatique
- ✅ Accès en lecture géo-redondant
- ✅ Azure Data Lake Storage Gen2
- ✅ Coûts réduits pour stockage blob uniquement

**Cas d'usage :**
- Applications nécessitant uniquement du stockage blob
- Stockage de fichiers statiques (images, vidéos)
- Sauvegarde et archivage
- Big Data et analytics

**4. Premium Storage - Performance Élevée**

**Caractéristiques :**
- **Performance** : SSD uniquement (haute performance)
- **Redondance** : LRS uniquement
- **Services** : Blobs, Files, Disques
- **Latence** : < 1ms
- **IOPS** : Jusqu'à 20,000 IOPS par disque

**Avantages :**
- ✅ Performance SSD élevée
- ✅ Latence ultra-faible
- ✅ IOPS élevées
- ✅ Idéal pour VMs critiques
- ✅ Bases de données haute performance

**Cas d'usage :**
- Machines virtuelles critiques
- Bases de données (SQL Server, Oracle)
- Applications nécessitant haute performance
- Charges de travail intensives en I/O

**5. Premium Block Blob Storage - Nouveau**

**Caractéristiques :**
- **Performance** : SSD uniquement
- **Redundance** : LRS et ZRS
- **Services** : Blobs uniquement
- **Latence** : < 1ms
- **Throughput** : Jusqu'à 2,500 MB/s

**Avantages :**
- ✅ Performance SSD pour blobs
- ✅ Latence ultra-faible
- ✅ Throughput élevé
- ✅ Idéal pour analytics et ML
- ✅ Streaming de données haute performance

**Cas d'usage :**
- Analytics en temps réel
- Machine Learning
- Streaming de données
- Applications nécessitant haute performance pour blobs

**Comparaison Détaillée des Types de Comptes :**

| Type | Performance | Services | Redondance | Coût | Recommandation |
|------|-------------|----------|------------|------|----------------|
| GPv2 | Standard/Premium | Tous | Tous | Optimisé | ✅ Nouveau |
| GPv1 | Standard uniquement | Tous | LRS/GRS | Classique | ❌ Hérité |
| Blob Storage | Standard/Premium | Blobs uniquement | Tous | Optimisé | ✅ Spécialisé |
| Premium Storage | SSD uniquement | Blobs/Files/Disques | LRS | Élevé | ✅ Performance |
| Premium Block Blob | SSD uniquement | Blobs uniquement | LRS/ZRS | Élevé | ✅ Performance Blob |

**Migration GPv1 vers GPv2 :**

**Commandes de Migration :**
```bash
# Azure CLI - Migration GPv1 vers GPv2
az storage account update \
  --name "mystorageaccount" \
  --resource-group "myResourceGroup" \
  --set kind=StorageV2

# PowerShell - Migration GPv1 vers GPv2
Set-AzStorageAccount \
  -ResourceGroupName "myResourceGroup" \
  -Name "mystorageaccount" \
  -Kind StorageV2
```

**Vérification de la Migration :**
```bash
# Vérifier le type de compte
az storage account show \
  --name "mystorageaccount" \
  --resource-group "myResourceGroup" \
  --query "kind"

# PowerShell
Get-AzStorageAccount \
  -ResourceGroupName "myResourceGroup" \
  -Name "mystorageaccount" | Select-Object Kind
```

**Avantages de la Migration :**
- ✅ Accès aux nouvelles fonctionnalités
- ✅ Réduction des coûts (tarification optimisée)
- ✅ Support de tous les types de redondance
- ✅ Hiérarchisation automatique
- ✅ Meilleure sécurité

**Commandes de Création par Type :**

**GPv2 (Recommandé) :**
```bash
# Azure CLI
az storage account create \
  --name "mystorageaccount" \
  --resource-group "myResourceGroup" \
  --location "eastus" \
  --sku "Standard_LRS" \
  --kind "StorageV2"

# PowerShell
New-AzStorageAccount \
  -ResourceGroupName "myResourceGroup" \
  -Name "mystorageaccount" \
  -Location "East US" \
  -SkuName "Standard_LRS" \
  -Kind "StorageV2"
```

**Blob Storage :**
```bash
# Azure CLI
az storage account create \
  --name "myblobstorage" \
  --resource-group "myResourceGroup" \
  --location "eastus" \
  --sku "Standard_LRS" \
  --kind "BlobStorage"

# PowerShell
New-AzStorageAccount \
  -ResourceGroupName "myResourceGroup" \
  -Name "myblobstorage" \
  -Location "East US" \
  -SkuName "Standard_LRS" \
  -Kind "BlobStorage"
```

**Premium Storage :**
```bash
# Azure CLI
az storage account create \
  --name "mypremiumstorage" \
  --resource-group "myResourceGroup" \
  --location "eastus" \
  --sku "Premium_LRS" \
  --kind "StorageV2"

# PowerShell
New-AzStorageAccount \
  -ResourceGroupName "myResourceGroup" \
  -Name "mypremiumstorage" \
  -Location "East US" \
  -SkuName "Premium_LRS" \
  -Kind "StorageV2"
```

**Premium Block Blob Storage :**
```bash
# Azure CLI
az storage account create \
  --name "mypremiumblob" \
  --resource-group "myResourceGroup" \
  --location "eastus" \
  --sku "Premium_LRS" \
  --kind "BlockBlobStorage"

# PowerShell
New-AzStorageAccount \
  -ResourceGroupName "myResourceGroup" \
  -Name "mypremiumblob" \
  -Location "East US" \
  -SkuName "Premium_LRS" \
  -Kind "BlockBlobStorage"
```

**Sécurité**
- **Chiffrement au repos** : AES-256 automatique
- **Chiffrement en transit** : HTTPS obligatoire
- **Clés d'accès** : Authentification par clé
- **SAS (Shared Access Signatures)** : Accès temporaire et sécurisé

#### Storage Account Commands

**Azure CLI:**
```bash
# Create storage account
az storage account create --name "mystorageaccount" --resource-group "myResourceGroup" --location "eastus" --sku "Standard_LRS" --kind "StorageV2"

# List storage accounts
az storage account list --output table

# Show storage account
az storage account show --name "mystorageaccount" --resource-group "myResourceGroup"

# Update storage account
az storage account update --name "mystorageaccount" --resource-group "myResourceGroup" --sku "Standard_GRS"

# Delete storage account
az storage account delete --name "mystorageaccount" --resource-group "myResourceGroup"

# Get storage account keys
az storage account keys list --account-name "mystorageaccount" --resource-group "myResourceGroup"

# Regenerate storage account key
az storage account keys renew --account-name "mystorageaccount" --resource-group "myResourceGroup" --key "key1"
```

**PowerShell:**
```powershell
# Create storage account
New-AzStorageAccount -ResourceGroupName "myResourceGroup" -Name "mystorageaccount" -Location "East US" -SkuName "Standard_LRS" -Kind "StorageV2"

# List storage accounts
Get-AzStorageAccount

# Show storage account
Get-AzStorageAccount -ResourceGroupName "myResourceGroup" -Name "mystorageaccount"

# Update storage account
Set-AzStorageAccount -ResourceGroupName "myResourceGroup" -Name "mystorageaccount" -SkuName "Standard_GRS"

# Delete storage account
Remove-AzStorageAccount -ResourceGroupName "myResourceGroup" -Name "mystorageaccount"

# Get storage account keys
Get-AzStorageAccountKey -ResourceGroupName "myResourceGroup" -Name "mystorageaccount"

# Regenerate storage account key
New-AzStorageAccountKey -ResourceGroupName "myResourceGroup" -Name "mystorageaccount" -KeyName "key1"
```

### Blob Storage

#### Concepts Blob Storage

**Qu'est-ce que Blob Storage ?**
Azure Blob Storage est un service de stockage d'objets optimisé pour stocker de grandes quantités de données non structurées, comme des images, vidéos, documents, sauvegardes, etc.

**Types de Blobs**
- **Block Blobs** : Pour les fichiers volumineux (max 190.7 TB)
- **Page Blobs** : Pour les disques de VM et bases de données (max 8 TB)
- **Append Blobs** : Pour les opérations d'ajout uniquement

**Niveaux d'Accès (Access Tiers)**
- **Hot** : Accès fréquent, coût de stockage plus élevé
- **Cool** : Accès moins fréquent, coût de stockage réduit
- **Archive** : Accès rare, coût minimal, latence élevée

**Conteneurs et Organisation**
- **Conteneurs** : Organisation logique des blobs
- **Nommage** : Règles de nommage spécifiques
- **Métadonnées** : Informations personnalisées sur les blobs

**Sécurité et Accès**
- **Authentification** : Clés de compte ou SAS
- **CORS** : Cross-Origin Resource Sharing
- **Lifecycle Management** : Règles automatiques de transition

**Azure CLI:**
```bash
# Create container
az storage container create --name "mycontainer" --account-name "mystorageaccount"

# List containers
az storage container list --account-name "mystorageaccount" --output table

# Upload blob
az storage blob upload --file "localfile.txt" --container-name "mycontainer" --name "myblob.txt" --account-name "mystorageaccount"

# Download blob
az storage blob download --container-name "mycontainer" --name "myblob.txt" --file "downloadedfile.txt" --account-name "mystorageaccount"

# List blobs
az storage blob list --container-name "mycontainer" --account-name "mystorageaccount" --output table

# Delete blob
az storage blob delete --container-name "mycontainer" --name "myblob.txt" --account-name "mystorageaccount"

# Set blob access tier
az storage blob set-tier --container-name "mycontainer" --name "myblob.txt" --tier "Cool" --account-name "mystorageaccount"

# Generate SAS token
az storage blob generate-sas --container-name "mycontainer" --name "myblob.txt" --permissions "r" --expiry "2024-12-31" --account-name "mystorageaccount"
```

**PowerShell:**
```powershell
# Get storage context
$ctx = (Get-AzStorageAccount -ResourceGroupName "myResourceGroup" -Name "mystorageaccount").Context

# Create container
New-AzStorageContainer -Name "mycontainer" -Context $ctx

# List containers
Get-AzStorageContainer -Context $ctx

# Upload blob
Set-AzStorageBlobContent -File "localfile.txt" -Container "mycontainer" -Blob "myblob.txt" -Context $ctx

# Download blob
Get-AzStorageBlobContent -Container "mycontainer" -Blob "myblob.txt" -Destination "downloadedfile.txt" -Context $ctx

# List blobs
Get-AzStorageBlob -Container "mycontainer" -Context $ctx

# Delete blob
Remove-AzStorageBlob -Container "mycontainer" -Blob "myblob.txt" -Context $ctx

# Set blob access tier
Set-AzStorageBlobContent -File "localfile.txt" -Container "mycontainer" -Blob "myblob.txt" -Context $ctx -StandardBlobTier "Cool"
```

### Azure Files

#### Concepts Azure Files

**Qu'est-ce qu'Azure Files ?**
Azure Files offre des partages de fichiers entièrement managés dans le cloud, accessibles via le protocole SMB (Server Message Block) standard de l'industrie.

**Cas d'Usage**
- **Migration d'applications** : Applications existantes utilisant des partages de fichiers
- **Partage de données** : Partage de fichiers entre plusieurs VMs
- **Sauvegarde et récupération** : Stockage de sauvegardes
- **Développement et test** : Environnements de développement

**Types de Partages**
- **Standard** : Stockage HDD pour charges de travail générales
- **Premium** : Stockage SSD pour charges de travail intensives

**Connexion et Accès**
- **SMB** : Protocole standard pour Windows
- **NFS** : Protocole pour Linux (Premium uniquement)
- **REST API** : Accès programmatique
- **Azure File Sync** : Synchronisation avec serveurs locaux

**Sécurité**
- **Authentification** : Azure AD ou clés de stockage
- **Chiffrement** : Au repos et en transit
- **RBAC** : Contrôle d'accès granulaire

**Azure CLI:**
```bash
# Create file share
az storage share create --name "myfileshare" --account-name "mystorageaccount"

# List file shares
az storage share list --account-name "mystorageaccount" --output table

# Upload file
az storage file upload --share-name "myfileshare" --source "localfile.txt" --path "myfile.txt" --account-name "mystorageaccount"

# Download file
az storage file download --share-name "myfileshare" --path "myfile.txt" --dest "downloadedfile.txt" --account-name "mystorageaccount"

# List files
az storage file list --share-name "myfileshare" --account-name "mystorageaccount" --output table

# Delete file
az storage file delete --share-name "myfileshare" --path "myfile.txt" --account-name "mystorageaccount"

# Create directory
az storage directory create --share-name "myfileshare" --name "mydirectory" --account-name "mystorageaccount"
```

**PowerShell:**
```powershell
# Create file share
New-AzStorageShare -Name "myfileshare" -Context $ctx

# List file shares
Get-AzStorageShare -Context $ctx

# Upload file
Set-AzStorageFileContent -ShareName "myfileshare" -Source "localfile.txt" -Path "myfile.txt" -Context $ctx

# Download file
Get-AzStorageFileContent -ShareName "myfileshare" -Path "myfile.txt" -Destination "downloadedfile.txt" -Context $ctx

# List files
Get-AzStorageFile -ShareName "myfileshare" -Context $ctx

# Delete file
Remove-AzStorageFile -ShareName "myfileshare" -Path "myfile.txt" -Context $ctx

# Create directory
New-AzStorageDirectory -ShareName "myfileshare" -Path "mydirectory" -Context $ctx
```

---

## 🖥️ 3. Azure Compute Resources

### Concepts de Compute Azure

**Qu'est-ce que Compute Azure ?**
Azure Compute est un ensemble de services cloud qui fournit des ressources de calcul à la demande pour héberger, exécuter et gérer des applications et des charges de travail.

**Services Compute Principaux**
- **Virtual Machines** : Machines virtuelles IaaS
- **App Service** : Plateforme PaaS pour applications web
- **Container Instances** : Conteneurs sans serveur
- **Azure Functions** : Computing serverless
- **Azure Kubernetes Service** : Orchestration de conteneurs

**Avantages du Cloud Computing**
- **Évolutivité** : Montée en charge automatique
- **Flexibilité** : Ressources à la demande
- **Coût** : Paiement à l'usage
- **Disponibilité** : Haute disponibilité intégrée

### Virtual Machines

#### Aide Virtual Machines

**PowerShell - Aide VMs :**
```powershell
# Aide sur les commandes de VMs
Get-Help *VM*

# Aide spécifique sur la création de VM
Get-Help New-AzVM -Detailed

# Aide sur la gestion des VMs
Get-Help Get-AzVM -Examples

# Aide sur les disques de VM
Get-Help *VMDisk* -Full

# Rechercher des commandes VM
Get-Help *VirtualMachine*
```

**Azure CLI - Aide VMs :**
```bash
# Aide sur les VMs
az vm --help

# Aide sur la création de VM
az vm create --help

# Aide sur la gestion des VMs
az vm list --help

# Aide sur les disques
az vm disk --help

# Rechercher des commandes VM
az find "vm"
```

**Découverte des Commandes VMs :**
```powershell
# PowerShell - Commandes VM disponibles
Get-Command *VM*
Get-Command *VirtualMachine*
Get-Command *VMDisk*
Get-Command *VMImage*
Get-Command *VMSize*
```

```bash
# Azure CLI - Explorer les commandes VM
az vm --help
az vm disk --help
az vm image --help
az vm size --help
az vm extension --help
```

#### Concepts des Machines Virtuelles

**Qu'est-ce qu'une VM Azure ?**
Une machine virtuelle Azure est un service d'infrastructure as a service (IaaS) qui vous permet de créer et utiliser des machines virtuelles dans le cloud.

**Types de VMs**
- **General Purpose** : Équilibre CPU/mémoire (B, Dsv3, Dv3)
- **Memory Optimized** : Mémoire élevée (Esv3, Ev3, M)
- **Compute Optimized** : CPU élevé (Fsv2, F)
- **GPU** : Calcul intensif (NC, ND, NV)
- **High Performance Compute** : Calcul haute performance (H)

**Composants d'une VM**
- **OS Disk** : Disque système d'exploitation
- **Data Disks** : Disques de données
- **Network Interface** : Interface réseau
- **Public IP** : Adresse IP publique (optionnelle)
- **NSG** : Groupe de sécurité réseau

**Gestion du Cycle de Vie**
- **Création** : Déploiement initial
- **Démarrage/Arrêt** : Contrôle de l'état
- **Redimensionnement** : Changement de taille
- **Sauvegarde** : Protection des données
- **Suppression** : Nettoyage des ressources

#### VM Management Commands

**Azure CLI:**
```bash
# Create VM
az vm create --resource-group "myResourceGroup" --name "myVM" --image "Ubuntu2204" --admin-username "azureuser" --generate-ssh-keys --size "Standard_B2s"

# Create Windows VM
az vm create --resource-group "myResourceGroup" --name "myWindowsVM" --image "Win2022Datacenter" --admin-username "azureuser" --admin-password "MyPassword123!" --size "Standard_B2s"

# List VMs
az vm list --output table

# Show VM details
az vm show --resource-group "myResourceGroup" --name "myVM"

# Start VM
az vm start --resource-group "myResourceGroup" --name "myVM"

# Stop VM
az vm stop --resource-group "myResourceGroup" --name "myVM"

# Restart VM
az vm restart --resource-group "myResourceGroup" --name "myVM"

# Deallocate VM
az vm deallocate --resource-group "myResourceGroup" --name "myVM"

# Delete VM
az vm delete --resource-group "myResourceGroup" --name "myVM"

# Resize VM
az vm resize --resource-group "myResourceGroup" --name "myVM" --size "Standard_B4ms"

# List available VM sizes
az vm list-sizes --location "eastus" --output table

# List VM images
az vm image list --output table

# Get VM status
az vm get-instance-view --resource-group "myResourceGroup" --name "myVM" --query "instanceView.statuses[1].displayStatus"
```

**PowerShell:**
```powershell
# Create VM
$cred = Get-Credential
New-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM" -Location "East US" -VirtualNetworkName "myVnet" -SubnetName "mySubnet" -SecurityGroupName "myNetworkSecurityGroup" -PublicIpAddressName "myPublicIpAddress" -Credential $cred

# List VMs
Get-AzVM

# Show VM details
Get-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM"

# Start VM
Start-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM"

# Stop VM
Stop-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM"

# Restart VM
Restart-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM"

# Delete VM
Remove-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM"

# Resize VM
$vm = Get-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM"
$vm.HardwareProfile.VmSize = "Standard_B4ms"
Update-AzVM -VM $vm -ResourceGroupName "myResourceGroup"

# List available VM sizes
Get-AzVMSize -Location "East US"

# Get VM status
Get-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM" -Status
```

### VM Disks

#### Concepts des Disques Managés

**Qu'est-ce qu'un Disque Managé Azure ?**
Les disques managés Azure sont des disques durs virtuels (VHD) stockés comme objets de page dans Azure Storage, gérés par Azure pour simplifier la gestion des disques de VM.

**Types de Disques**
- **OS Disk** : Disque système d'exploitation (max 4 TB)
- **Data Disk** : Disque de données (max 32 TB)
- **Temporary Disk** : Disque temporaire (non persistant)

**Types de Performance**
- **Standard HDD** : Stockage économique pour charges de travail générales
- **Standard SSD** : Performance intermédiaire avec latence réduite
- **Premium SSD** : Haute performance pour charges de travail intensives
- **Ultra Disk** : Performance maximale pour charges critiques

**Tailles et Limites**
- **Taille minimale** : 4 GB
- **Taille maximale** : 32 TB (Premium SSD)
- **IOPS** : Dépend du type et de la taille
- **Throughput** : Dépend du type et de la taille

**Gestion des Disques**
- **Attachement/Détachement** : Dynamique sans arrêt de VM
- **Redimensionnement** : Augmentation possible, réduction limitée
- **Snapshots** : Sauvegardes ponctuelles
- **Encryption** : Chiffrement automatique au repos

**Azure CLI:**
```bash
# Create managed disk
az disk create --resource-group "myResourceGroup" --name "myDataDisk" --size-gb 128 --sku "Premium_LRS"

# List disks
az disk list --output table

# Attach disk to VM
az vm disk attach --resource-group "myResourceGroup" --vm-name "myVM" --name "myDataDisk"

# Detach disk from VM
az vm disk detach --resource-group "myResourceGroup" --vm-name "myVM" --name "myDataDisk"

# Update disk size
az disk update --resource-group "myResourceGroup" --name "myDataDisk" --size-gb 256

# Create snapshot
az snapshot create --resource-group "myResourceGroup" --name "mySnapshot" --source "myDataDisk"

# Create disk from snapshot
az disk create --resource-group "myResourceGroup" --name "myNewDisk" --source "mySnapshot"
```

**PowerShell:**
```powershell
# Create managed disk
$diskConfig = New-AzDiskConfig -Location "East US" -CreateOption "Empty" -DiskSizeGB 128 -SkuName "Premium_LRS"
New-AzDisk -ResourceGroupName "myResourceGroup" -DiskName "myDataDisk" -Disk $diskConfig

# List disks
Get-AzDisk

# Attach disk to VM
$vm = Get-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM"
$disk = Get-AzDisk -ResourceGroupName "myResourceGroup" -Name "myDataDisk"
$vm = Add-AzVMDataDisk -VM $vm -Name "myDataDisk" -CreateOption "Attach" -ManagedDiskId $disk.Id -Lun 1
Update-AzVM -VM $vm -ResourceGroupName "myResourceGroup"

# Detach disk from VM
$vm = Get-AzVM -ResourceGroupName "myResourceGroup" -Name "myVM"
Remove-AzVMDataDisk -VM $vm -Name "myDataDisk"
Update-AzVM -VM $vm -ResourceGroupName "myResourceGroup"

# Create snapshot
$disk = Get-AzDisk -ResourceGroupName "myResourceGroup" -Name "myDataDisk"
$snapshotConfig = New-AzSnapshotConfig -SourceUri $disk.Id -CreateOption "Copy" -Location "East US"
New-AzSnapshot -ResourceGroupName "myResourceGroup" -SnapshotName "mySnapshot" -Snapshot $snapshotConfig
```

### Azure App Service

#### Découverte des Commandes App Service

**PowerShell - Commandes App Service disponibles :**
```powershell
# PowerShell - Commandes App Service disponibles
Get-Command *WebApp*
Get-Command *AppService*
Get-Command *AppServicePlan*
Get-Command *WebAppSlot*
```

**Azure CLI - Explorer les commandes App Service :**
```bash
# Azure CLI - Explorer les commandes App Service
az webapp --help
az appservice --help
az appservice plan --help
az webapp deployment --help
```

#### Concepts Azure App Service

**Qu'est-ce qu'Azure App Service ?**
Azure App Service est une plateforme PaaS (Platform as a Service) qui vous permet de créer et déployer rapidement des applications web, des API REST et des applications mobiles sans gérer l'infrastructure sous-jacente.

**Types d'Applications**
- **Web Apps** : Applications web et sites web
- **API Apps** : APIs REST et microservices
- **Mobile Apps** : Applications mobiles backend
- **Function Apps** : Serverless computing
- **Static Web Apps** : Sites statiques avec API

**Plans de Service**
- **Free** : Développement et test (limitations)
- **Basic** : Applications de production simples
- **Standard** : Applications de production avec auto-scaling
- **Premium** : Applications critiques avec fonctionnalités avancées
- **Isolated** : Environnements isolés pour conformité

**Fonctionnalités Avancées**
- **Auto-scaling** : Mise à l'échelle automatique
- **Deployment Slots** : Environnements de staging
- **Custom Domains** : Domaines personnalisés
- **SSL/TLS** : Certificats SSL
- **Authentication** : Intégration Azure AD
- **Backup** : Sauvegardes automatiques

**Runtimes Supportés**
- **.NET** : .NET Core, .NET Framework
- **Java** : Java 8, 11, 17
- **Node.js** : Versions LTS
- **Python** : Python 3.x
- **PHP** : PHP 7.x, 8.x
- **Ruby** : Ruby 2.x

**Azure CLI:**
```bash
# Create App Service plan
az appservice plan create --name "myAppServicePlan" --resource-group "myResourceGroup" --sku "B1" --location "eastus"

# List App Service plans
az appservice plan list --output table

# Create web app
az webapp create --resource-group "myResourceGroup" --plan "myAppServicePlan" --name "myWebApp" --runtime "NODE|14-lts"

# List web apps
az webapp list --output table

# Show web app
az webapp show --resource-group "myResourceGroup" --name "myWebApp"

# Start web app
az webapp start --resource-group "myResourceGroup" --name "myWebApp"

# Stop web app
az webapp stop --resource-group "myResourceGroup" --name "myWebApp"

# Restart web app
az webapp restart --resource-group "myResourceGroup" --name "myWebApp"

# Delete web app
az webapp delete --resource-group "myResourceGroup" --name "myWebApp"

# Deploy code
az webapp deployment source config --resource-group "myResourceGroup" --name "myWebApp" --repo-url "https://github.com/user/repo" --branch "main"

# Create deployment slot
az webapp deployment slot create --resource-group "myResourceGroup" --name "myWebApp" --slot "staging"

# Swap deployment slots
az webapp deployment slot swap --resource-group "myResourceGroup" --name "myWebApp" --slot "staging" --target-slot "production"
```

**PowerShell:**
```powershell
# Create App Service plan
New-AzAppServicePlan -ResourceGroupName "myResourceGroup" -Name "myAppServicePlan" -Location "East US" -Tier "Basic" -NumberofWorkers 1 -WorkerSize "Small"

# List App Service plans
Get-AzAppServicePlan

# Create web app
New-AzWebApp -ResourceGroupName "myResourceGroup" -Name "myWebApp" -Location "East US" -AppServicePlan "myAppServicePlan"

# List web apps
Get-AzWebApp

# Show web app
Get-AzWebApp -ResourceGroupName "myResourceGroup" -Name "myWebApp"

# Start web app
Start-AzWebApp -ResourceGroupName "myResourceGroup" -Name "myWebApp"

# Stop web app
Stop-AzWebApp -ResourceGroupName "myResourceGroup" -Name "myWebApp"

# Restart web app
Restart-AzWebApp -ResourceGroupName "myResourceGroup" -Name "myWebApp"

# Delete web app
Remove-AzWebApp -ResourceGroupName "myResourceGroup" -Name "myWebApp"

# Create deployment slot
New-AzWebAppSlot -ResourceGroupName "myResourceGroup" -Name "myWebApp" -Slot "staging"

# Swap deployment slots
Switch-AzWebAppSlot -ResourceGroupName "myResourceGroup" -Name "myWebApp" -SourceSlotName "staging" -DestinationSlotName "production"
```

### Container Instances

#### Découverte des Commandes Container

**PowerShell - Commandes Container disponibles :**
```powershell
# PowerShell - Commandes Container disponibles
Get-Command *Container*
Get-Command *ContainerGroup*
Get-Command *ContainerInstance*
```

**Azure CLI - Explorer les commandes Container :**
```bash
# Azure CLI - Explorer les commandes Container
az container --help
az container group --help
az container instance --help
az acr --help  # Azure Container Registry
```

#### Concepts Azure Container Instances

**Qu'est-ce qu'Azure Container Instances (ACI) ?**
Azure Container Instances est un service serverless qui vous permet de déployer des conteneurs dans Azure sans gérer de serveurs virtuels ou d'orchestrateur.

**Avantages d'ACI**
- **Serverless** : Pas de gestion d'infrastructure
- **Rapidité** : Déploiement en secondes
- **Coût** : Paiement uniquement pendant l'exécution
- **Simplicité** : Interface simple pour conteneurs simples

**Cas d'Usage**
- **Traitement par lots** : Jobs de traitement de données
- **Microservices** : Services simples et indépendants
- **DevOps** : Pipelines CI/CD
- **Test et développement** : Environnements temporaires
- **Tâches événementielles** : Traitement d'événements

**Types de Conteneurs**
- **Linux** : Conteneurs Linux
- **Windows** : Conteneurs Windows
- **Multi-container** : Groupes de conteneurs

**Ressources et Limites**
- **CPU** : 1-4 vCPUs
- **Mémoire** : 1-16 GB
- **Stockage** : Volumes persistants
- **Réseau** : IP publique ou privée

**Sécurité**
- **Isolation** : Conteneurs isolés
- **Secrets** : Gestion des secrets
- **Identités managées** : Authentification Azure AD

**Azure CLI:**
```bash
# Create container instance
az container create --resource-group "myResourceGroup" --name "mycontainer" --image "mcr.microsoft.com/azuredocs/aci-helloworld" --dns-name-label "my-container-dns" --ports 80

# List container instances
az container list --output table

# Show container instance
az container show --resource-group "myResourceGroup" --name "mycontainer"

# Get container logs
az container logs --resource-group "myResourceGroup" --name "mycontainer"

# Execute command in container
az container exec --resource-group "myResourceGroup" --name "mycontainer" --exec-command "/bin/bash"

# Delete container instance
az container delete --resource-group "myResourceGroup" --name "mycontainer"

# Create container with environment variables
az container create --resource-group "myResourceGroup" --name "mycontainer" --image "myimage" --environment-variables "ENV_VAR1=value1" "ENV_VAR2=value2"

# Create container with persistent storage
az container create --resource-group "myResourceGroup" --name "mycontainer" --image "myimage" --azure-file-volume-account-name "mystorageaccount" --azure-file-volume-account-key "key" --azure-file-volume-share-name "myshare" --azure-file-volume-mount-path "/mnt/azurefile"
```

**PowerShell:**
```powershell
# Create container instance
New-AzContainerGroup -ResourceGroupName "myResourceGroup" -Name "mycontainer" -Image "mcr.microsoft.com/azuredocs/aci-helloworld" -Location "East US" -DnsNameLabel "my-container-dns" -Port 80

# List container instances
Get-AzContainerGroup

# Show container instance
Get-AzContainerGroup -ResourceGroupName "myResourceGroup" -Name "mycontainer"

# Get container logs
Get-AzContainerInstanceLog -ResourceGroupName "myResourceGroup" -ContainerGroupName "mycontainer" -ContainerName "mycontainer"

# Delete container instance
Remove-AzContainerGroup -ResourceGroupName "myResourceGroup" -Name "mycontainer"
```

---

## 🌐 4. Virtual Networking

### Concepts de Réseau Azure

**Qu'est-ce qu'Azure Virtual Network ?**
Azure Virtual Network (VNet) est le service de mise en réseau fondamental d'Azure qui permet aux ressources Azure de communiquer entre elles, avec Internet et avec les réseaux locaux.

**Composants du Réseau Azure**
- **Virtual Networks** : Réseaux privés dans le cloud
- **Subnets** : Segments de réseau pour organiser les ressources
- **Network Security Groups** : Filtrage du trafic réseau
- **Route Tables** : Contrôle du routage
- **DNS** : Résolution de noms
- **Load Balancers** : Distribution de charge
- **Application Gateways** : Load balancing de couche 7

**Avantages du Réseau Azure**
- **Isolation** : Séparation logique des ressources
- **Sécurité** : Contrôle granulaire du trafic
- **Connectivité** : Liaison avec réseaux locaux
- **Évolutivité** : Adaptation aux besoins
- **Performance** : Optimisation du trafic

### Virtual Networks

#### Aide Virtual Networks

**PowerShell - Aide Réseau :**
```powershell
# Aide sur les commandes de réseau
Get-Help *Network*

# Aide spécifique sur les VNets
Get-Help New-AzVirtualNetwork -Detailed

# Aide sur les NSGs
Get-Help *NetworkSecurityGroup* -Examples

# Aide sur les sous-réseaux
Get-Help *Subnet* -Full

# Rechercher des commandes réseau
Get-Help *VirtualNetwork*
```

**Azure CLI - Aide Réseau :**
```bash
# Aide sur les VNets
az network vnet --help

# Aide sur les NSGs
az network nsg --help

# Aide sur les sous-réseaux
az network vnet subnet --help

# Aide sur une sous-commande spécifique
az network vnet create --help

# Rechercher des commandes réseau
az find "network"
```

**Découverte des Commandes Réseau :**
```powershell
# PowerShell - Commandes Network disponibles
Get-Command *Network*
Get-Command *VirtualNetwork*
Get-Command *NetworkSecurityGroup*
Get-Command *Subnet*
Get-Command *NetworkInterface*
```

```bash
# Azure CLI - Explorer les commandes Network
az network --help
az network vnet --help
az network nsg --help
az network nic --help
az network lb --help
```

#### Concepts des Réseaux Virtuels

**Qu'est-ce qu'un VNet ?**
Un Virtual Network (VNet) est une représentation de votre propre réseau dans le cloud Azure. Il vous permet de contrôler complètement votre environnement réseau.

**Architecture VNet**
- **Address Space** : Plage d'adresses IP privées
- **Subnets** : Segments de réseau pour organiser les ressources
- **DNS Settings** : Configuration DNS personnalisée
- **Peering** : Connexion entre VNets
- **Service Endpoints** : Accès privé aux services Azure

**Types d'Adressage**
- **RFC 1918** : Adresses privées (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16)
- **RFC 6598** : Adresses partagées (100.64.0.0/10)
- **Plages personnalisées** : Selon les besoins

**Segmentation Réseau**
- **Subnets** : Division logique du VNet
- **NSG** : Contrôle d'accès par subnet
- **Route Tables** : Routage personnalisé
- **Service Endpoints** : Accès direct aux services Azure

**Azure CLI:**
```bash
# Create virtual network
az network vnet create --resource-group "myResourceGroup" --name "myVnet" --address-prefix "10.0.0.0/16" --subnet-name "mySubnet" --subnet-prefix "10.0.1.0/24"

# List virtual networks
az network vnet list --output table

# Show virtual network
az network vnet show --resource-group "myResourceGroup" --name "myVnet"

# Update virtual network
az network vnet update --resource-group "myResourceGroup" --name "myVnet" --dns-servers "8.8.8.8" "8.8.4.4"

# Delete virtual network
az network vnet delete --resource-group "myResourceGroup" --name "myVnet"

# Create subnet
az network vnet subnet create --resource-group "myResourceGroup" --vnet-name "myVnet" --name "mySubnet2" --address-prefix "10.0.2.0/24"

# List subnets
az network vnet subnet list --resource-group "myResourceGroup" --vnet-name "myVnet" --output table

# Update subnet
az network vnet subnet update --resource-group "myResourceGroup" --vnet-name "myVnet" --name "mySubnet" --network-security-group "myNSG"

# Delete subnet
az network vnet subnet delete --resource-group "myResourceGroup" --vnet-name "myVnet" --name "mySubnet2"
```

**PowerShell:**
```powershell
# Create virtual network
$subnet = New-AzVirtualNetworkSubnetConfig -Name "mySubnet" -AddressPrefix "10.0.1.0/24"
New-AzVirtualNetwork -ResourceGroupName "myResourceGroup" -Location "East US" -Name "myVnet" -AddressPrefix "10.0.0.0/16" -Subnet $subnet

# List virtual networks
Get-AzVirtualNetwork

# Show virtual network
Get-AzVirtualNetwork -ResourceGroupName "myResourceGroup" -Name "myVnet"

# Update virtual network
$vnet = Get-AzVirtualNetwork -ResourceGroupName "myResourceGroup" -Name "myVnet"
$vnet.DhcpOptions.DnsServers = @("8.8.8.8", "8.8.4.4")
Set-AzVirtualNetwork -VirtualNetwork $vnet

# Delete virtual network
Remove-AzVirtualNetwork -ResourceGroupName "myResourceGroup" -Name "myVnet"

# Create subnet
$vnet = Get-AzVirtualNetwork -ResourceGroupName "myResourceGroup" -Name "myVnet"
Add-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name "mySubnet2" -AddressPrefix "10.0.2.0/24"
Set-AzVirtualNetwork -VirtualNetwork $vnet

# List subnets
Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet

# Delete subnet
$vnet = Get-AzVirtualNetwork -ResourceGroupName "myResourceGroup" -Name "myVnet"
Remove-AzVirtualNetworkSubnetConfig -VirtualNetwork $vnet -Name "mySubnet2"
Set-AzVirtualNetwork -VirtualNetwork $vnet
```

### Network Security Groups

#### Concepts des Groupes de Sécurité Réseau

**Qu'est-ce qu'un NSG ?**
Un Network Security Group (NSG) est un service de filtrage du trafic réseau qui permet de contrôler l'accès réseau aux ressources Azure en autorisant ou refusant le trafic réseau.

**Fonctionnement des NSG**
- **Règles de sécurité** : Autoriser ou refuser le trafic
- **Direction** : Inbound (entrant) ou Outbound (sortant)
- **Priorité** : Ordre d'évaluation des règles (100-4096)
- **Évaluation** : Première règle correspondante appliquée

**Types de Règles**
- **Règles de sécurité** : Contrôle du trafic par port/protocole
- **Règles par défaut** : Règles système automatiques
- **Règles personnalisées** : Règles définies par l'utilisateur

**Composants des Règles**
- **Nom** : Identifiant unique de la règle
- **Priorité** : Ordre d'évaluation (plus petit = plus prioritaire)
- **Source** : Adresse IP source ou tag de service
- **Destination** : Adresse IP destination
- **Port** : Port source et destination
- **Protocole** : TCP, UDP, ICMP, ou Any
- **Action** : Allow ou Deny

**Association des NSG**
- **Subnet** : Application à toutes les ressources du subnet
- **NIC** : Application à une interface réseau spécifique
- **VM** : Application via la NIC de la VM

**Bonnes Pratiques**
- **Principe du moindre privilège** : Autoriser uniquement le trafic nécessaire
- **Règles spécifiques** : Éviter les règles trop larges
- **Documentation** : Documenter le but de chaque règle
- **Test** : Tester les règles avant production

**Azure CLI:**
```bash
# Create NSG
az network nsg create --resource-group "myResourceGroup" --name "myNSG"

# List NSGs
az network nsg list --output table

# Show NSG
az network nsg show --resource-group "myResourceGroup" --name "myNSG"

# Create NSG rule
az network nsg rule create --resource-group "myResourceGroup" --nsg-name "myNSG" --name "AllowHTTP" --protocol "TCP" --priority 1000 --destination-port-range "80" --access "Allow"

# List NSG rules
az network nsg rule list --resource-group "myResourceGroup" --nsg-name "myNSG" --output table

# Update NSG rule
az network nsg rule update --resource-group "myResourceGroup" --nsg-name "myNSG" --name "AllowHTTP" --priority 1001

# Delete NSG rule
az network nsg rule delete --resource-group "myResourceGroup" --nsg-name "myNSG" --name "AllowHTTP"

# Associate NSG with subnet
az network vnet subnet update --resource-group "myResourceGroup" --vnet-name "myVnet" --name "mySubnet" --network-security-group "myNSG"

# Associate NSG with NIC
az network nic update --resource-group "myResourceGroup" --name "myNIC" --network-security-group "myNSG"
```

**PowerShell:**
```powershell
# Create NSG
New-AzNetworkSecurityGroup -ResourceGroupName "myResourceGroup" -Location "East US" -Name "myNSG"

# List NSGs
Get-AzNetworkSecurityGroup

# Show NSG
Get-AzNetworkSecurityGroup -ResourceGroupName "myResourceGroup" -Name "myNSG"

# Create NSG rule
$nsg = Get-AzNetworkSecurityGroup -ResourceGroupName "myResourceGroup" -Name "myNSG"
Add-AzNetworkSecurityRuleConfig -NetworkSecurityGroup $nsg -Name "AllowHTTP" -Description "Allow HTTP" -Access "Allow" -Protocol "TCP" -Direction "Inbound" -Priority 1000 -SourceAddressPrefix "*" -SourcePortRange "*" -DestinationAddressPrefix "*" -DestinationPortRange "80"
Set-AzNetworkSecurityGroup -NetworkSecurityGroup $nsg

# List NSG rules
Get-AzNetworkSecurityRuleConfig -NetworkSecurityGroup $nsg

# Update NSG rule
$nsg = Get-AzNetworkSecurityGroup -ResourceGroupName "myResourceGroup" -Name "myNSG"
Set-AzNetworkSecurityRuleConfig -NetworkSecurityGroup $nsg -Name "AllowHTTP" -Priority 1001 -Access "Allow" -Protocol "TCP" -Direction "Inbound" -SourceAddressPrefix "*" -SourcePortRange "*" -DestinationAddressPrefix "*" -DestinationPortRange "80"
Set-AzNetworkSecurityGroup -NetworkSecurityGroup $nsg

---

## 📊 5. Monitor and Back Up Azure Resources

### Concepts de Surveillance Azure

#### Aide Surveillance

**PowerShell - Aide Monitor :**
```powershell
# Aide sur les commandes de surveillance
Get-Help *Monitor*

# Aide spécifique sur les métriques
Get-Help Get-AzMetric -Detailed

# Aide sur les alertes
Get-Help *Alert* -Examples

# Aide sur les logs
Get-Help Get-AzLog -Full

# Rechercher des commandes de surveillance
Get-Help *Monitor*
```

**Azure CLI - Aide Monitor :**
```bash
# Aide sur Azure Monitor
az monitor --help

# Aide sur les métriques
az monitor metrics --help

# Aide sur les alertes
az monitor metrics alert --help

# Aide sur les logs d'activité
az monitor activity-log --help

# Rechercher des commandes de surveillance
az find "monitor"
```

**Découverte des Commandes Monitor :**
```powershell
# PowerShell - Commandes Monitor disponibles
Get-Command *Monitor*
Get-Command *Metric*
Get-Command *Alert*
Get-Command *Log*
Get-Command *Diagnostic*
```

```bash
# Azure CLI - Explorer les commandes Monitor
az monitor --help
az monitor metrics --help
az monitor alert --help
az monitor log --help
az monitor diagnostic --help
```

**Qu'est-ce qu'Azure Monitor ?**
Azure Monitor est une plateforme complète de surveillance qui collecte, analyse et agit sur les données de télémétrie de vos applications et ressources Azure.

**Composants d'Azure Monitor**
- **Metrics** : Données numériques sur les performances
- **Logs** : Données textuelles détaillées
- **Alerts** : Notifications basées sur des conditions
- **Dashboards** : Visualisations personnalisées
- **Workbooks** : Rapports interactifs
- **Application Insights** : Surveillance des applications

**Types de Surveillance**
- **Infrastructure** : Surveillance des ressources Azure
- **Application** : Surveillance du code d'application
- **Network** : Surveillance du trafic réseau
- **Security** : Surveillance de la sécurité

### Azure Monitor Commands

**Azure CLI:**
```bash
# List metrics
az monitor metrics list --resource "/subscriptions/subscription-id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM" --metric "Percentage CPU"

# Get activity logs
az monitor activity-log list --start-time "2024-01-01T00:00:00Z" --end-time "2024-01-02T00:00:00Z"

# Create alert rule
az monitor metrics alert create --name "HighCPUAlert" --resource-group "myResourceGroup" --scopes "/subscriptions/subscription-id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM" --condition "avg Percentage CPU > 80" --description "Alert when CPU usage is high"

# List alert rules
az monitor metrics alert list --resource-group "myResourceGroup"

# Get diagnostic settings
az monitor diagnostic-settings list --resource "/subscriptions/subscription-id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM"
```

**PowerShell:**
```powershell
# Get metrics
Get-AzMetric -ResourceId "/subscriptions/subscription-id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM" -MetricName "Percentage CPU" -TimeGrain "PT1H"

# Get activity logs
Get-AzLog -ResourceGroupName "myResourceGroup" -StartTime (Get-Date).AddDays(-1) -EndTime (Get-Date)

# Create alert rule
Add-AzMetricAlertRule -Name "HighCPUAlert" -Location "East US" -ResourceGroup "myResourceGroup" -TargetResourceId "/subscriptions/subscription-id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM" -MetricName "Percentage CPU" -Operator "GreaterThan" -Threshold 80

# Get diagnostic settings
Get-AzDiagnosticSetting -ResourceId "/subscriptions/subscription-id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM"
```

### Concepts de Sauvegarde Azure

**Qu'est-ce qu'Azure Backup ?**
Azure Backup est un service de sauvegarde cloud qui protège vos données dans Azure et sur site avec une solution de sauvegarde simple, sécurisée et économique.

**Types de Sauvegarde**
- **VM Backup** : Sauvegarde complète des machines virtuelles
- **File/Folder Backup** : Sauvegarde de fichiers spécifiques
- **SQL Backup** : Sauvegarde de bases de données SQL
- **SAP HANA Backup** : Sauvegarde de bases SAP HANA
- **Azure Files Backup** : Sauvegarde de partages de fichiers

**Stratégies de Sauvegarde**
- **Rétention** : Durée de conservation des sauvegardes
- **Fréquence** : Fréquence des sauvegardes
- **Type** : Complète, incrémentielle, différentielle
- **Compression** : Réduction de l'espace de stockage

**Récupération**
- **Point de récupération** : Moment précis de restauration
- **Récupération complète** : Restauration de toute la VM
- **Récupération de fichiers** : Restauration de fichiers spécifiques
- **Récupération croisée** : Restauration dans une autre région

### Azure Backup Commands

#### Découverte des Commandes Backup

**PowerShell - Commandes Backup disponibles :**
```powershell
# PowerShell - Commandes Backup disponibles
Get-Command *Backup*
Get-Command *Recovery*
Get-Command *Vault*
Get-Command *RecoveryServices*
```

**Azure CLI - Explorer les commandes Backup :**
```bash
# Azure CLI - Explorer les commandes Backup
az backup --help
az backup vault --help
az backup protection --help
az backup recoverypoint --help
```

**Azure CLI:**
```bash
# Create recovery services vault
az backup vault create --name "myVault" --resource-group "myResourceGroup" --location "eastus"

# List recovery services vaults
az backup vault list --output table

# Enable backup for VM
az backup protection enable-for-vm --resource-group "myResourceGroup" --vault-name "myVault" --vm "myVM" --policy-name "DefaultPolicy"

# List backup items
az backup item list --resource-group "myResourceGroup" --vault-name "myVault" --output table

# Create backup policy
az backup protection set-policy --resource-group "myResourceGroup" --vault-name "myVault" --policy-name "MyPolicy" --backup-management-type "AzureIaasVM" --workload-type "VM" --schedule-policy "schedule.json" --retention-policy "retention.json"

# Trigger backup
az backup protection backup-now --resource-group "myResourceGroup" --vault-name "myVault" --item-name "myVM"

# List recovery points
az backup recoverypoint list --resource-group "myResourceGroup" --vault-name "myVault" --item-name "myVM" --output table

# Restore VM
az backup restore restore-disks --resource-group "myResourceGroup" --vault-name "myVault" --item-name "myVM" --rp-name "recovery-point-name" --storage-account "mystorageaccount"
```

**PowerShell:**
```powershell
# Create recovery services vault
New-AzRecoveryServicesVault -Name "myVault" -ResourceGroupName "myResourceGroup" -Location "East US"

# List recovery services vaults
Get-AzRecoveryServicesVault

# Enable backup for VM
$vault = Get-AzRecoveryServicesVault -Name "myVault"
$policy = Get-AzRecoveryServicesBackupProtectionPolicy -VaultId $vault.ID -Name "DefaultPolicy"
Enable-AzRecoveryServicesBackupProtection -Item $vm -Policy $policy -VaultId $vault.ID

# List backup items
Get-AzRecoveryServicesBackupItem -VaultId $vault.ID -BackupManagementType "AzureVM"

# Create backup policy
$schedule = New-AzRecoveryServicesBackupSchedulePolicyObject -WorkloadType "AzureVM" -ScheduleType "Daily" -RetentionDuration 30
$retention = New-AzRecoveryServicesBackupRetentionPolicyObject -WorkloadType "AzureVM" -RetentionType "Daily" -RetentionCount 30
New-AzRecoveryServicesBackupProtectionPolicy -VaultId $vault.ID -Name "MyPolicy" -WorkloadType "AzureVM" -SchedulePolicy $schedule -RetentionPolicy $retention

# Trigger backup
Backup-AzRecoveryServicesBackupItem -Item $item -VaultId $vault.ID

# List recovery points
Get-AzRecoveryServicesBackupRecoveryPoint -Item $item -VaultId $vault.ID

# Restore VM
Restore-AzRecoveryServicesBackupItem -RecoveryPoint $rp -StorageAccountName "mystorageaccount" -StorageAccountResourceGroupName "myResourceGroup" -VaultId $vault.ID
```

---

## 🎯 Conseils pour l'Examen AZ-104

### Découverte Complète des Commandes

**PowerShell - Toutes les commandes Az par domaine :**
```powershell
# Identités et Gouvernance
Get-Command *ADUser*        # Utilisateurs Azure AD
Get-Command *ADGroup*        # Groupes Azure AD
Get-Command *Role*           # RBAC
Get-Command *Policy*         # Azure Policy

# Stockage
Get-Command *Storage*        # Stockage Azure
Get-Command *StorageAccount* # Comptes de stockage
Get-Command *StorageBlob*    # Blob Storage
Get-Command *StorageShare*   # File Storage

# Compute
Get-Command *VM*             # Machines virtuelles
Get-Command *WebApp*         # App Service
Get-Command *Container*      # Container Instances
Get-Command *AppService*     # App Service Plans

# Réseau
Get-Command *Network*        # Réseau Azure
Get-Command *VirtualNetwork* # VNets
Get-Command *NetworkSecurityGroup* # NSGs
Get-Command *Subnet*         # Sous-réseaux

# Surveillance
Get-Command *Monitor*        # Azure Monitor
Get-Command *Metric*         # Métriques
Get-Command *Alert*          # Alertes
Get-Command *Backup*         # Sauvegarde
```

**Azure CLI - Tous les groupes de commandes :**
```bash
# Identités et Gouvernance
az ad --help                 # Azure Active Directory
az role --help               # RBAC
az policy --help             # Azure Policy

# Stockage
az storage --help            # Stockage Azure
az storage account --help    # Comptes de stockage
az storage blob --help       # Blob Storage
az storage share --help      # File Storage

# Compute
az vm --help                 # Machines virtuelles
az webapp --help             # App Service
az container --help          # Container Instances
az appservice --help         # App Service Plans

# Réseau
az network --help            # Réseau Azure
az network vnet --help       # VNets
az network nsg --help        # NSGs
az network subnet --help     # Sous-réseaux

# Surveillance
az monitor --help            # Azure Monitor
az backup --help             # Azure Backup
az metrics --help            # Métriques
az alert --help              # Alertes
```

### Stratégie de Révision
1. **Comprendre les concepts** : Ne pas seulement mémoriser les commandes
2. **Pratiquer** : Utiliser Azure Portal et CLI/PowerShell
3. **Scénarios** : Étudier les cas d'usage réels
4. **Limites** : Connaître les limites des services
5. **Sécurité** : Focus sur les bonnes pratiques de sécurité

### Points Clés par Domaine
- **Identités** : RBAC, Azure AD, MFA, Conditional Access
- **Stockage** : Types de stockage, redondance, sécurité
- **Compute** : VMs, App Service, conteneurs, scaling
- **Réseau** : VNets, NSGs, load balancing, connectivité
- **Surveillance** : Azure Monitor, logs, alertes, sauvegarde

### Ressources Recommandées
- Documentation officielle Microsoft
- Labs pratiques Azure
- Examens blancs
- Communauté Azure
