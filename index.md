---
layout: default
title: AZ-104 Guide - Microsoft Azure Administrator
---

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

## 🏷️ Azure Tags - Organisation et Gouvernance

### Qu'est-ce qu'un Tag Azure ?
Les tags Azure sont des paires clé-valeur qui vous permettent de catégoriser et organiser vos ressources Azure pour la gestion, la facturation et l'optimisation.

### Avantages des Tags
- **Organisation** : Catégorisation logique des ressources
- **Facturation** : Suivi des coûts par projet/département
- **Gouvernance** : Application de politiques et contrôles
- **Automatisation** : Scripts basés sur les tags
- **Conformité** : Respect des standards organisationnels

### Bonnes Pratiques de Tagging
- **Convention de nommage** : Standardiser les noms de tags
- **Tags obligatoires** : Environment, Project, Owner, CostCenter
- **Valeurs cohérentes** : Utiliser des valeurs standardisées
- **Documentation** : Documenter la stratégie de tagging
- **Audit régulier** : Vérifier la conformité des tags

### Commandes de Gestion des Tags

**Azure CLI:**
```bash
# Ajouter des tags à une ressource
az resource tag --tags "Environment=Production" "Project=WebApp" "Owner=ITTeam" --name "myVM" --resource-group "myResourceGroup" --resource-type "Microsoft.Compute/virtualMachines"

# Lister les tags d'une ressource
az resource show --name "myVM" --resource-group "myResourceGroup" --resource-type "Microsoft.Compute/virtualMachines" --query "tags"

# Mettre à jour les tags
az resource tag --tags "Environment=Production" "Project=WebApp" "Owner=ITTeam" "CostCenter=IT001" --name "myVM" --resource-group "myResourceGroup" --resource-type "Microsoft.Compute/virtualMachines"

# Supprimer des tags
az resource tag --tags "Environment=Production" "Project=WebApp" --name "myVM" --resource-group "myResourceGroup" --resource-type "Microsoft.Compute/virtualMachines"

# Lister toutes les ressources avec un tag spécifique
az resource list --tag "Environment=Production" --output table

# Lister toutes les ressources d'un groupe avec leurs tags
az resource list --resource-group "myResourceGroup" --query "[].{Name:name, Type:type, Tags:tags}" --output table
```

**PowerShell:**
```powershell
# Ajouter des tags à une ressource
$tags = @{
    "Environment" = "Production"
    "Project" = "WebApp"
    "Owner" = "ITTeam"
}
Set-AzResource -ResourceId "/subscriptions/subscription-id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM" -Tag $tags

# Obtenir les tags d'une ressource
$resource = Get-AzResource -ResourceId "/subscriptions/subscription-id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM"
$resource.Tags

# Mettre à jour les tags
$tags = @{
    "Environment" = "Production"
    "Project" = "WebApp"
    "Owner" = "ITTeam"
    "CostCenter" = "IT001"
}
Set-AzResource -ResourceId "/subscriptions/subscription-id/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM" -Tag $tags

# Lister toutes les ressources avec un tag spécifique
Get-AzResource -Tag @{"Environment"="Production"}

# Lister toutes les ressources d'un groupe avec leurs tags
Get-AzResource -ResourceGroupName "myResourceGroup" | Select-Object Name, ResourceType, @{Name="Tags";Expression={$_.Tags}}
```

### Exemples de Tags Recommandés

**Tags Organisationnels:**
- `Environment`: Production, Development, Testing, Staging
- `Project`: Nom du projet ou application
- `Department`: IT, Finance, Marketing, HR
- `CostCenter`: Code de centre de coût
- `Owner`: Responsable de la ressource

**Tags Techniques:**
- `Backup`: Yes, No, Daily, Weekly
- `SecurityLevel`: Public, Internal, Confidential, Restricted
- `Compliance`: SOX, HIPAA, GDPR, PCI
- `DataClassification`: Public, Internal, Confidential, Restricted

**Tags Opérationnels:**
- `MaintenanceWindow`: Heures de maintenance
- `AutoShutdown`: Yes, No
- `Monitoring`: Enabled, Disabled
- `Alerting`: Critical, Warning, Info

---

## 📖 [Lire le Guide Complet](README.md)

Ce guide complet couvre tous les domaines de l'examen AZ-104 avec des exemples pratiques, des commandes PowerShell et Azure CLI, et des conseils pour la préparation à l'examen.

### 🚀 Fonctionnalités du Guide

- ✅ **Commandes PowerShell et Azure CLI** pour chaque domaine
- ✅ **Exemples pratiques** et cas d'usage réels
- ✅ **Concepts fondamentaux** expliqués en détail
- ✅ **Conseils d'examen** et stratégies de révision
- ✅ **Découverte des commandes** avec Get-Help et --help
- ✅ **Migration des commandes** dépréciées vers les modernes
- ✅ **Gestion des tags** pour l'organisation et la gouvernance

### 📚 Sections Principales

1. **Azure Identities and Governance** - RBAC, Azure AD, Azure Policy, Tags
2. **Azure Storage** - Blob Storage, File Storage, Storage Accounts
3. **Azure Compute** - VMs, App Service, Container Instances
4. **Virtual Networking** - VNets, NSGs, Load Balancers
5. **Monitor and Backup** - Azure Monitor, Azure Backup

---

## 🔗 Liens Utiles

- [Documentation officielle Microsoft Azure](https://docs.microsoft.com/azure/)
- [Page de certification AZ-104](https://docs.microsoft.com/certifications/exams/az-104)
- [Labs pratiques Azure](https://docs.microsoft.com/azure/developer/azure-developer-cli/tutorial-quickstart)
- [Azure CLI Documentation](https://docs.microsoft.com/cli/azure/)
- [Azure PowerShell Documentation](https://docs.microsoft.com/powershell/azure/)
- [Guide des tags Azure](https://docs.microsoft.com/azure/azure-resource-manager/management/tag-resources)

---

*Dernière mise à jour : {{ site.time | date: "%B %d, %Y" }}* 