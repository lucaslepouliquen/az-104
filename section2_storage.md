# AZ-104 - Section 2: Implement and Manage Storage (15-20%)

## 📑 Table des matières

- [2.1 Storage Accounts](#21-storage-accounts)
  - [Types de Storage Accounts](#types-de-storage-accounts-mise-à-jour-2024)
  - [Services de Stockage Azure](#services-de-stockage-azure---différences-clés)
  - [Réplication et Durabilité](#réplication-et-durabilité)
  - [Changement de Type de Réplication](#changement-de-type-de-réplication-upgradedowngrade)
  - [Sécurité Réseau et Firewall](#storage-account-firewall-et-sécurité-réseau)
- [2.2 Blob Storage](#22-blob-storage)
  - [Types de Blobs](#types-de-blobs)
  - [Blob Access Tiers](#blob-access-tiers---optimisation-des-coûts)
  - [Lifecycle Management](#lifecycle-management---automatisation-des-transitions)
- [2.3 Azure Files](#23-azure-files-mise-à-jour-2024)
  - [Protocoles Supportés](#protocoles-supportés)
  - [Types de File Shares](#types-de-file-shares-mise-à-jour-2024)
  - [Capacités et Limites](#capacités-et-limites-mise-à-jour-2024)
- [2.4 Azure Data Lake Storage Gen2](#24-azure-data-lake-storage-gen2)
  - [Hierarchical Namespace](#hierarchical-namespace---concept-clé)
  - [Sécurité et Contrôle d'Accès](#sécurité-et-contrôle-daccès)
  - [Organisation des Données](#organisation-des-données---best-practices)
  - [Performance et Optimisation](#performance-et-optimisation)
- [2.5 Data Transfer Solutions](#25-data-transfer-solutions-mise-à-jour-2024)
  - [Azure Import/Export](#azure-importexport-service)
  - [Outils de Transfert](#outils-de-transfert-mise-à-jour-2024)
  - [Rôles et Permissions](#storage-account-roles-et-permissions-mise-à-jour-2024)
  - [Sécurité et Conformité](#sécurité-et-conformité-nouveautés-2024)

---

## 2.1 Storage Accounts

### Types de Storage Accounts (Mise à jour 2024)

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

**⚠️ Erreur fréquente identifiée :** Confusion entre types de Storage Accounts pour Azure Files

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

### Services de Stockage Azure - Différences Clés

**Comparaison des 4 services de stockage principaux :**

**1. Blob Storage (Binary Large Objects)**
- **Usage** : Stockage de fichiers non structurés (documents, images, vidéos, backups)
- **Types de blobs** : Block, Page, Append
- **Accès** : REST API, SDK, Azure Storage Explorer
- **Cas d'usage** : Sites web statiques, archives, médias, sauvegardes
- **Niveaux** : Hot, Cool, Cold, Archive (optimisation des coûts)

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

### Réplication et Durabilité

**⚠️ Points d'attention identifiés :**

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

### Changement de Type de Réplication (Upgrade/Downgrade)

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

### Storage Account Firewall et Sécurité Réseau

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

---

## 2.2 Blob Storage

### Types de Blobs

**⚠️ Comprendre les 3 types de blobs Azure - Points critiques pour l'examen :**
                                         
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
- **Taille maximale** : 8 TiB par page blob
- **Performance** : Optimisé pour opérations de lecture/écriture aléatoires fréquentes
- **Cas d'usage typiques** :
  - Disques OS et disques de données des VMs
  - Bases de données nécessitant accès aléatoire
  - Applications nécessitant des performances I/O élevées
- **⚠️ Point clé** : Seul type de blob supportant les disques de VMs

**3. Append Blobs**
- **Usage principal** : Données ajoutées séquentiellement (logs, audit trails)
- **Structure** : Optimisé pour opérations d'ajout uniquement
- **Taille maximale** : 195 GiB par append blob
- **Limitation** : Pas de modification des données existantes, ajout uniquement
- **Cas d'usage typiques** :
  - Fichiers de logs d'applications
  - Journaux d'audit et de sécurité
  - Streaming de données en temps réel
  - Données IoT collectées en continu

**Matrice de décision rapide :**

| Besoin | Type de Blob | Raison |
|--------|--------------|--------|
| Stocker des images/vidéos | **Block Blob** | Accès aléatoire, streaming optimisé |
| Disque de VM | **Page Blob** | Seul type supporté pour VHD |
| Logs d'application | **Append Blob** | Ajout séquentiel optimisé |
| Site web statique | **Block Blob** | Hébergement web, CDN |
| Base de données | **Page Blob** | Accès aléatoire haute performance |
| Données IoT | **Append Blob** | Collecte continue, ajout uniquement |

**⚠️ Erreurs fréquentes identifiées :**
- **Erreur** : Utiliser Append Blobs pour des fichiers modifiables
- **Correct** : Block Blobs pour fichiers modifiables, Append Blobs pour ajout uniquement
- **Erreur** : Essayer d'utiliser Block Blobs pour disques de VMs
- **Correct** : Page Blobs obligatoires pour tous les disques de VMs

### Blob Access Tiers - Optimisation des Coûts

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

### Lifecycle Management - Automatisation des Transitions

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

---

## 2.3 Azure Files (Mise à jour 2024)

### Protocoles Supportés

- **SMB 3.0/3.1** : Windows, Linux, macOS
- **NFS 4.1** : Linux, Premium uniquement
- **REST API** : Accès programmatique
- **Nouveauté 2024** : Chiffrement en transit pour NFS 4.1

**⚠️ Point clé identifié :** Port SMB
- **Port 445 TCP** obligatoire pour accès SMB
- Doit être ouvert sur les firewalls clients
- Nécessaire pour mapper des lecteurs réseau

### Types de File Shares (Mise à jour 2024)

**Standard File Shares**
- **Comptes** : General Purpose v2 (GPv2)
- **Performance** : Standard (HDD)
- **Capacité** : Jusqu'à 5 TiB par share
- **Usage** : Applications générales, partages basiques

**Premium File Shares (SSD)**
- **Comptes** : General Purpose v2 (GPv2) ou FileStorage
- **Performance** : Premium (SSD)
- **Capacité** : Jusqu'à 100 TiB par share (Standard) ou 256 TiB (v2 approvisionné)
- **Usage** : Applications haute performance, bases de données
- **Nouveauté 2024** : Modèle v2 approvisionné avec prévisibilité des coûts

### Nouveautés 2024 - Fonctionnalités Avancées

- **Chiffrement en transit NFS** : Sécurité renforcée pour partages NFS 4.1
- **Mise en cache des métadonnées** : Réduction de latence, augmentation IOPS
- **Identités managées** : Authentification sécurisée sans clés partagées
- **Sauvegarde archivée** : Protection contre ransomwares, rétention jusqu'à 10 ans
- **Azure File Sync via Azure Arc** : Gestion simplifiée des agents de synchronisation

### Capacités et Limites (Mise à jour 2024)

- **Standard** : Maximum 5 TB par share
- **Premium** : Maximum 100 TB par share (Standard) ou 256 TiB (v2 approvisionné)
- **Azure Import/Export** : Support Blob Storage et Azure Files
- **Nouveauté** : Support des identités managées pour Azure File Sync

---

## 2.4 Azure Data Lake Storage Gen2

### Vue d'ensemble et Concepts Fondamentaux

**Azure Data Lake Storage Gen2** est une solution de stockage optimisée pour l'analyse de données massives (Big Data)

**Caractéristiques principales :**
- **Basé sur Blob Storage** : Construit sur Azure Blob Storage avec fonctionnalités additionnelles
- **Hierarchical Namespace (HNS)** : Organisation hiérarchique des fichiers et répertoires
- **Haute performance** : Optimisé pour analytics et traitement parallèle
- **Compatibilité Hadoop** : Support natif des systèmes de fichiers distribués
- **Sécurité granulaire** : ACLs POSIX au niveau fichier/répertoire

### Hierarchical Namespace - Concept Clé

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

### Avantages de Hierarchical Namespace

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

### Différences Clés : Blob Storage vs Data Lake Storage Gen2

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

### Activation de Hierarchical Namespace

**Processus de création :**
1. **Créer un compte de stockage** : StorageV2 (General Purpose v2)
2. **Advanced settings** : Activer "Hierarchical namespace"
3. **Validation** : Compte devient Data Lake Storage Gen2
4. **Impact** : Activation irréversible

**⚠️ Point d'attention critique identifié :**
- **Irréversibilité** : HNS ne peut pas être désactivé après activation
- **Migration** : Migrer les données existantes vers nouveau compte si besoin
- **Planification** : Décider en amont si HNS est nécessaire

### Sécurité et Contrôle d'Accès

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

### Méthodes d'Authentification

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

### Rôles RBAC pour Data Lake Storage Gen2

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

**⚠️ Stratégie de sécurité identifiée :**
1. **RBAC au niveau compte** : Contrôle d'accès global
2. **ACLs au niveau fichier/répertoire** : Contrôle d'accès granulaire
3. **Combinaison** : RBAC + ACLs pour sécurité maximale
4. **Principe** : Le plus restrictif entre RBAC et ACLs s'applique

### Intégration avec Services Analytics Azure

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

### Protocoles et APIs

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

### Cas d'Usage et Scénarios

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

### Organisation des Données - Best Practices

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

**⚠️ Points clés identifiés :**
- **Séparation des couches** : Isolation des étapes de transformation
- **Gouvernance** : ACLs différentes par couche
- **Performance** : Optimisation par use case
- **Coûts** : Lifecycle policies par couche

### Performance et Optimisation

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

### Lifecycle Management et Coûts

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

### Erreurs Fréquentes et Pièges

**⚠️ Erreur 1 : HNS non activé pour Big Data**
- **Symptôme** : Performances dégradées pour analytics
- **Cause** : Compte Blob Storage standard utilisé
- **Solution** : Créer nouveau compte avec HNS activé
- **Prévention** : Toujours activer HNS pour workloads analytics

**⚠️ Erreur 2 : Confusion RBAC et ACLs**
- **Symptôme** : Utilisateurs ne peuvent pas accéder malgré RBAC
- **Cause** : ACLs POSIX bloquent l'accès
- **Solution** : Vérifier les deux niveaux (RBAC + ACLs)
- **Règle** : Le plus restrictif s'applique

**⚠️ Erreur 3 : Millions de petits fichiers**
- **Symptôme** : Requêtes extrêmement lentes
- **Cause** : Small files problem (fichiers < 1 MB)
- **Solution** : Compaction avec Delta Lake ou ADF
- **Prévention** : Configurer batch size d'ingestion (128 MB+)

**⚠️ Erreur 4 : Mauvais partitionnement**
- **Symptôme** : Scans complets malgré filtres
- **Cause** : Partitionnement non aligné avec requêtes
- **Solution** : Re-partitionner selon colonnes filtrées
- **Exemple** : Partitionner par date si filtres par date

**⚠️ Erreur 5 : Format JSON/CSV en production**
- **Symptôme** : Coûts élevés, performances faibles
- **Cause** : Formats non optimisés pour analytics
- **Solution** : Convertir en Parquet
- **Gain** : 5-10x compression, 10-100x performance

### Monitoring et Diagnostics

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

### Comparaison avec Autres Solutions

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

### Points Critiques pour l'Examen AZ-104

✅ **Hierarchical Namespace** : Irréversible, requis pour analytics
✅ **ACLs POSIX** : Permissions granulaires fichier/répertoire
✅ **RBAC + ACLs** : Deux couches de sécurité, plus restrictif s'applique
✅ **ABFS protocol** : Protocole optimisé pour Hadoop/Spark
✅ **Storage Blob Data Owner** : Seul rôle permettant modification ACLs
✅ **Partitionnement** : Clé pour performance analytics
✅ **Parquet format** : Format recommandé pour Big Data
✅ **Lifecycle policies** : Optimisation automatique des coûts
✅ **Integration** : Synapse, Databricks, HDInsight, Data Factory

---

## 2.5 Data Transfer Solutions (Mise à jour 2024)

### Azure Import/Export Service

**⚠️ Destinations supportées identifiées :**
- **Azure Blob Storage**
- **Azure Files** (max 5 TB)
- SQL Database, autres services

**Process :**
1. Préparer les disques (BitLocker pour Windows)
2. Créer le job Import/Export
3. Expédier vers datacenter Azure
4. Azure transfert les données

### Outils de Transfert (Mise à jour 2024)

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

### Storage Account Roles et Permissions (Mise à jour 2024)

**⚠️ Rôles de gestion des comptes de stockage :**

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

**⚠️ Différenciation clé identifiée :**
- **Storage Account Contributor** : Gestion du compte + accès aux clés
- **Storage Blob Data Contributor** : Accès aux données uniquement
- **Reader** : Visualisation sans modification
- **Owner** : Contrôle total + délégation d'accès

### Sécurité et Conformité (Nouveautés 2024)

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
