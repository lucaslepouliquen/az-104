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
  - [Soft Delete](#soft-delete)
  - [Object Replication](#object-replication)
  - [Static Website Hosting](#static-website-hosting)
- [2.3 Azure Files](#23-azure-files-mise-à-jour-2024)
  - [Protocoles Supportés](#protocoles-supportés)
  - [Types de File Shares](#types-de-file-shares-mise-à-jour-2024)
  - [Capacités et Limites](#capacités-et-limites-mise-à-jour-2024)
  - [Méthodes d'Authentification](#méthodes-dauthentification-pour-azure-storage)
  - [Azure File Sync](#azure-file-sync)
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
- [2.6 Shared Access Signatures (SAS)](#26-shared-access-signatures-sas)
  - [Types de SAS](#types-de-sas)
  - [SAS Tokens et Permissions](#sas-tokens-et-permissions)
  - [Stored Access Policies](#stored-access-policies)
  - [Best Practices et Sécurité](#best-practices-et-sécurité)

---

## 2.1 Storage Accounts

### Types de Storage Accounts (Mise à jour 2024)

**General Purpose v2 (GPv2) - Standard**
- **Services** : Blobs, Files (Standard uniquement), Queues, Tables
- **Performance** : Standard (HDD)
- **Réplication** : Toutes les options (LRS, ZRS, GRS, GZRS, RA-GRS, RA-GZRS)
- **Usage** : Polyvalent, recommandé pour la plupart des cas
- **⚠️ Important** : GPv2 supporte uniquement les Standard File Shares (HDD). Pour Premium File Shares, utilisez un compte FileStorage.

**Premium Block Blobs (BlockBlobStorage)**
- **Services** : Block Blobs uniquement
- **Performance** : Premium (SSD) uniquement
- **Réplication** : LRS, ZRS uniquement
- **Usage** : Applications haute performance, bases de données, workloads IoT
- **Avantage** : IOPS élevées, latence faible (<10ms)
- **⚠️ Note** : Ne pas confondre avec "BlobStorage" (ancien type de compte deprecated)

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
| **Premium_LRS** | GRS, ZRS | ❌ Non | Conversion directe non supportée - Migration manuelle requise vers nouveau compte |

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

**⚠️ Note importante sur les prix** : Les prix indiqués ci-dessous sont approximatifs et basés sur la région US East. Les tarifs varient selon les régions Azure et sont sujets à changement. Consultez toujours la [page officielle de tarification Azure](https://azure.microsoft.com/pricing/details/storage/blobs/) pour les prix actuels.

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

**⚠️ Note** : Prix indicatifs pour US East. Archive moins avantageux si accès fréquent. Consultez la tarification officielle pour votre région.

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

### Soft Delete

**⚠️ Concept Clé pour AZ-104 : Soft Delete protège vos données contre les suppressions accidentelles en conservant les objets supprimés pendant une période de rétention configurable**

**Définition :**
- **Soft Delete** : Fonctionnalité de protection qui conserve les données supprimées pendant une période définie (7-365 jours)
- **Objectif** : Récupération de données supprimées accidentellement ou malicieusement
- **Scope** : Disponible pour Blobs, Containers, et File Shares

**Types de Soft Delete :**

**1. Soft Delete pour Blobs**

**Caractéristiques :**
- **Période de rétention** : 7 à 365 jours (configurable)
- **Protection** : Blobs supprimés et versions écrasées
- **Coût** : Stockage des blobs soft-deleted facturé au même tier
- **Compatibilité** : Fonctionne avec blob versioning

**Activation - Blob Soft Delete :**

```bash
# Via Azure CLI
az storage account blob-service-properties update \
  --account-name mystorageaccount \
  --resource-group myResourceGroup \
  --enable-delete-retention true \
  --delete-retention-days 30

# Vérifier la configuration
az storage account blob-service-properties show \
  --account-name mystorageaccount \
  --resource-group myResourceGroup \
  --query deleteRetentionPolicy
```

**Via PowerShell :**

```powershell
# Activer soft delete pour blobs
Enable-AzStorageBlobDeleteRetentionPolicy `
  -ResourceGroupName "myResourceGroup" `
  -StorageAccountName "mystorageaccount" `
  -RetentionDays 30

# Vérifier
Get-AzStorageBlobServiceProperty `
  -ResourceGroupName "myResourceGroup" `
  -StorageAccountName "mystorageaccount"
```

**Récupération de Blobs Soft-Deleted :**

```bash
# Via Azure CLI - Lister les blobs supprimés
az storage blob list \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --include d \
  --output table

# Restaurer un blob supprimé (undelete)
az storage blob undelete \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myblob.txt
```

**Via PowerShell :**

```powershell
# Obtenir le contexte
$ctx = New-AzStorageContext -StorageAccountName "mystorageaccount" -StorageAccountKey "xxxx"

# Lister blobs supprimés
Get-AzStorageBlob -Container "mycontainer" -Context $ctx -IncludeDeleted

# Restaurer un blob
$blob = Get-AzStorageBlob -Container "mycontainer" -Blob "myblob.txt" -Context $ctx -IncludeDeleted
$blob.ICloudBlob.Undelete()
```

**Via Azure Portal :**
```
Storage Account → Containers → Select Container → Show deleted blobs → Select blob → Undelete
```

**2. Soft Delete pour Containers**

**Caractéristiques :**
- **Période de rétention** : 1 à 365 jours
- **Protection** : Containers entiers supprimés
- **Contenu** : Tous les blobs du container sont restaurés
- **Nouveauté 2024** : Disponible en GA (Generally Available)

**Activation - Container Soft Delete :**

```bash
# Via Azure CLI
az storage account blob-service-properties update \
  --account-name mystorageaccount \
  --resource-group myResourceGroup \
  --enable-container-delete-retention true \
  --container-delete-retention-days 30

# Lister containers supprimés
az storage container list \
  --account-name mystorageaccount \
  --include-deleted \
  --output table
```

**Via PowerShell :**

```powershell
# Activer soft delete pour containers
Enable-AzStorageContainerDeleteRetentionPolicy `
  -ResourceGroupName "myResourceGroup" `
  -StorageAccountName "mystorageaccount" `
  -RetentionDays 30

# Lister containers supprimés
Get-AzStorageContainer -Context $ctx -IncludeDeleted

# Restaurer un container
Restore-AzStorageContainer `
  -Name "mycontainer" `
  -Context $ctx
```

**3. Soft Delete pour File Shares**

**Caractéristiques :**
- **Période de rétention** : 1 à 365 jours
- **Protection** : File shares entiers supprimés
- **Limitation** : Fonctionne uniquement avec Standard File Shares (pas Premium)
- **Quota** : Le quota du share supprimé compte dans la limite du compte

**Activation - File Share Soft Delete :**

```bash
# Via Azure CLI
az storage account file-service-properties update \
  --account-name mystorageaccount \
  --resource-group myResourceGroup \
  --enable-delete-retention true \
  --delete-retention-days 14

# Lister file shares supprimés
az storage share list \
  --account-name mystorageaccount \
  --include-deleted \
  --output table
```

**Via PowerShell :**

```powershell
# Activer soft delete pour file shares
Update-AzStorageFileServiceProperty `
  -ResourceGroupName "myResourceGroup" `
  -StorageAccountName "mystorageaccount" `
  -EnableShareDeleteRetentionPolicy $true `
  -ShareRetentionDays 14

# Restaurer un file share
Restore-AzStorageShare `
  -Name "myfileshare" `
  -Context $ctx
```

**⚠️ Comparaison des Soft Delete :**

| Type | Période Min/Max | Coût | Restauration | Limitation |
|------|----------------|------|--------------|------------|
| **Blob Soft Delete** | 7-365 jours | ✅ Tier d'origine | Undelete | Aucune |
| **Container Soft Delete** | 1-365 jours | ✅ Tier d'origine | Restore | Nom unique requis |
| **File Share Soft Delete** | 1-365 jours | ✅ Quota compte | Restore | Standard uniquement |

**⚠️ Scénarios d'Utilisation - Pour l'Examen :**

**Scénario 1 : Protection contre Ransomware**

```
Problème :
- Ransomware supprime des blobs
- Besoin de récupération rapide
- Minimiser la perte de données

Solution :
1. Activer Blob Soft Delete (30 jours minimum)
2. Activer Blob Versioning (historique complet)
3. Activer Container Soft Delete (30 jours)
4. En cas d'attaque : Restaurer depuis soft delete

# Configuration complète
az storage account blob-service-properties update \
  --account-name mystorageaccount \
  --enable-delete-retention true \
  --delete-retention-days 30 \
  --enable-versioning true \
  --enable-container-delete-retention true \
  --container-delete-retention-days 30

Avantages :
✅ Protection multi-niveaux
✅ Récupération rapide (minutes)
✅ Pas besoin de backup restore
✅ Historique complet avec versioning
```

**Scénario 2 : Suppression Accidentelle par Utilisateur**

```
Problème :
- Utilisateur supprime un container par erreur
- 500 GB de données perdues
- Besoin de restauration immédiate

Solution :
1. Vérifier que Container Soft Delete est activé
2. Lister containers supprimés
3. Restaurer le container

# Commandes
az storage container list --account-name mystorageaccount --include-deleted
az storage container restore --name mycontainer --account-name mystorageaccount

Temps de récupération : < 5 minutes
```

**⚠️ Best Practices Soft Delete :**

**1. Période de Rétention**
- ✅ **Minimum 30 jours** pour blobs et containers
- ✅ **14 jours** pour file shares (suffisant pour la plupart des cas)
- ✅ Aligner avec les politiques de backup

**2. Coûts**
- ⚠️ Soft-deleted blobs **comptent dans le stockage**
- ✅ Monitorer l'espace utilisé par soft delete
- ✅ Ajuster période de rétention selon budget

```bash
# Vérifier l'espace utilisé par soft delete
az storage blob list \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --include d \
  --query "[?properties.deletedTime!=null].{Name:name, Size:properties.contentLength}" \
  --output table
```

**3. Combinaison avec Autres Protections**
- ✅ **Soft Delete + Versioning** : Protection maximale
- ✅ **Soft Delete + Immutable Storage** : Compliance
- ✅ **Soft Delete + Backup** : Stratégie 3-2-1

**⚠️ Erreurs Courantes QCM :**

| Question | Réponse Incorrecte ❌ | Réponse Correcte ✅ |
|----------|----------------------|---------------------|
| **"Soft Delete protège contre quoi ?"** | "Corruption de données" | "Suppression accidentelle ou malicieuse" |
| **"Soft Delete est gratuit ?"** | "Oui, pas de coût supplémentaire" | "Non, stockage facturé au tier d'origine" |
| **"Période de rétention max pour blobs ?"** | "90 jours" | "365 jours (1 an)" |
| **"File Share Soft Delete fonctionne sur Premium ?"** | "Oui, tous les tiers" | "Non, Standard uniquement" |
| **"Peut-on restaurer après expiration rétention ?"** | "Oui, via support Azure" | "Non, données définitivement supprimées" |

**⚠️ Points Clés pour l'Examen :**
- ✅ **3 types de Soft Delete** : Blobs (7-365j), Containers (1-365j), File Shares (1-365j)
- ✅ **Coût** : Stockage facturé pendant période de rétention
- ✅ **Restauration** : Undelete (blobs) ou Restore (containers/shares)
- ✅ **File Share Soft Delete** : Standard uniquement, pas Premium
- ✅ **Combinaison** : Soft Delete + Versioning = Protection maximale
- ✅ **Ransomware** : Soft Delete est une défense efficace
- ✅ **Portal** : Peut restaurer via UI (Show deleted blobs/containers)

### Object Replication

**⚠️ Concept Clé pour AZ-104 : Object Replication copie automatiquement les blobs entre storage accounts pour réduire la latence et améliorer la disponibilité**

**Définition :**
- **Object Replication** : Réplication asynchrone de block blobs entre storage accounts
- **Objectif** : Disaster Recovery, réduction de latence, conformité géographique
- **Direction** : Unidirectionnelle (source → destination)

**Architecture Object Replication :**

```
┌─────────────────────────────────────────────────────┐
│         Source Storage Account (Region A)           │
│  ┌──────────────────────────────────────────┐      │
│  │     Source Container: "prod-data"         │      │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  │      │
│  │  │ Blob 1  │  │ Blob 2  │  │ Blob 3  │  │      │
│  │  └─────────┘  └─────────┘  └─────────┘  │      │
│  └──────────────────┬───────────────────────┘      │
└─────────────────────┼─────────────────────────────┘
                      │ Replication Rule
                      │ (Asynchronous)
                      ▼
┌─────────────────────────────────────────────────────┐
│      Destination Storage Account (Region B)         │
│  ┌──────────────────────────────────────────┐      │
│  │  Destination Container: "prod-data-dr"    │      │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────┐  │      │
│  │  │ Blob 1  │  │ Blob 2  │  │ Blob 3  │  │      │
│  │  └─────────┘  └─────────┘  └─────────┘  │      │
│  └──────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────┘
```

**Prérequis Object Replication :**

**1. Prérequis Techniques :**
- ✅ **Blob Versioning** activé sur source ET destination
- ✅ **Change Feed** activé sur source
- ✅ **Block Blobs** uniquement (pas Page ou Append blobs)
- ✅ **GPv2 ou Premium Block Blob** accounts
- ✅ **Même région ou régions différentes** (cross-region supporté)

**2. Activation des Prérequis :**

```bash
# Sur le compte SOURCE
az storage account blob-service-properties update \
  --account-name sourcestorageaccount \
  --resource-group myResourceGroup \
  --enable-versioning true \
  --enable-change-feed true

# Sur le compte DESTINATION
az storage account blob-service-properties update \
  --account-name deststorageaccount \
  --resource-group myResourceGroup \
  --enable-versioning true
```

**Configuration Object Replication :**

**Méthode 1 : Via Azure Portal (Recommandé)**

```
1. Storage Account Source → Object replication → Set up replication rules
2. Sélectionner Destination storage account
3. Créer règle de réplication :
   - Source container : prod-data
   - Destination container : prod-data-dr
   - Filtres (optionnel) : Prefixes, tags
4. Save
```

**Méthode 2 : Via Azure CLI**

```bash
# Étape 1 : Créer la policy sur destination (génère policy ID)
az storage account or-policy create \
  --account-name deststorageaccount \
  --resource-group myResourceGroup \
  --source-account sourcestorageaccount \
  --destination-account deststorageaccount \
  --source-container prod-data \
  --destination-container prod-data-dr \
  --policy-id "policy1"

# Étape 2 : Appliquer la policy sur source
az storage account or-policy show \
  --account-name deststorageaccount \
  --policy-id "policy1" \
  --resource-group myResourceGroup

# Copier la policy et l'appliquer sur source
az storage account or-policy create \
  --account-name sourcestorageaccount \
  --resource-group myResourceGroup \
  --policy @policy.json
```

**Via PowerShell :**

```powershell
# Obtenir les contextes
$srcCtx = New-AzStorageContext -StorageAccountName "sourcestorageaccount" -UseConnectedAccount
$destCtx = New-AzStorageContext -StorageAccountName "deststorageaccount" -UseConnectedAccount

# Créer règle de réplication
$rule = New-AzStorageObjectReplicationPolicyRule `
  -SourceContainer "prod-data" `
  -DestinationContainer "prod-data-dr" `
  -PrefixMatch "important/"

# Créer policy
New-AzStorageObjectReplicationPolicy `
  -ResourceGroupName "myResourceGroup" `
  -StorageAccountName "sourcestorageaccount" `
  -PolicyId "policy1" `
  -Rule $rule
```

**Règles de Réplication - Filtres :**

**1. Prefix Match (Filtrer par chemin)**

```json
{
  "rules": [
    {
      "ruleId": "rule1",
      "sourceContainer": "prod-data",
      "destinationContainer": "prod-data-dr",
      "filters": {
        "prefixMatch": [
          "important/",
          "critical/"
        ]
      }
    }
  ]
}
```

**2. Minimum Creation Time (Filtrer par date)**

```json
{
  "rules": [
    {
      "ruleId": "rule2",
      "sourceContainer": "prod-data",
      "destinationContainer": "prod-data-dr",
      "filters": {
        "minCreationTime": "2024-01-01T00:00:00Z"
      }
    }
  ]
}
```

**⚠️ Comportement de la Réplication :**

**Ce qui est répliqué :**
- ✅ **Nouveaux blobs** créés après configuration
- ✅ **Modifications** de blobs existants (nouvelles versions)
- ✅ **Métadonnées** et tags
- ✅ **Blob tiers** (Hot, Cool, Archive)

**Ce qui N'EST PAS répliqué :**
- ❌ **Blobs existants** avant configuration (sauf si re-uploadés)
- ❌ **Suppressions** (soft delete ou hard delete)
- ❌ **Snapshots** (uniquement versions)
- ❌ **Immutable blobs** (WORM)

**Réplication de Blobs Existants :**

```bash
# Pour répliquer blobs existants, utiliser AzCopy
azcopy copy \
  "https://sourcestorageaccount.blob.core.windows.net/prod-data/*?<SAS>" \
  "https://deststorageaccount.blob.core.windows.net/prod-data-dr/?<SAS>" \
  --recursive

# Puis activer object replication pour changements futurs
```

**⚠️ Use Cases Object Replication :**

**Use Case 1 : Disaster Recovery (DR)**

```
Problème :
- Région primaire (West Europe) indisponible
- Besoin de failover vers région secondaire (North Europe)
- RTO < 1 heure

Solution :
1. Object Replication : West Europe → North Europe
2. En cas de panne :
   - Pointer applications vers North Europe storage
   - Données récentes disponibles (latence réplication ~minutes)
3. Après restauration : Inverser réplication si nécessaire

Configuration :
Source : westeuropeaccount/prod-data
Destination : northeuropeaccount/prod-data-dr
Filters : Tous les blobs
```

**Use Case 2 : Réduction de Latence Géographique**

```
Problème :
- Utilisateurs en Asie accèdent à storage en Europe
- Latence élevée (200-300ms)
- Besoin de performance locale

Solution :
1. Object Replication : Europe → Asie
2. Applications en Asie lisent depuis storage local
3. Écritures toujours vers Europe (source)
4. Réplication automatique vers Asie

Configuration :
Source : europestorageaccount/media
Destination : asiastorageaccount/media-replica
Filters : prefixMatch: ["images/", "videos/"]

Résultat : Latence réduite de 250ms → 20ms
```

**Use Case 3 : Conformité Multi-Région**

```
Problème :
- Réglementation exige données en Europe ET US
- Données doivent être identiques
- Audit trail requis

Solution :
1. Object Replication bidirectionnelle (2 policies)
   - Policy 1 : Europe → US
   - Policy 2 : US → Europe
2. Versioning activé (audit trail)
3. Monitoring de replication status

⚠️ Attention : Bidirectionnel peut causer conflits
Recommandation : Unidirectionnel avec source unique
```

**Monitoring Object Replication :**

**Vérifier le Statut de Réplication :**

```bash
# Via Azure CLI - Vérifier policy
az storage account or-policy show \
  --account-name sourcestorageaccount \
  --policy-id "policy1" \
  --resource-group myResourceGroup

# Vérifier statut d'un blob spécifique
az storage blob show \
  --account-name sourcestorageaccount \
  --container-name prod-data \
  --name myblob.txt \
  --query "objectReplicationSourceProperties"
```

**Via PowerShell :**

```powershell
# Obtenir statut de réplication d'un blob
$blob = Get-AzStorageBlob -Container "prod-data" -Blob "myblob.txt" -Context $srcCtx
$blob.BlobProperties.ObjectReplicationSourceProperties

# Statuts possibles :
# - Completed : Réplication réussie
# - Failed : Échec de réplication
# - InProgress : En cours
```

**Métriques Azure Monitor :**

```
Storage Account → Monitoring → Metrics

Métriques clés :
- Object Replication Latency : Temps de réplication
- Object Replication Bytes : Volume répliqué
- Object Replication Count : Nombre d'objets répliqués
```

**⚠️ Limites Object Replication :**

| Limite | Valeur | Notes |
|--------|--------|-------|
| **Policies par storage account** | 1000 | Source + Destination |
| **Rules par policy** | 1000 | Filtres multiples possibles |
| **Source containers par rule** | 1 | Un container source par règle |
| **Destination containers par rule** | 1 | Un container destination par règle |
| **Latence de réplication** | Minutes à heures | Dépend de la taille et région |

**⚠️ Coûts Object Replication :**

```
Composants de Coût :
1. Stockage destination (même prix que source)
2. Transactions de réplication (PUT operations)
3. Egress data (sortie de région source)
4. Change Feed (inclus, pas de coût supplémentaire)
5. Versioning (stockage des versions)

Optimisation :
- Utiliser filtres (prefixMatch) pour limiter scope
- Choisir régions avec coûts egress faibles
- Monitorer volume répliqué
```

**⚠️ Erreurs Courantes QCM :**

| Question | Réponse Incorrecte ❌ | Réponse Correcte ✅ |
|----------|----------------------|---------------------|
| **"Object Replication est synchrone ?"** | "Oui, temps réel" | "Non, asynchrone (minutes)" |
| **"Peut répliquer blobs existants automatiquement ?"** | "Oui, tous les blobs" | "Non, uniquement nouveaux blobs après config" |
| **"Réplication bidirectionnelle automatique ?"** | "Oui, en activant option" | "Non, nécessite 2 policies séparées" |
| **"Suppression répliquée ?"** | "Oui, automatiquement" | "Non, suppressions ne sont pas répliquées" |
| **"Prérequis pour Object Replication ?"** | "Aucun" | "Versioning + Change Feed sur source" |

**⚠️ Points Clés pour l'Examen :**
- ✅ **Réplication asynchrone** : Latence de minutes (pas temps réel)
- ✅ **Prérequis** : Versioning (source + dest) + Change Feed (source)
- ✅ **Unidirectionnel** : Source → Destination (bidirectionnel = 2 policies)
- ✅ **Block Blobs uniquement** : Pas Page ou Append blobs
- ✅ **Nouveaux blobs** : Réplication automatique après configuration
- ✅ **Blobs existants** : Utiliser AzCopy pour migration initiale
- ✅ **Suppressions non répliquées** : Comportement par design
- ✅ **Use Cases** : DR, réduction latence, conformité multi-région
- ✅ **Filtres** : prefixMatch, minCreationTime pour scope limité

### Static Website Hosting

**⚠️ Concept Clé pour AZ-104 : Static Website Hosting permet d'héberger des sites web statiques (HTML, CSS, JS) directement depuis Azure Blob Storage à faible coût**

**Définition :**
- **Static Website Hosting** : Fonctionnalité qui sert des fichiers statiques via HTTP/HTTPS
- **Objectif** : Hébergement simple et économique de sites statiques, SPAs (Single Page Applications)
- **Container spécial** : `$web` créé automatiquement

**Architecture Static Website :**

```
┌─────────────────────────────────────────────────────┐
│         Storage Account (GPv2 ou BlockBlobStorage)   │
│  ┌──────────────────────────────────────────┐      │
│  │     Container: $web (automatique)         │      │
│  │  ┌─────────────┐  ┌──────────────┐      │      │
│  │  │ index.html  │  │ error.html   │      │      │
│  │  └─────────────┘  └──────────────┘      │      │
│  │  ┌─────────────┐  ┌──────────────┐      │      │
│  │  │ styles.css  │  │ script.js    │      │      │
│  │  └─────────────┘  └──────────────┘      │      │
│  │  ┌─────────────────────────────┐        │      │
│  │  │ images/                      │        │      │
│  │  │   ├── logo.png               │        │      │
│  │  │   └── banner.jpg             │        │      │
│  │  └─────────────────────────────┘        │      │
│  └──────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────┘
                      ▼
         Public Endpoint (HTTPS)
https://mystorageaccount.z6.web.core.windows.net
                      ▼
              Custom Domain (optionnel)
           https://www.example.com
```

**Activation Static Website Hosting :**

**Via Azure CLI :**

```bash
# Activer static website hosting
az storage blob service-properties update \
  --account-name mystorageaccount \
  --static-website \
  --404-document error.html \
  --index-document index.html

# Vérifier la configuration
az storage blob service-properties show \
  --account-name mystorageaccount \
  --query "staticWebsite"

# Output : Primary endpoint
# https://mystorageaccount.z6.web.core.windows.net
```

**Via PowerShell :**

```powershell
# Activer static website
Enable-AzStorageStaticWebsite `
  -ResourceGroupName "myResourceGroup" `
  -StorageAccountName "mystorageaccount" `
  -IndexDocument "index.html" `
  -ErrorDocument404Path "error.html"

# Obtenir l'endpoint
$account = Get-AzStorageAccount -ResourceGroupName "myResourceGroup" -Name "mystorageaccount"
$account.PrimaryEndpoints.Web
```

**Via Azure Portal :**

```
Storage Account → Data management → Static website
├── Enable : On
├── Index document name : index.html
├── Error document path : error.html
└── Save

Primary endpoint : https://mystorageaccount.z6.web.core.windows.net
```

**Container $web - Caractéristiques :**

**Propriétés du Container $web :**
- **Création automatique** : Créé lors de l'activation de static website
- **Nom réservé** : `$web` (ne peut pas être renommé)
- **Accès public** : Configuré automatiquement en lecture anonyme
- **Structure** : Peut contenir dossiers et sous-dossiers
- **Suppression** : Désactiver static website pour supprimer

**Upload de Fichiers vers $web :**

```bash
# Via Azure CLI
az storage blob upload-batch \
  --account-name mystorageaccount \
  --source ./website \
  --destination '$web' \
  --overwrite

# Upload fichier individuel
az storage blob upload \
  --account-name mystorageaccount \
  --container-name '$web' \
  --name index.html \
  --file ./index.html \
  --content-type "text/html"
```

**Via AzCopy (Recommandé pour gros volumes) :**

```bash
# Upload récursif
azcopy copy './website/*' \
  'https://mystorageaccount.blob.core.windows.net/$web?<SAS>' \
  --recursive

# Sync (supprime fichiers non présents en local)
azcopy sync './website' \
  'https://mystorageaccount.blob.core.windows.net/$web?<SAS>' \
  --recursive --delete-destination=true
```

**Documents Index et Error :**

**1. Index Document (index.html)**
- **Rôle** : Page par défaut servie à la racine et dans les dossiers
- **Comportement** : 
  - `https://mystorageaccount.z6.web.core.windows.net/` → Sert `index.html`
  - `https://mystorageaccount.z6.web.core.windows.net/blog/` → Sert `blog/index.html`

**Exemple index.html :**

```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>Mon Site Static</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <h1>Bienvenue sur mon site Azure Static</h1>
    <p>Hébergé sur Azure Blob Storage</p>
    <script src="script.js"></script>
</body>
</html>
```

**2. Error Document (error.html)**
- **Rôle** : Page servie pour erreurs 404 (Not Found)
- **Comportement** : Toute URL non trouvée redirige vers error.html
- **HTTP Status** : 404 (pas 200)

**Exemple error.html :**

```html
<!DOCTYPE html>
<html lang="fr">
<head>
    <meta charset="UTF-8">
    <title>Page Non Trouvée - 404</title>
</head>
<body>
    <h1>404 - Page Non Trouvée</h1>
    <p>La page que vous recherchez n'existe pas.</p>
    <a href="/">Retour à l'accueil</a>
</body>
</html>
```

**Custom Domains avec Azure CDN :**

**Pourquoi utiliser un Custom Domain ?**
- ✅ **Branding** : www.example.com au lieu de mystorageaccount.z6.web.core.windows.net
- ✅ **HTTPS personnalisé** : Certificat SSL pour domaine custom
- ✅ **Performance** : CDN cache le contenu globalement
- ✅ **Compression** : Gzip automatique via CDN

**Configuration Custom Domain avec Azure CDN :**

**Étape 1 : Créer Azure CDN Profile et Endpoint**

```bash
# Créer CDN Profile
az cdn profile create \
  --resource-group myResourceGroup \
  --name myCDNProfile \
  --sku Standard_Microsoft

# Créer CDN Endpoint
az cdn endpoint create \
  --resource-group myResourceGroup \
  --profile-name myCDNProfile \
  --name mywebsite \
  --origin mystorageaccount.z6.web.core.windows.net \
  --origin-host-header mystorageaccount.z6.web.core.windows.net \
  --enable-compression true \
  --content-types-to-compress "text/html" "text/css" "application/javascript"

# Output : Endpoint URL
# https://mywebsite.azureedge.net
```

**Étape 2 : Configurer DNS pour Custom Domain**

```
Chez votre registrar DNS (GoDaddy, Cloudflare, etc.) :

Type : CNAME
Name : www
Value : mywebsite.azureedge.net
TTL : 3600
```

**Étape 3 : Ajouter Custom Domain au CDN Endpoint**

```bash
# Ajouter custom domain
az cdn custom-domain create \
  --resource-group myResourceGroup \
  --profile-name myCDNProfile \
  --endpoint-name mywebsite \
  --name www-example-com \
  --hostname www.example.com

# Activer HTTPS (certificat gratuit via CDN)
az cdn custom-domain enable-https \
  --resource-group myResourceGroup \
  --profile-name myCDNProfile \
  --endpoint-name mywebsite \
  --name www-example-com \
  --min-tls-version "1.2"
```

**Étape 4 : Vérification**

```bash
# Tester l'accès
curl -I https://www.example.com

# Vérifier cache CDN
curl -I https://www.example.com | grep "X-Cache"
# X-Cache: HIT (contenu servi depuis CDN)
# X-Cache: MISS (contenu récupéré depuis origin)
```

**⚠️ Configuration CDN Avancée :**

**1. Compression**

```bash
# Activer compression Gzip
az cdn endpoint update \
  --resource-group myResourceGroup \
  --profile-name myCDNProfile \
  --name mywebsite \
  --enable-compression true \
  --content-types-to-compress \
    "text/html" \
    "text/css" \
    "text/javascript" \
    "application/javascript" \
    "application/json" \
    "image/svg+xml"
```

**2. Caching Rules**

```bash
# Configurer cache duration
az cdn endpoint rule add \
  --resource-group myResourceGroup \
  --profile-name myCDNProfile \
  --endpoint-name mywebsite \
  --order 1 \
  --rule-name "CacheStaticAssets" \
  --match-variable UrlFileExtension \
  --operator Equal \
  --match-values "jpg" "png" "css" "js" \
  --action-name CacheExpiration \
  --cache-behavior Override \
  --cache-duration "7.00:00:00"
```

**3. Purge CDN Cache**

```bash
# Purger tout le cache
az cdn endpoint purge \
  --resource-group myResourceGroup \
  --profile-name myCDNProfile \
  --name mywebsite \
  --content-paths "/*"

# Purger fichiers spécifiques
az cdn endpoint purge \
  --resource-group myResourceGroup \
  --profile-name myCDNProfile \
  --name mywebsite \
  --content-paths "/index.html" "/styles.css"
```

**⚠️ Use Cases Static Website Hosting :**

**Use Case 1 : Portfolio / Site Vitrine**

```
Caractéristiques :
- HTML/CSS/JS statique
- Pas de backend
- Faible trafic (<10K visiteurs/mois)

Solution :
1. Static Website Hosting sur Storage Account
2. Pas besoin de CDN (coût minimal)
3. Custom domain via CNAME direct (sans HTTPS)

Coût mensuel : ~$0.50 (stockage + transactions)
```

**Use Case 2 : Single Page Application (SPA) - React/Vue/Angular**

```
Caractéristiques :
- Application React buildée (build/)
- API backend séparée
- Trafic global

Solution :
1. Build de l'app : npm run build
2. Upload build/ vers $web
3. Azure CDN avec custom domain
4. HTTPS gratuit via CDN
5. Compression Gzip activée

# Déploiement automatisé
npm run build
azcopy sync './build' 'https://mystorageaccount.blob.core.windows.net/$web?<SAS>' --recursive
az cdn endpoint purge --profile-name myCDNProfile --name mywebsite --content-paths "/*"

Coût mensuel : ~$5-20 (selon trafic CDN)
```

**Use Case 3 : Documentation Technique**

```
Caractéristiques :
- Site généré (MkDocs, Hugo, Jekyll)
- Nombreuses pages
- Recherche côté client

Solution :
1. Générer site statique : mkdocs build
2. Upload vers $web
3. CDN pour performance
4. Versioning des déploiements

Avantages :
✅ Pas de serveur à maintenir
✅ Scalabilité automatique
✅ Coût très faible
```

**⚠️ Limites Static Website Hosting :**

| Limitation | Impact | Workaround |
|-----------|--------|------------|
| **Pas de HTTPS natif** | Sécurité | Utiliser Azure CDN avec HTTPS |
| **Pas de redirections** | SEO | Gérer côté client (JS) ou CDN rules |
| **Pas de headers custom** | Cache control | Configurer via CDN |
| **404 pour toutes erreurs** | UX | SPA avec client-side routing |
| **Pas de server-side logic** | Fonctionnalités | Utiliser Azure Functions + API |

**⚠️ Erreurs Courantes QCM :**

| Question | Réponse Incorrecte ❌ | Réponse Correcte ✅ |
|----------|----------------------|---------------------|
| **"Static Website supporte PHP/Python ?"** | "Oui, via configuration" | "Non, fichiers statiques uniquement (HTML/CSS/JS)" |
| **"HTTPS disponible par défaut ?"** | "Oui, automatique" | "Non, nécessite Azure CDN pour HTTPS custom" |
| **"Container $web peut être renommé ?"** | "Oui, dans settings" | "Non, nom réservé et fixe" |
| **"Peut héberger API backend ?"** | "Oui, avec Node.js" | "Non, utiliser Azure Functions ou App Service" |
| **"Custom domain sans CDN possible ?"** | "Non, CDN obligatoire" | "Oui, mais sans HTTPS (HTTP uniquement)" |

**⚠️ Best Practices Static Website :**

**1. Performance**
- ✅ Utiliser Azure CDN pour distribution globale
- ✅ Activer compression Gzip
- ✅ Optimiser images (WebP, compression)
- ✅ Minifier CSS/JS

**2. Sécurité**
- ✅ HTTPS via CDN (certificat gratuit)
- ✅ Content Security Policy headers (via CDN)
- ✅ Pas de données sensibles dans fichiers statiques

**3. Déploiement**
- ✅ CI/CD avec GitHub Actions ou Azure DevOps
- ✅ Purge CDN après déploiement
- ✅ Versioning des releases

**Exemple GitHub Actions Workflow :**

```yaml
name: Deploy Static Website
on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Build
        run: npm run build
      
      - name: Deploy to Azure Storage
        uses: azure/CLI@v1
        with:
          inlineScript: |
            az storage blob upload-batch \
              --account-name mystorageaccount \
              --source ./build \
              --destination '$web' \
              --overwrite
      
      - name: Purge CDN
        run: |
          az cdn endpoint purge \
            --profile-name myCDNProfile \
            --name mywebsite \
            --content-paths "/*"
```

**⚠️ Points Clés pour l'Examen :**
- ✅ **Container $web** : Créé automatiquement, nom réservé
- ✅ **index.html** : Document par défaut (racine et dossiers)
- ✅ **error.html** : Page 404 personnalisée
- ✅ **Fichiers statiques uniquement** : HTML, CSS, JS, images
- ✅ **Pas de HTTPS natif** : Utiliser Azure CDN pour custom domain HTTPS
- ✅ **Custom domain** : CNAME vers CDN endpoint
- ✅ **Compression** : Activer Gzip via CDN
- ✅ **Use Cases** : SPAs, portfolios, documentation, sites vitrines
- ✅ **Coût** : Très faible (stockage + transactions + CDN optionnel)
- ✅ **Déploiement** : AzCopy, Azure CLI, ou CI/CD pipelines

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
- **Capacité** : 
  - Par défaut : Jusqu'à 5 TiB par share
  - Avec Large File Shares activé : Jusqu'à 100 TiB par share
- **Usage** : Applications générales, partages basiques
- **⚠️ Important** : Large File Shares doit être activé à la création du compte (irréversible)

**Premium File Shares (SSD)**
- **Comptes** : FileStorage uniquement (compte dédié)
- **Performance** : Premium (SSD)
- **Capacité** : Jusqu'à 100 TiB par share (modèle standard)
- **Usage** : Applications haute performance, bases de données
- **Nouveauté 2024** : Modèle v2 approvisionné permettant jusqu'à 256 TiB avec provisionnement flexible du stockage, IOPS et throughput

### Nouveautés 2024 - Fonctionnalités Avancées

- **Chiffrement en transit NFS** : Sécurité renforcée pour partages NFS 4.1
- **Mise en cache des métadonnées** : Réduction de latence, augmentation IOPS
- **Identités managées** : Authentification sécurisée sans clés partagées
- **Sauvegarde archivée** : Protection contre ransomwares, rétention jusqu'à 10 ans
- **Azure File Sync via Azure Arc** : Gestion simplifiée des agents de synchronisation

### Capacités et Limites (Mise à jour 2024)

- **Standard File Shares** : 
  - Par défaut : Maximum 5 TiB par share
  - Avec Large File Shares activé : Maximum 100 TiB par share
- **Premium File Shares** : 
  - Modèle standard : Maximum 100 TiB par share
  - Modèle v2 approvisionné : Jusqu'à 256 TiB par share
- **Azure Import/Export** : Support Blob Storage et Azure Files
- **Nouveauté** : Support des identités managées pour Azure File Sync

### Méthodes d'Authentification pour Azure Storage

**⚠️ Concept Clé pour AZ-104 : Chaque service de stockage Azure supporte différentes méthodes d'authentification - Comprendre quelle méthode utiliser pour quel service est ESSENTIEL pour l'examen**

**Vue d'Ensemble des Méthodes d'Authentification :**

| Méthode d'Authentification | Type | Azure Files (SMB) | Blob Storage | Queue Storage | Table Storage |
|----------------------------|------|-------------------|--------------|---------------|---------------|
| **Kerberos (Microsoft Entra)** | **IAM** | ✅ **OUI** | ❌ Non | ❌ Non | ❌ Non |
| **Azure AD (OAuth 2.0)** | **IAM** | ⚠️ Via REST API | ✅ Oui | ✅ Oui | ✅ Oui |
| **Shared Key (Account Key)** | **Credential** | ✅ Oui | ✅ Oui | ✅ Oui | ✅ Oui |
| **Shared Access Signature (SAS)** | **Credential** | ✅ Oui | ✅ Oui | ✅ Oui | ✅ Oui |
| **Anonymous Public Access** | **None** | ❌ Non | ✅ Oui | ❌ Non | ❌ Non |

**⚠️ Point Critique pour l'Examen :**

**Question Type Examen :** *"Which storage data service can be configured to use identity-based access?"*

**Réponse : Azure Files (file shares)** ✅

**Pourquoi ?** Azure Files est le SEUL service de stockage qui supporte **Microsoft Entra Kerberos** (anciennement Azure AD Kerberos), qui est une authentification basée sur l'identité (IAM).

---

### 1. Kerberos Authentication (Microsoft Entra) - IDENTITY-BASED (IAM)

**⚠️ Azure Files UNIQUEMENT - Méthode d'Authentification Basée sur l'Identité**

**Définition :**
- **Microsoft Entra Kerberos** : Protocole d'authentification IAM qui permet aux utilisateurs Azure AD d'accéder aux file shares
- **Type** : **Identity and Access Management (IAM)** - Authentification basée sur l'identité
- **Scope** : **Azure Files (SMB) uniquement** - NE fonctionne PAS pour Blobs, Queues, ou Tables
- **Avantage** : Pas besoin de clés de stockage, authentification SSO (Single Sign-On)

**Cas d'Usage :**
- Utilisateurs Azure AD accédant à des file shares
- Migration lift-and-shift depuis AD on-premises
- Scénarios hybrides avec Azure AD Connect

**Configuration Microsoft Entra Kerberos :**

**Prérequis :**
- Azure Files (Standard ou Premium)
- Azure AD tenant
- Utilisateurs/groupes Azure AD
- Optionnel : Azure AD Connect pour synchronisation hybrid

**Étape 1 : Activer l'Authentification Identity-Based**

```bash
# Via Azure CLI
az storage account update \
  --name mystorageaccount \
  --resource-group myResourceGroup \
  --enable-files-aadds false \
  --enable-files-aadkerb true

# Paramètres :
# --enable-files-aadds : Azure AD Domain Services (legacy)
# --enable-files-aadkerb : Microsoft Entra Kerberos (recommandé 2024)
```

**Via PowerShell :**

```powershell
# Activer Microsoft Entra Kerberos
Set-AzStorageAccount `
  -ResourceGroupName "myResourceGroup" `
  -Name "mystorageaccount" `
  -EnableAzureActiveDirectoryKerberosForFile $true
```

**Étape 2 : Configurer les Permissions RBAC sur le File Share**

```bash
# Obtenir l'ID du file share
$fileShareResourceId = (Get-AzStorageShare `
  -Context (New-AzStorageContext -StorageAccountName "mystorageaccount" -UseConnectedAccount) `
  -Name "myfileshare").Id

# Assigner le rôle "Storage File Data SMB Share Contributor"
az role assignment create \
  --assignee user@contoso.com \
  --role "Storage File Data SMB Share Contributor" \
  --scope "/subscriptions/xxx/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/mystorageaccount/fileServices/default/fileshares/myfileshare"
```

**Rôles RBAC pour Azure Files (SMB) :**

| Rôle | Permissions | Use Case |
|------|------------|----------|
| **Storage File Data SMB Share Reader** | Lecture seule | Consultation, audits |
| **Storage File Data SMB Share Contributor** | Lecture, écriture, modification | Utilisateurs standard |
| **Storage File Data SMB Share Elevated Contributor** | Lecture, écriture, modification, changement ACLs | Administrateurs de données |

**Étape 3 : Configurer les ACLs Windows (NTFS Permissions)**

```powershell
# Monter le file share avec identité Azure AD
$connectTestResult = Test-NetConnection -ComputerName mystorageaccount.file.core.windows.net -Port 445
if ($connectTestResult.TcpTestSucceeded) {
    # Mapper le lecteur réseau
    New-PSDrive -Name Z -PSProvider FileSystem `
      -Root "\\mystorageaccount.file.core.windows.net\myfileshare" `
      -Persist
    
    # Configurer NTFS permissions (comme Windows traditionnel)
    icacls Z:\ /grant "user@contoso.com:(OI)(CI)M"
}
```

**Étape 4 : Accès depuis les Clients**

```powershell
# Windows 10/11 - Connexion avec identité Azure AD
net use Z: \\mystorageaccount.file.core.windows.net\myfileshare /user:Azure\user@contoso.com

# Ou avec cmdkey (SSO)
cmdkey /add:mystorageaccount.file.core.windows.net /user:Azure\user@contoso.com /pass:<password>
net use Z: \\mystorageaccount.file.core.windows.net\myfileshare
```

**⚠️ Types d'Identity-Based Access pour Azure Files :**

**1. Microsoft Entra Kerberos (Recommandé 2024)**
- **Scenario** : Cloud-only ou hybrid identity
- **Prérequis** : Azure AD tenant
- **Avantage** : Pas besoin de Azure AD Domain Services (AADDS)
- **Limitation** : SMB uniquement

**2. Azure AD Domain Services (AADDS) - Legacy**
- **Scenario** : Environnements entièrement cloud
- **Prérequis** : AADDS déployé (coût supplémentaire ~$100/mois)
- **Avantage** : Émulation complète d'AD Domain Controller
- **Limitation** : Complexité et coût

**3. On-Premises AD DS avec Azure AD Connect**
- **Scenario** : Environnements hybrides
- **Prérequis** : AD on-premises + Azure AD Connect + connectivité
- **Avantage** : Intégration complète avec AD existant
- **Limitation** : Infrastructure on-premises requise

**Comparaison des Méthodes Identity-Based :**

| Critère | Microsoft Entra Kerberos | Azure AD DS | On-Premises AD DS |
|---------|-------------------------|-------------|-------------------|
| **Coût** | ✅ Gratuit | ❌ ~$100/mois | ⚠️ Infrastructure existante |
| **Complexité** | ✅ Simple | ⚠️ Moyenne | ❌ Complexe |
| **Cloud-Only** | ✅ Oui | ✅ Oui | ❌ Non (hybrid) |
| **Prérequis** | Azure AD | AADDS déployé | AD + Azure AD Connect |
| **Recommandation 2024** | ✅ Préféré | ⚠️ Legacy | ⚠️ Si hybrid requis |

**⚠️ Scénario d'Examen - Identity-Based Access :**

```
Question : Which storage data service can be configured to use identity-based access?
A) containers (Blob Storage)
B) file shares (Azure Files)  ✅ CORRECT
C) queues
D) tables

Explication :
- Azure Files supporte Microsoft Entra Kerberos (IAM)
- Les utilisateurs s'authentifient avec leurs identités Azure AD
- RBAC + NTFS permissions pour contrôle d'accès granulaire
- Blob, Queue, Table utilisent Azure AD OAuth mais PAS Kerberos
```

---

### 2. Authentification Azure AD (OAuth 2.0) - IAM pour Blobs, Queues, Tables

**Définition :**
- **Azure AD OAuth 2.0** : Authentification IAM basée sur tokens pour accès programmatique
- **Type** : **Identity and Access Management (IAM)**
- **Scope** : Blob, Queue, Table Storage (pas SMB pour Files)
- **Méthode** : Utilisée par applications, service principals, managed identities

**⚠️ Différence Kerberos vs OAuth pour Identity-Based :**

| Aspect | Kerberos (Azure Files SMB) | OAuth 2.0 (Blob/Queue/Table) |
|--------|---------------------------|------------------------------|
| **Protocol** | Kerberos (SMB) | OAuth 2.0 (REST API) |
| **Usage** | Mappage réseau, utilisateurs finaux | Applications, services, APIs |
| **SSO** | ✅ Oui (Windows integrated) | ⚠️ Via managed identities |
| **Permissions** | RBAC + NTFS ACLs | RBAC uniquement |
| **Exam Focus** | ✅ Identity-based access | ⚠️ Programmatic access |

**Utilisation Azure AD pour Blob Storage :**

```bash
# Authentification avec Azure AD (pas de clés)
az login

# Accès à un blob avec identité
az storage blob download \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myblob.txt \
  --file ./myblob.txt \
  --auth-mode login

# Via Managed Identity dans une application
from azure.identity import DefaultAzureCredential
from azure.storage.blob import BlobServiceClient

credential = DefaultAzureCredential()
blob_service_client = BlobServiceClient(
    account_url="https://mystorageaccount.blob.core.windows.net",
    credential=credential
)
```

**Rôles RBAC pour Blob Storage :**
- **Storage Blob Data Reader** : Lecture seule
- **Storage Blob Data Contributor** : Lecture + écriture
- **Storage Blob Data Owner** : Full control + ACLs (ADLS Gen2)

---

### 3. Shared Key (Account Key) - CREDENTIAL-BASED

**Définition :**
- **Shared Key** : Clé secrète du compte de stockage (512-bit)
- **Type** : **Credential-based** (NOT IAM)
- **Scope** : Tous les services (Blob, File, Queue, Table)
- **Permissions** : Accès COMPLET au compte de stockage

**⚠️ Important :**
- ❌ **PAS de l'IAM** : C'est une clé partagée, pas basé sur l'identité
- ❌ **Trop de privilèges** : Accès complet à toutes les ressources
- ⚠️ **À éviter** : Utiliser Azure AD ou SAS quand possible

**Utilisation :**

```bash
# Via Azure CLI
az storage blob download \
  --account-name mystorageaccount \
  --account-key "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" \
  --container-name mycontainer \
  --name myblob.txt \
  --file ./myblob.txt
```

**Rotation des Clés :**

```bash
# Régénérer key1
az storage account keys renew \
  --account-name mystorageaccount \
  --resource-group myResourceGroup \
  --key key1

# Processus recommandé :
# 1. Applications utilisent key1
# 2. Régénérer key2
# 3. Basculer applications vers key2
# 4. Régénérer key1
# 5. Répéter tous les 90 jours
```

---

### 4. Shared Access Signature (SAS) - CREDENTIAL-BASED avec Délégation

**Définition :**
- **SAS** : Token d'accès délégué avec permissions limitées
- **Type** : **Credential-based** (sauf User Delegation SAS qui est IAM)
- **Scope** : Granulaire (ressource spécifique)
- **Avantage** : Délégation sécurisée sans partager clés du compte

**⚠️ 3 Types de SAS et leur Relation avec IAM :**

| Type SAS | Basé sur | IAM ou Credential | Scope |
|----------|----------|-------------------|-------|
| **User Delegation SAS** | Azure AD (OAuth) | ✅ **IAM** | Blob, ADLS Gen2 |
| **Service SAS** | Account Key | ❌ Credential | 1 service |
| **Account SAS** | Account Key | ❌ Credential | Multi-services |

**⚠️ Seul User Delegation SAS est considéré IAM !**

```bash
# User Delegation SAS (IAM) - Recommandé
az storage blob generate-sas \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myblob.txt \
  --permissions r \
  --expiry 2024-12-31T23:59:59Z \
  --auth-mode login \
  --as-user

# Service SAS (Credential-based)
az storage blob generate-sas \
  --account-name mystorageaccount \
  --account-key "xxxx" \
  --container-name mycontainer \
  --name myblob.txt \
  --permissions r \
  --expiry 2024-12-31T23:59:59Z
```

---

### 5. Anonymous Public Access - NO AUTHENTICATION

**Définition :**
- **Anonymous Public Access** : Accès public sans authentification
- **Type** : **Aucune authentification**
- **Scope** : Blob Storage uniquement (containers et blobs)
- **Use Case** : Contenu public (images, CSS, JS pour websites)

**Configuration :**

```bash
# Activer anonymous access sur le storage account
az storage account update \
  --name mystorageaccount \
  --resource-group myResourceGroup \
  --allow-blob-public-access true

# Configurer anonymous access sur un container
az storage container set-permission \
  --name mycontainer \
  --account-name mystorageaccount \
  --public-access blob

# Niveaux :
# --public-access off : Pas d'accès anonyme
# --public-access blob : Accès blob individuel
# --public-access container : Accès container + blobs
```

**⚠️ Sécurité :**
- ❌ **Risque** : Données accessibles publiquement
- ✅ **Best Practice** : Désactiver par défaut au niveau compte
- ✅ **Audit** : Vérifier régulièrement les containers publics

---

### Tableau Récapitulatif - Méthodes d'Authentification par Service

**⚠️ À CONNAÎTRE PAR CŒUR pour l'Examen AZ-104 :**

| Service | Kerberos (IAM) | Azure AD OAuth (IAM) | Shared Key | SAS | Anonymous |
|---------|----------------|----------------------|------------|-----|-----------|
| **Azure Files (SMB)** | ✅ **SEUL service avec Kerberos** | ⚠️ REST API uniquement | ✅ Oui | ✅ Oui | ❌ Non |
| **Blob Storage** | ❌ Non | ✅ Oui (REST) | ✅ Oui | ✅ Oui | ✅ Oui |
| **Queue Storage** | ❌ Non | ✅ Oui (REST) | ✅ Oui | ✅ Oui | ❌ Non |
| **Table Storage** | ❌ Non | ✅ Oui (REST) | ✅ Oui | ✅ Oui | ❌ Non |

**⚠️ Scénarios d'Examen par Méthode :**

**Scénario 1 : Utilisateurs Finaux Accédant à File Shares**

```
Requirement : Les employés doivent accéder à des file shares avec leurs identités Azure AD
Question : Quelle méthode utiliser ?

Réponse : Microsoft Entra Kerberos ✅

Configuration :
1. Activer Microsoft Entra Kerberos sur storage account
2. Assigner RBAC (Storage File Data SMB Share Contributor)
3. Configurer NTFS permissions
4. Utilisateurs mappent le lecteur avec Azure\user@contoso.com

Type : IAM (Identity-Based Access)
```

**Scénario 2 : Application Azure VM Accédant à Blob Storage**

```
Requirement : VM doit lire/écrire des blobs sans clés dans le code
Question : Quelle méthode utiliser ?

Réponse : Managed Identity + Azure AD RBAC ✅

Configuration :
1. Activer System-Assigned Managed Identity sur VM
2. Assigner RBAC (Storage Blob Data Contributor)
3. Application utilise DefaultAzureCredential()

Type : IAM (Identity-Based Access via OAuth)
```

**Scénario 3 : Partage Temporaire avec Client Externe**

```
Requirement : Partager un fichier avec un partenaire externe pour 7 jours
Question : Quelle méthode utiliser ?

Réponse : User Delegation SAS ✅ (si Blob) ou Service SAS (si File)

Configuration :
1. User Delegation SAS pour Blobs (IAM)
2. Service SAS pour Files (Credential)
3. Expiration 7 jours
4. HTTPS uniquement

Type : IAM (User Delegation) ou Credential (Service SAS)
```

**Scénario 4 : Website Statique Public**

```
Requirement : Images et CSS doivent être accessibles publiquement
Question : Quelle méthode utiliser ?

Réponse : Anonymous Public Access ✅

Configuration :
1. Activer anonymous access sur storage account
2. Container public-access level = blob
3. Uploader images dans container

Type : Aucune authentification
```

**⚠️ Erreurs Courantes QCM :**

| Question | Réponse Incorrecte ❌ | Réponse Correcte ✅ |
|----------|----------------------|---------------------|
| **"Quel service supporte Kerberos identity-based access ?"** | "Blob Storage" | "Azure Files (SMB) uniquement" |
| **"User Delegation SAS est IAM ?"** | "Non, c'est credential-based" | "Oui, basé sur Azure AD (IAM)" |
| **"Account Key est identity-based ?"** | "Oui, lié à l'account" | "Non, c'est credential-based (clé partagée)" |
| **"Blob Storage supporte identity-based access ?"** | "Non, seulement clés" | "Oui, via Azure AD OAuth (mais pas Kerberos)" |
| **"Peut-on utiliser Kerberos pour Queues ?"** | "Oui, comme Files" | "Non, Kerberos = Azure Files SMB uniquement" |

**⚠️ Différence IAM vs Credential - Points Clés :**

**Identity-Based (IAM) :**
- ✅ Authentification basée sur QUI vous êtes (identité)
- ✅ Pas de secrets dans le code
- ✅ Révocation via Azure AD
- ✅ Audit trail complet
- ✅ **Exemples** : Kerberos (Files SMB), Azure AD OAuth, User Delegation SAS, Managed Identities

**Credential-Based :**
- ⚠️ Authentification basée sur CE QUE vous avez (secret)
- ❌ Secrets/clés à gérer
- ⚠️ Révocation = rotation clés
- ⚠️ Moins d'audit trail
- ⚠️ **Exemples** : Account Key, Service SAS, Account SAS

**⚠️ Points Clés pour l'Examen :**
- ✅ **Kerberos = IAM = Azure Files SMB UNIQUEMENT**
- ✅ **Identity-based access** dans questions d'examen = Chercher **Kerberos** ou **Azure Files**
- ✅ **User Delegation SAS** = Seul SAS qui est IAM (Blob/ADLS Gen2)
- ✅ **OAuth 2.0** = IAM mais via REST API (pas mappage réseau)
- ✅ **Account Key** = PAS IAM (credential-based)
- ✅ **RBAC requis** pour toutes les méthodes IAM
- ✅ **Microsoft Entra Kerberos** = Nouveau nom pour Azure AD Kerberos (2024)
- ✅ **3 niveaux** : RBAC (share-level) + NTFS (file-level) + Azure AD (tenant-level)

### Azure File Sync

**⚠️ Concept Clé pour AZ-104 : Azure File Sync centralise vos partages de fichiers dans Azure Files tout en conservant la flexibilité, la performance et la compatibilité de Windows Server**

**Définition :**
- **Azure File Sync** : Service qui synchronise les fichiers entre des serveurs Windows on-premises et Azure Files
- **Objectif** : Cache distribué d'Azure Files sur site, avec tiering cloud optionnel
- **Use Case Principal** : Migration hybride, disaster recovery, consolidation de file servers

**Architecture Azure File Sync :**

```
┌─────────────────────────────────────────────────────────┐
│                     Azure (Cloud)                        │
│  ┌─────────────────────────────────────────────────┐   │
│  │         Storage Sync Service                     │   │
│  │    (Orchestration et gestion centrale)           │   │
│  └──────────────┬──────────────────────────────────┘   │
│                 │                                        │
│  ┌──────────────▼──────────────────────────────────┐   │
│  │        Azure File Share (Cloud Endpoint)         │   │
│  │     ─────────────────────────────────────────    │   │
│  │          Source centrale de vérité               │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
                         ▲ ▼ Sync
┌─────────────────────────────────────────────────────────┐
│              On-Premises / Branch Offices                │
│  ┌─────────────────┐  ┌─────────────────┐              │
│  │  Windows Server │  │  Windows Server │              │
│  │   (HQ Office)   │  │ (Branch Office) │              │
│  │  ┌───────────┐  │  │  ┌───────────┐  │              │
│  │  │Azure File │  │  │  │Azure File │  │              │
│  │  │Sync Agent │  │  │  │Sync Agent │  │              │
│  │  └─────┬─────┘  │  │  └─────┬─────┘  │              │
│  │  ┌─────▼─────┐  │  │  ┌─────▼─────┐  │              │
│  │  │ Registered│  │  │  │ Registered│  │              │
│  │  │  Servers  │  │  │  │  Servers  │  │              │
│  │  └───────────┘  │  │  └───────────┘  │              │
│  └─────────────────┘  └─────────────────┘              │
│     Server Endpoint       Server Endpoint               │
└─────────────────────────────────────────────────────────┘
```

**Composants Principaux :**

**1. Storage Sync Service**
- **Rôle** : Ressource Azure de niveau supérieur pour orchestrer la synchronisation
- **Création** : Via Azure Portal, PowerShell ou CLI
- **Région** : Doit être dans la même région que le Storage Account
- **Gestion** : Point central pour tous les sync groups et registered servers

**2. Sync Group**
- **Définition** : Définit la topologie de synchronisation pour un ensemble de fichiers
- **Contenu** : 1 Cloud Endpoint + 1 ou plusieurs Server Endpoints
- **Limites** : Jusqu'à 100 sync groups par Storage Sync Service

**3. Cloud Endpoint**
- **Définition** : Azure File Share dans le cloud
- **Limite** : Un seul cloud endpoint par sync group
- **Requis** : Azure File Share doit être vide ou contenir uniquement des données à synchroniser

**4. Server Endpoint**
- **Définition** : Chemin spécifique sur un Windows Server enregistré (ex: D:\Shares\Finance)
- **Limite** : Jusqu'à 100 server endpoints par sync group
- **Contrainte** : Un seul server endpoint par volume (disque) sur un serveur

**5. Registered Server**
- **Définition** : Windows Server avec l'agent Azure File Sync installé
- **Limite** : Jusqu'à 99 registered servers par Storage Sync Service
- **Prérequis** : Windows Server 2016 ou supérieur (2019/2022 recommandé)

**Installation et Configuration - Étapes Détaillées :**

**Étape 1 : Prérequis**

```powershell
# Vérifier la version de Windows Server
Get-ComputerInfo | Select-Object WindowsProductName, OSDisplayVersion

# Versions supportées :
# - Windows Server 2016 (minimum)
# - Windows Server 2019 (recommandé)
# - Windows Server 2022
# - Azure Virtual Machines supportées

# Vérifier la connectivité réseau
Test-NetConnection -ComputerName file.core.windows.net -Port 443
Test-NetConnection -ComputerName kailani-afs.one.microsoft.com -Port 443
```

**Étape 2 : Créer Storage Sync Service**

```bash
# Via Azure CLI
az resource create \
  --resource-group myResourceGroup \
  --name myStorageSyncService \
  --resource-type "Microsoft.StorageSync/storageSyncServices" \
  --location "westeurope" \
  --properties '{}'

# Via PowerShell
New-AzStorageSyncService `
  -ResourceGroupName "myResourceGroup" `
  -Name "myStorageSyncService" `
  -Location "westeurope"
```

**Étape 3 : Installer Azure File Sync Agent sur Windows Server**

```powershell
# Télécharger et installer l'agent (PowerShell sur le serveur Windows)
# Lien : https://go.microsoft.com/fwlink/?linkid=858257

# Installation silencieuse
Start-Process -FilePath "StorageSyncAgent.msi" -ArgumentList "/quiet /qn" -Wait

# Vérifier l'installation
Get-Module -ListAvailable -Name StorageSync
```

**Étape 4 : Enregistrer le Windows Server**

```powershell
# Se connecter à Azure
Connect-AzAccount

# Enregistrer le serveur
Register-AzStorageSyncServer `
  -ResourceGroupName "myResourceGroup" `
  -StorageSyncServiceName "myStorageSyncService"

# Vérifier l'enregistrement
Get-AzStorageSyncServer `
  -ResourceGroupName "myResourceGroup" `
  -StorageSyncServiceName "myStorageSyncService"
```

**Étape 5 : Créer Sync Group**

```powershell
# Via PowerShell
New-AzStorageSyncGroup `
  -ResourceGroupName "myResourceGroup" `
  -StorageSyncServiceName "myStorageSyncService" `
  -SyncGroupName "SyncGroup01"
```

**Étape 6 : Créer Cloud Endpoint**

```powershell
# Obtenir l'ID du Storage Account et File Share
$storageAccount = Get-AzStorageAccount `
  -ResourceGroupName "myResourceGroup" `
  -Name "mystorageaccount"

# Créer le cloud endpoint
New-AzStorageSyncCloudEndpoint `
  -ResourceGroupName "myResourceGroup" `
  -StorageSyncServiceName "myStorageSyncService" `
  -SyncGroupName "SyncGroup01" `
  -StorageAccountResourceId $storageAccount.Id `
  -AzureFileShareName "myfileshare"
```

**Étape 7 : Créer Server Endpoint**

```powershell
# Obtenir le Registered Server ID
$registeredServer = Get-AzStorageSyncServer `
  -ResourceGroupName "myResourceGroup" `
  -StorageSyncServiceName "myStorageSyncService"

# Créer le server endpoint
New-AzStorageSyncServerEndpoint `
  -ResourceGroupName "myResourceGroup" `
  -StorageSyncServiceName "myStorageSyncService" `
  -SyncGroupName "SyncGroup01" `
  -ServerId $registeredServer.ResourceId `
  -ServerLocalPath "D:\Shares\Finance" `
  -CloudTiering `
  -VolumeFreeSpacePercent 20 `
  -TierFilesOlderThanDays 30
```

**⚠️ Cloud Tiering - Fonctionnalité Clé :**

**Définition :**
- **Cloud Tiering** : Fonctionnalité optionnelle qui garde les fichiers fréquemment accédés en local et "tiers" les fichiers froids vers Azure
- **Avantage** : Libère de l'espace disque local tout en maintenant l'accès transparent
- **Fonctionnement** : Remplace les fichiers par des "reparse points" (pointeurs)

**Politiques de Tiering :**

**1. Volume Free Space Policy**
```powershell
# Conserver 20% d'espace libre sur le volume
-VolumeFreeSpacePercent 20
```
- **Comportement** : Tiers automatiquement les fichiers les moins récemment accédés jusqu'à atteindre 20% d'espace libre
- **Recommandé** : 20-30% pour équilibrer performance et capacité

**2. Date Policy**
```powershell
# Tier les fichiers non accédés depuis 30 jours
-TierFilesOlderThanDays 30
```
- **Comportement** : Les fichiers non accédés pendant X jours sont tiered vers le cloud
- **Recommandé** : 7-60 jours selon les patterns d'accès

**Fonctionnement du Tiering :**

```
Fichier Initial (Local)
├── Taille : 100 MB
└── Accès : Dernier accès il y a 45 jours

     ▼ Tiering Policy Applied (TierFilesOlderThanDays 30)

Fichier Tiered (Reparse Point Local)
├── Taille locale : ~1 KB (métadonnées)
├── Taille cloud : 100 MB (dans Azure File Share)
└── Comportement : Transparent pour l'utilisateur

Utilisateur ouvre le fichier
     ▼ Automatic Recall

Fichier Recalled (Local à nouveau)
├── Téléchargement depuis Azure
├── Fichier complet disponible localement
└── Prochains accès instantanés
```

**Recall de Fichiers :**

```powershell
# Recall manuel d'un fichier spécifique
Invoke-StorageSyncFileRecall -Path "D:\Shares\Finance\ImportantDoc.pdf"

# Recall d'un dossier entier
Invoke-StorageSyncFileRecall -Path "D:\Shares\Finance\Q4Reports" -Recurse

# Vérifier l'état du tiering
Get-StorageSyncFileTieringResult -Path "D:\Shares\Finance"
```

**⚠️ Limites et Contraintes Azure File Sync :**

| Limite | Valeur | Notes |
|--------|--------|-------|
| **Storage Sync Services par subscription** | 100 | Par région Azure |
| **Sync Groups par Storage Sync Service** | 200 | Anciennement 100, augmenté en 2024 |
| **Cloud Endpoints par Sync Group** | 1 | Un seul Azure File Share |
| **Server Endpoints par Sync Group** | 100 | Plusieurs serveurs possibles |
| **Registered Servers par Storage Sync Service** | 99 | Windows Servers enregistrés |
| **Taille maximale fichier** | 100 TiB | Pour Premium File Shares |
| **Nombre de fichiers par volume** | ~100 millions | Dépend de la taille namespace |
| **Longueur chemin maximum** | 2048 caractères | Limitation Windows |

**⚠️ Scénarios d'Utilisation - Pour l'Examen :**

**Scénario 1 : Consolidation de File Servers de Branches**

```
Problème : 
- 20 bureaux régionaux avec leurs propres file servers
- Données dupliquées et non synchronisées
- Coût de maintenance élevé

Solution Azure File Sync :
1. Créer Azure File Share central (cloud endpoint)
2. Installer agents sur les 20 serveurs locaux
3. Créer 20 server endpoints dans le même sync group
4. Activer cloud tiering pour optimiser l'espace disque local
5. Résultat : Source de vérité unique dans Azure, accès local rapide

Avantages :
✅ Données centralisées dans Azure
✅ Cache local pour performance
✅ Synchronisation multi-sites automatique
✅ Réduction coûts infrastructure
```

**Scénario 2 : Disaster Recovery**

```
Problème :
- File server critique on-premises sans HA
- RTO/RPO élevés en cas de panne
- Backups traditionnels lents à restaurer

Solution Azure File Sync :
1. Synchroniser file server vers Azure Files
2. En cas de panne du serveur :
   - Monter Azure File Share directement depuis Azure VMs
   - Ou restaurer sur nouveau serveur avec sync
3. RTO < 1 heure au lieu de plusieurs heures

Avantages :
✅ Copie cloud automatique et continue
✅ Récupération rapide
✅ Pas besoin de restauration backup complète
```

**Scénario 3 : Lift and Shift avec Cache Local**

```
Problème :
- Migration d'applications vers Azure
- Applications nécessitent accès fichiers local rapide
- Bande passante WAN limitée vers Azure

Solution Azure File Sync :
1. Azure File Share comme stockage primaire
2. Azure File Sync sur Azure VMs pour cache local
3. Cloud tiering pour optimiser stockage VM
4. Fichiers fréquents en local (performance)
5. Fichiers froids dans le cloud (coût)

Avantages :
✅ Performance locale maintenue
✅ Capacité illimitée (Azure)
✅ Coûts optimisés
```

**⚠️ Monitoring et Troubleshooting :**

**Vérifier l'État de Synchronisation :**

```powershell
# Statut général du sync
Get-AzStorageSyncServerEndpoint `
  -ResourceGroupName "myResourceGroup" `
  -StorageSyncServiceName "myStorageSyncService" `
  -SyncGroupName "SyncGroup01"

# Activité de synchronisation récente
Get-AzStorageSyncSyncSessionStatus `
  -ResourceGroupName "myResourceGroup" `
  -StorageSyncServiceName "myStorageSyncService" `
  -SyncGroupName "SyncGroup01"

# Fichiers non synchronisés (erreurs)
Get-StorageSyncFileSyncActivity -ServerEndpointPath "D:\Shares\Finance"
```

**Logs de Diagnostic :**

```
Event Viewer (Windows Server) :
- Applications and Services Logs
  └── Microsoft
      └── FileSync
          └── Agent
              ├── FileSyncEventLog
              ├── FileSyncTelemetryLog
              └── FileSyncRecallLog
```

**Métriques Azure Monitor :**

```bash
# Via Azure Portal
Storage Sync Service → Monitoring → Metrics

Métriques Clés :
- Sync session result (success/failure)
- Files synced
- Bytes synced
- Cloud tiering recall size
- Cloud tiering recall throughput
- Server online/offline status
```

**Problèmes Courants et Solutions :**

| Problème | Cause | Solution |
|----------|-------|----------|
| **Sync ne démarre pas** | Firewall bloque port 443 | Ouvrir ports 443 vers *.afs.azure.net |
| **Fichiers ne sont pas tiered** | Cloud tiering désactivé | Vérifier `-CloudTiering` paramètre |
| **Recall très lent** | Bande passante limitée | Augmenter QoS, vérifier throttling |
| **Erreur "ECS_E_SYNC_METADATA_KNOWLEDGE_SOFT_LIMIT_REACHED"** | Trop de fichiers modifiés | Augmenter sync session, planifier syncs |
| **Server endpoint health "No Activity"** | Agent non démarré | Redémarrer service FileSyncSvc |

**⚠️ Erreurs Courantes QCM :**

| Question | Réponse Incorrecte ❌ | Réponse Correcte ✅ |
|----------|----------------------|---------------------|
| **"Peut-on synchroniser entre 2 Azure File Shares directement ?"** | "Oui, avec Azure File Sync" | "Non, Azure File Sync synchronise serveurs Windows vers Azure Files" |
| **"Cloud Tiering supprime les fichiers du cloud ?"** | "Oui, pour libérer espace" | "Non, garde fichiers dans Azure et libère espace local" |
| **"Combien de cloud endpoints par sync group ?"** | "Illimité" ou "10" | "1 seul cloud endpoint par sync group" |
| **"Azure File Sync fonctionne sur Linux ?"** | "Oui, avec agent Linux" | "Non, Windows Server uniquement" |
| **"Peut-on avoir 2 server endpoints sur le même volume ?"** | "Oui, dans des dossiers différents" | "Non, un seul server endpoint par volume" |

**Comparaison avec Alternatives :**

| Solution | Use Case | Différence avec File Sync |
|----------|----------|---------------------------|
| **Azure Files Direct Mount** | Accès SMB simple sans cache | Pas de cache local, latence WAN complète |
| **Azure File Sync** | Cache distribué + synchronisation | Cache local, sync multi-sites, cloud tiering |
| **Storage Account Replication (GRS)** | Disaster Recovery stockage | Réplication au niveau stockage, pas de cache applicatif |
| **DFS Replication** | Synchronisation on-premises uniquement | Pas de cloud, maintenance complexe |

**Coûts Azure File Sync :**

```
Composants de Coût :
1. Azure File Share stockage (selon tier et taille)
2. Transactions de synchronisation (lectures/écritures)
3. Egress data (recall depuis cloud)
4. Storage Sync Service : GRATUIT
5. Agent Azure File Sync : GRATUIT

Optimisation :
- Utiliser Cool tier pour fichiers rarement accédés
- Activer cloud tiering pour réduire stockage local
- Monitorer recall patterns pour ajuster policies
```

**⚠️ Best Practices :**

**1. Planification**
- ✅ Évaluer patterns d'accès fichiers avant déploiement
- ✅ Dimensionner bande passante réseau (100 Mbps minimum recommandé)
- ✅ Tester avec pilot group avant déploiement complet

**2. Configuration**
- ✅ Activer cloud tiering pour serveurs avec espace limité
- ✅ Utiliser Date Policy (30-60 jours) ET Volume Free Space (20-30%)
- ✅ Créer server endpoints au niveau racine de volume si possible

**3. Sécurité**
- ✅ Utiliser HTTPS uniquement (port 443)
- ✅ Activer Azure AD authentication pour Azure Files
- ✅ Appliquer RBAC sur Storage Sync Service
- ✅ Activer encryption at rest et in transit

**4. Monitoring**
- ✅ Configurer alertes Azure Monitor pour sync failures
- ✅ Vérifier health status quotidiennement
- ✅ Monitorer métriques de tiering et recall

**5. Maintenance**
- ✅ Maintenir agents à jour (auto-update recommandé)
- ✅ Vérifier espace disque régulièrement
- ✅ Tester disaster recovery procedures

**⚠️ Points Clés pour l'Examen :**
- ✅ Azure File Sync = **Cache distribué** d'Azure Files sur Windows Server
- ✅ **1 cloud endpoint** (Azure File Share) par sync group
- ✅ **Jusqu'à 100 server endpoints** (serveurs Windows) par sync group
- ✅ **Cloud Tiering** : Libère espace local, garde fichiers dans Azure
- ✅ **Recall automatique** : Fichiers téléchargés à la demande depuis Azure
- ✅ **Windows Server uniquement** : Pas de support Linux
- ✅ **1 server endpoint par volume** : Contrainte importante
- ✅ **Synchronisation bidirectionnelle** : Modifications synchronisées dans les deux sens
- ✅ **Registered Server** : Windows Server avec agent installé
- ✅ **Use Cases** : Consolidation file servers, DR, migration hybride

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

## 2.6 Shared Access Signatures (SAS)

**âš ï¸ Concept Clé pour AZ-104 : Les Shared Access Signatures (SAS) fournissent un accès délégué sécurisé aux ressources de stockage Azure sans partager les clés du compte**

**Définition :**
- **Shared Access Signature (SAS)** : URI qui accorde des droits d'accès restreints aux ressources Azure Storage
- **Objectif** : Délégation d'accès granulaire avec contrôle précis des permissions et de la durée
- **Avantage Principal** : Partage sécurisé sans exposer les clés du compte de stockage

**Principe de Fonctionnement :**

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚            Storage Account                          â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”     â”‚
â”‚  â”‚       Account Key (Ne JAMAIS partager)    â”‚     â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜     â”‚
â”‚                   â”‚                                 â”‚
â”‚                   ▼ Signe le SAS Token              â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”     â”‚
â”‚  â”‚      SAS Token Généré                     â”‚     â”‚
â”‚  â”‚  ?sv=2021-06-08&ss=b&srt=sco&sp=rwdlacx  â”‚     â”‚
â”‚  â”‚  &se=2024-12-31T23:59:59Z&st=...&sig=... â”‚     â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜     â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¼â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
                    â”‚
                    ▼ Partagé avec
         â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
         â”‚   Application Tierce  â”‚
         â”‚   ou Utilisateur      â”‚
         â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
                    â”‚
                    ▼ Accès limité aux ressources
         â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
         â”‚   Blob / Container    â”‚
         â”‚   (lecture seule, etc)â”‚
         â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### Types de SAS

**âš ï¸ Il existe 3 types principaux de SAS - Chacun avec des use cases spécifiques**

**1. User Delegation SAS (Recommandé - Plus Sécurisé)**

**Caractéristiques :**
- **Authentification** : Basée sur Azure AD credentials (OAuth 2.0)
- **Signature** : Utilise une User Delegation Key obtenue d'Azure AD
- **Avantage** : NE nécessite PAS les clés du compte de stockage
- **Scope** : Blob Storage et Data Lake Storage Gen2 uniquement
- **Sécurité** : Meilleure pratique recommandée par Microsoft

**Création - User Delegation SAS :**

```bash
# Prérequis : Utilisateur doit avoir le rôle "Storage Blob Data Contributor" ou supérieur

# Étape 1 : Se connecter avec Azure AD
az login

# Étape 2 : Générer User Delegation SAS
az storage blob generate-sas \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myblob.txt \
  --permissions r \
  --expiry 2024-12-31T23:59:59Z \
  --auth-mode login \
  --as-user

# Output : SAS Token
# ?sv=2021-06-08&st=2024-01-01T00:00:00Z&se=2024-12-31T23:59:59Z&sr=b&sp=r&sig=...

# Construire URL complète
https://mystorageaccount.blob.core.windows.net/mycontainer/myblob.txt?sv=2021-06-08&st=...&sig=...
```

**Via PowerShell avec Azure AD :**

```powershell
# Se connecter avec Azure AD
Connect-AzAccount

# Obtenir le contexte du Storage Account
$ctx = New-AzStorageContext -StorageAccountName "mystorageaccount" -UseConnectedAccount

# Générer User Delegation SAS pour un blob
$sasToken = New-AzStorageBlobSASToken `
  -Context $ctx `
  -Container "mycontainer" `
  -Blob "myblob.txt" `
  -Permission r `
  -ExpiryTime (Get-Date).AddDays(7)

# URL complète
$blobUri = $ctx.BlobEndPoint + "mycontainer/myblob.txt" + $sasToken
Write-Output $blobUri
```

**âš ï¸ Avantages User Delegation SAS :**
- ✅ **Pas de clés exposées** : N'utilise pas les clés du compte
- ✅ **Azure AD integration** : Révocation via Azure AD
- ✅ **Audit trail** : Traçabilité complète dans Azure AD logs
- ✅ **Rotation automatique** : User Delegation Key rotée automatiquement
- ✅ **Least privilege** : Basé sur RBAC Azure AD

**2. Service SAS**

**Caractéristiques :**
- **Authentification** : Signée avec la clé du compte de stockage
- **Scope** : Ressource spécifique dans UN seul service (Blob, Queue, Table, File)
- **Granularité** : Accès Ã  un blob, container, file share, queue, ou table
- **Use Case** : Délégation d'accès Ã  des ressources spécifiques

**Création - Service SAS :**

```bash
# Service SAS pour un blob spécifique
az storage blob generate-sas \
  --account-name mystorageaccount \
  --account-key "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" \
  --container-name mycontainer \
  --name myblob.txt \
  --permissions rw \
  --expiry 2024-12-31T23:59:59Z \
  --https-only

# Service SAS pour un container entier
az storage container generate-sas \
  --account-name mystorageaccount \
  --account-key "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" \
  --name mycontainer \
  --permissions rl \
  --expiry 2024-12-31T23:59:59Z

# Service SAS pour Azure File Share
az storage share generate-sas \
  --account-name mystorageaccount \
  --account-key "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" \
  --name myfileshare \
  --permissions rl \
  --expiry 2024-12-31T23:59:59Z
```

**Via PowerShell :**

```powershell
# Obtenir le contexte avec la clé du compte
$ctx = New-AzStorageContext -StorageAccountName "mystorageaccount" -StorageAccountKey "xxxx"

# Service SAS pour blob
$blobSAS = New-AzStorageBlobSASToken `
  -Context $ctx `
  -Container "mycontainer" `
  -Blob "myblob.txt" `
  -Permission rwd `
  -ExpiryTime (Get-Date).AddHours(24) `
  -Protocol HttpsOnly

# Service SAS pour container
$containerSAS = New-AzStorageContainerSASToken `
  -Context $ctx `
  -Name "mycontainer" `
  -Permission rl `
  -ExpiryTime (Get-Date).AddDays(7)
```

**3. Account SAS**

**Caractéristiques :**
- **Authentification** : Signée avec la clé du compte de stockage
- **Scope** : Accès Ã  PLUSIEURS services (Blob, Queue, Table, File)
- **Granularité** : Opérations au niveau du service ET de la ressource
- **Use Case** : Applications nécessitant accès multi-services

**Création - Account SAS :**

```bash
# Account SAS donnant accès Ã  Blob et File Services
az storage account generate-sas \
  --account-name mystorageaccount \
  --account-key "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx" \
  --services bf \
  --resource-types sco \
  --permissions rl \
  --expiry 2024-12-31T23:59:59Z \
  --https-only

# Paramètres :
# --services : b=Blob, f=File, q=Queue, t=Table
# --resource-types : s=Service, c=Container, o=Object
```

**Via PowerShell :**

```powershell
# Account SAS avec accès multi-services
$accountSAS = New-AzStorageAccountSASToken `
  -Service Blob,File `
  -ResourceType Service,Container,Object `
  -Permission rl `
  -ExpiryTime (Get-Date).AddMonths(1) `
  -Protocol HttpsOnly `
  -Context $ctx

Write-Output "Account SAS: $accountSAS"
```

**âš ï¸ Comparaison des 3 Types de SAS :**

| Critère | User Delegation SAS | Service SAS | Account SAS |
|---------|---------------------|-------------|-------------|
| **Authentification** | Azure AD (OAuth) | Account Key | Account Key |
| **Sécurité** | ✅ Meilleure | âŒ Clé exposée | âŒ Clé exposée |
| **Scope Services** | Blob, ADLS Gen2 | 1 service | Plusieurs services |
| **Révocation** | ✅ Via Azure AD | âŒ Rotation clé uniquement | âŒ Rotation clé uniquement |
| **Audit** | ✅ Azure AD logs | âš ï¸ Storage logs | âš ï¸ Storage logs |
| **Use Case** | Applications modernes | Accès ressource unique | Multi-services legacy |
| **Recommandation** | ✅ Préféré | âš ï¸ Si User Delegation impossible | âŒ Éviter si possible |

### SAS Tokens et Permissions

**âš ï¸ Structure d'un SAS Token :**

```
https://mystorageaccount.blob.core.windows.net/mycontainer/myblob.txt
?sv=2021-06-08           # Storage Service Version
&st=2024-01-01T00:00:00Z # Start Time (optionnel)
&se=2024-12-31T23:59:59Z # Expiry Time (obligatoire)
&sr=b                     # Signed Resource (b=blob, c=container)
&sp=r                     # Signed Permissions (r=read)
&sip=168.1.5.60-168.1.5.70 # Signed IP Range (optionnel)
&spr=https               # Signed Protocol (https only)
&sig=xxxxxxxxxxxxxx      # Signature (HMAC-SHA256)
```

**Paramètres Clés :**

**1. Signed Permissions (sp) :**

**Pour Blobs :**
- **r** : Read (lire contenu, propriétés, métadonnées)
- **a** : Add (ajouter des blocks Ã  un append blob)
- **c** : Create (créer ou upload un blob)
- **w** : Write (écrire ou upload - écrase le blob existant)
- **d** : Delete (supprimer le blob)
- **x** : Delete Version (supprimer une version de blob)
- **y** : Permanent Delete (suppression permanente - soft delete)
- **l** : List (lister les blobs dans un container)
- **t** : Tags (lire/écrire blob tags)
- **m** : Move (rename blob - ADLS Gen2)
- **e** : Execute (ADLS Gen2 - exécution répertoire)

**Pour Containers :**
- **r** : Read
- **a** : Add
- **c** : Create
- **w** : Write
- **d** : Delete
- **l** : List
- **t** : Tags

**Pour File Shares :**
- **r** : Read
- **c** : Create
- **w** : Write
- **d** : Delete
- **l** : List

**Exemples de Permissions Combinées :**

```bash
# Lecture seule
--permissions r

# Lecture et listage
--permissions rl

# Lecture, écriture, suppression (full control)
--permissions rwd

# Toutes les permissions
--permissions rwdlacxtme
```

**2. Signed Resource (sr) :**
- **b** : Blob
- **c** : Container
- **f** : File
- **s** : Share (File Share)
- **d** : Directory (ADLS Gen2)
- **bs** : Blob Service (Account SAS)
- **bfs** : Blob File System (ADLS Gen2)

**3. Start Time (st) et Expiry Time (se) :**

```bash
# Commence maintenant, expire dans 7 jours
--start "$(date -u +%Y-%m-%dT%H:%M:%SZ)" \
--expiry "$(date -u -d '+7 days' +%Y-%m-%dT%H:%M:%SZ)"

# Commence dans 1 heure, expire dans 24 heures
--start "$(date -u -d '+1 hour' +%Y-%m-%dT%H:%M:%SZ)" \
--expiry "$(date -u -d '+1 day' +%Y-%m-%dT%H:%M:%SZ)"

# PowerShell équivalent
-StartTime (Get-Date) `
-ExpiryTime (Get-Date).AddDays(7)
```

**âš ï¸ Important :** 
- **Expiry obligatoire** pour tous les SAS
- **Start time optionnel** (si omis, commence immédiatement)
- **Clock skew** : Azure accepte jusqu'Ã  15 minutes de différence d'horloge

**4. Signed IP Range (sip) - Restriction par IP :**

```bash
# Une seule IP
--ip 203.0.113.5

# Plage d'IPs
--ip 203.0.113.0-203.0.113.255

# Exemple complet
az storage blob generate-sas \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myblob.txt \
  --permissions r \
  --expiry 2024-12-31T23:59:59Z \
  --ip 203.0.113.5 \
  --https-only \
  --auth-mode login \
  --as-user
```

**5. Signed Protocol (spr) - Forcer HTTPS :**

```bash
# HTTPS uniquement (recommandé)
--https-only

# Ou via paramètre
--protocol https

# PowerShell
-Protocol HttpsOnly
```

### Stored Access Policies

**âš ï¸ Concept Clé : Les Stored Access Policies permettent de contrôler et révoquer des SAS après leur émission**

**Définition :**
- **Stored Access Policy** : Politique d'accès stockée sur le container/share/queue/table
- **Avantage Principal** : Possibilité de modifier ou révoquer des SAS sans régénérer les tokens
- **Limitation** : Uniquement pour Service SAS (pas Account SAS ni User Delegation SAS)

**Pourquoi Utiliser Stored Access Policies ?**

**Sans Stored Access Policy :**
```
âŒ SAS créé avec expiration 2024-12-31
âŒ Impossible de révoquer avant expiration
âŒ Pour révoquer : Rotation de la clé du compte (impact global)
```

**Avec Stored Access Policy :**
```
✅ SAS lié Ã  une policy nommée
✅ Modification de la policy â†’ Impact immédiat sur tous les SAS liés
✅ Suppression de la policy â†’ Révocation de tous les SAS liés
✅ Pas besoin de rotation de clé
```

**Création d'une Stored Access Policy :**

```bash
# Étape 1 : Créer une Stored Access Policy sur un container
az storage container policy create \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name mypolicy \
  --permissions rl \
  --start 2024-01-01T00:00:00Z \
  --expiry 2024-12-31T23:59:59Z

# Étape 2 : Créer un SAS lié Ã  la policy
az storage blob generate-sas \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name myblob.txt \
  --policy-name mypolicy

# Étape 3 : Modifier la policy (impact immédiat sur tous les SAS)
az storage container policy update \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name mypolicy \
  --permissions r \
  --expiry 2024-06-30T23:59:59Z

# Étape 4 : Révoquer tous les SAS liés Ã  la policy
az storage container policy delete \
  --account-name mystorageaccount \
  --container-name mycontainer \
  --name mypolicy
```

**Via PowerShell :**

```powershell
# Obtenir le contexte
$ctx = New-AzStorageContext -StorageAccountName "mystorageaccount" -StorageAccountKey "xxxx"

# Créer Stored Access Policy
$policy = New-AzStorageContainerStoredAccessPolicy `
  -Container "mycontainer" `
  -Policy "mypolicy" `
  -Permission rl `
  -StartTime (Get-Date) `
  -ExpiryTime (Get-Date).AddYears(1) `
  -Context $ctx

# Générer SAS avec policy
$sas = New-AzStorageBlobSASToken `
  -Container "mycontainer" `
  -Blob "myblob.txt" `
  -Policy "mypolicy" `
  -Context $ctx

# Révoquer en supprimant la policy
Remove-AzStorageContainerStoredAccessPolicy `
  -Container "mycontainer" `
  -Policy "mypolicy" `
  -Context $ctx
```

**âš ï¸ Limites des Stored Access Policies :**

| Limite | Valeur | Notes |
|--------|--------|-------|
| **Policies par container** | 5 | Maximum 5 policies nommées |
| **Policies par file share** | 5 | Maximum 5 policies nommées |
| **Policies par queue** | 5 | Maximum 5 policies nommées |
| **Policies par table** | 5 | Maximum 5 policies nommées |
| **Nom de policy** | 64 caractères | Alphanumériques uniquement |

**âš ï¸ Scénarios d'Utilisation - Pour l'Examen :**

**Scénario 1 : Partage Temporaire avec Client Externe**

```
Problème :
- Besoin de partager des fichiers avec un client
- Accès limité Ã  30 jours
- Lecture seule
- Possibilité de révoquer si nécessaire

Solution :
1. Créer User Delegation SAS (plus sécurisé)
2. Permissions : Read (r)
3. Expiry : +30 jours
4. IP restriction si possible
5. HTTPS uniquement

# Commande
az storage blob generate-sas \
  --account-name mystorageaccount \
  --container-name client-files \
  --name report-Q4.pdf \
  --permissions r \
  --expiry "$(date -u -d '+30 days' +%Y-%m-%dT%H:%M:%SZ)" \
  --https-only \
  --auth-mode login \
  --as-user

Avantages :
✅ Pas de clé de compte exposée
✅ Accès granulaire
✅ Expiration automatique
✅ Révocation via Azure AD si besoin
```

**Scénario 2 : Upload de Fichiers par Application Tierce**

```
Problème :
- Application mobile doit uploader des images
- Chaque utilisateur doit pouvoir écrire uniquement
- Pas de lecture des autres fichiers
- Contrôle par IP du datacenter

Solution :
1. Service SAS par utilisateur
2. Permissions : Create, Write (cw)
3. Expiry : Session utilisateur (2 heures)
4. IP restriction : Datacenter
5. Stored Access Policy pour révocation

# Commande
az storage container policy create \
  --account-name mystorageaccount \
  --container-name user-uploads \
  --name upload-policy \
  --permissions cw \
  --expiry "$(date -u -d '+2 hours' +%Y-%m-%dT%H:%M:%SZ)"

az storage blob generate-sas \
  --account-name mystorageaccount \
  --container-name user-uploads \
  --name user123/image.jpg \
  --policy-name upload-policy \
  --ip 203.0.113.0-203.0.113.255

Avantages :
✅ Write-only (sécurité)
✅ Révocation centralisée via policy
✅ Restriction IP
✅ Expiration courte
```

**Scénario 3 : Application Multitenant avec Account SAS**

```
Problème :
- Application SaaS multi-tenant
- Besoin d'accès Blob ET File Storage
- Différents tenants avec différentes permissions
- Audit trail nécessaire

Solution :
1. User Delegation SAS si possible (Blob uniquement)
2. Sinon Account SAS
3. Générer SAS par tenant avec metadata
4. Monitoring Azure Monitor
5. Rotation régulière des clés

# Commande (Account SAS pour Blob + File)
az storage account generate-sas \
  --account-name mystorageaccount \
  --services bf \
  --resource-types sco \
  --permissions rl \
  --expiry "$(date -u -d '+7 days' +%Y-%m-%dT%H:%M:%SZ)" \
  --https-only

Avantages :
✅ Multi-services
✅ Granularité par tenant
✅ Rotation possible
```

**Scénario 4 : CDN avec SAS pour Contenu Privé**

```
Problème :
- CDN doit servir du contenu privé
- Utilisateurs authentifiés uniquement
- Expiration courte des liens

Solution :
1. Générer SAS Ã  la demande côté serveur
2. Permissions : Read (r)
3. Expiry : 15 minutes
4. Passer SAS Ã  CDN
5. CDN utilise SAS pour fetch depuis Storage

# Application backend génère
$sasToken = New-AzStorageBlobSASToken `
  -Container "private-content" `
  -Blob "video-premium.mp4" `
  -Permission r `
  -ExpiryTime (Get-Date).AddMinutes(15) `
  -Context $ctx

# URL finale
https://cdn.example.com/video-premium.mp4?{$sasToken}

Avantages :
✅ Contenu privé sécurisé
✅ Expiration courte
✅ Performance CDN maintenue
```

### Best Practices et Sécurité

**âš ï¸ Best Practices pour SAS :**

**1. Choix du Type de SAS**
- ✅ **Toujours préférer User Delegation SAS** pour Blob Storage
- ✅ Utiliser Service SAS pour ressources spécifiques
- âŒ Éviter Account SAS sauf si multi-services requis

**2. Permissions (Principe du Moindre Privilège)**
- ✅ Donner **uniquement** les permissions nécessaires
- ✅ Préférer **Read-only** si possible
- âŒ Éviter permissions "rwdlacx" (trop large)
- ✅ Utiliser **List** uniquement si nécessaire

```bash
# âŒ Mauvais : Trop de permissions
--permissions rwdlacxtme

# ✅ Bon : Permissions minimales
--permissions r  # Si lecture seule suffit
```

**3. Expiration**
- ✅ Toujours définir une **expiration courte** (heures/jours, pas mois/années)
- ✅ Régénérer SAS si besoin d'accès prolongé
- âš ï¸ **15 minutes** : Sessions temporaires (CDN, uploads)
- âš ï¸ **24 heures** : Accès quotidien
- âš ï¸ **7 jours** : Maximum pour la plupart des cas

```bash
# ✅ Bon : Expiration courte
--expiry "$(date -u -d '+2 hours' +%Y-%m-%dT%H:%M:%SZ)"

# âŒ Mauvais : Expiration trop longue
--expiry "2025-12-31T23:59:59Z"  # Trop loin dans le futur
```

**4. Restrictions Réseau**
- ✅ **HTTPS uniquement** : Toujours utiliser `--https-only`
- ✅ **IP restrictions** : Si IPs connues et fixes
- ✅ **Private Endpoints** : Si accès VNet

```bash
# ✅ Bon : HTTPS + IP restriction
az storage blob generate-sas \
  --permissions r \
  --https-only \
  --ip 203.0.113.5
```

**5. Stored Access Policies**
- ✅ Utiliser pour **contrôle de révocation**
- ✅ Limite de **5 policies** : Planifier les noms
- ✅ **Nommage clair** : upload-policy, read-policy-2024

**6. Monitoring et Audit**
- ✅ Activer **Storage Analytics** pour logs
- ✅ Monitorer **Azure Monitor metrics** pour anomalies
- ✅ Créer **alertes** pour accès suspicieux

```bash
# Activer logging
az storage logging update \
  --account-name mystorageaccount \
  --services b \
  --log rwd \
  --retention 90
```

**7. Rotation des Clés**
- ✅ Rotation régulière (tous les **90 jours**)
- ✅ Utiliser **key2** pendant rotation de **key1**
- ✅ Tester avant basculement complet

**âš ï¸ Erreurs Courantes QCM :**

| Question | Réponse Incorrecte âŒ | Réponse Correcte ✅ |
|----------|----------------------|---------------------|
| **"Quel SAS est le plus sécurisé ?"** | "Account SAS (multi-services)" | "User Delegation SAS (Azure AD)" |
| **"Peut-on révoquer un SAS sans Stored Policy ?"** | "Oui, via Azure Portal" | "Non, seulement en régénérant la clé du compte" |
| **"User Delegation SAS fonctionne pour Files ?"** | "Oui, tous les services" | "Non, Blob et ADLS Gen2 uniquement" |
| **"Combien de Stored Policies par container ?"** | "Illimité" ou "10" | "5 policies maximum" |
| **"SAS sans expiration est possible ?"** | "Oui, pour Account SAS" | "Non, expiration toujours obligatoire" |

**Problèmes Courants et Solutions :**

| Problème | Cause | Solution |
|----------|-------|----------|
| **403 Forbidden avec SAS** | SAS expiré ou permissions insuffisantes | Vérifier expiry time et permissions |
| **SAS ne fonctionne pas** | Clock skew (différence d'horloge) | Utiliser start time -15 minutes |
| **Impossible de révoquer SAS** | Pas de Stored Access Policy | Créer policy ou régénérer clé compte |
| **SAS trop long (URL)** | Trop de paramètres | Minimaliser permissions et paramètres |
| **Erreur signature invalide** | Clé changée ou SAS mal formé | Régénérer SAS avec bonne clé |

**Comparaison avec Autres Méthodes d'Accès :**

| Méthode | Use Case | Avantages | Inconvénients |
|---------|----------|-----------|---------------|
| **Account Key** | Administration | Accès complet | âŒ Trop de privilèges, pas granulaire |
| **User Delegation SAS** | Applications modernes | ✅ Sécurisé, Azure AD | Blob/ADLS uniquement |
| **Service SAS** | Accès délégué | ✅ Granulaire, contrôlé | Nécessite clé compte |
| **Account SAS** | Multi-services legacy | ✅ Multi-services | âŒ Moins sécurisé |
| **Azure AD (RBAC)** | Identités permanentes | ✅ Gestion centralisée | Pas pour accès anonyme |
| **Anonymous Public Access** | Contenu public | ✅ Simple | âŒ Pas sécurisé |

**âš ï¸ Points Clés pour l'Examen :**
- ✅ **3 types de SAS** : User Delegation (recommandé), Service, Account
- ✅ **User Delegation SAS** = Plus sécurisé (Azure AD, pas de clés)
- ✅ **Stored Access Policy** = Permet révocation des SAS
- ✅ **5 policies max** par container/share/queue/table
- ✅ **Expiration obligatoire** pour tous les SAS
- ✅ **HTTPS uniquement** = Best practice
- ✅ **Permissions minimales** = Principe du moindre privilège
- ✅ **sp=r** : Read, **sp=w** : Write, **sp=d** : Delete, **sp=l** : List
- ✅ **Révocation** : Supprimer Stored Policy OU régénérer clé compte
- ✅ **IP restriction** possible avec paramètre `--ip`