# AZ-104 Microsoft Azure Administrator - Guide Complet d'Examen

## 📋 Table des Matières
1. [Manage Azure Identities and Governance (15-20%)](#1-manage-azure-identities-and-governance)
2. [Implement and Manage Storage (15-20%)](#2-implement-and-manage-storage)
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

**🎯 Erreur fréquente identifiée :** Syntaxe des règles dynamiques
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

**🎯 Point d'attention identifié :** Types d'enregistrements DNS acceptés
- **TXT** : Méthode recommandée (plus flexible)
- **MX** : Alternative acceptable
- Exemple : `MS=ms12345678` dans un enregistrement TXT

#### Licensing et Dynamic Groups

**🎯 Processus d'assignation automatique de licences :**
1. **Créer un groupe de sécurité dynamique** basé sur des attributs personnalisés
2. **Configurer les règles** du groupe dynamique
3. **Ajouter le groupe à un groupe de licences** pour synchronisation automatique
4. **Tous les utilisateurs** du groupe reçoivent automatiquement la licence

**🎯 Points clés identifiés :**
- **Dynamic security groups** : Obligatoires pour assignation automatique
- **Custom attributes** : Base des règles de groupe
- **License groups** : Synchronisation automatique requise
- **Automatic assignment policies** : Non utilisées pour les licences

#### B2B Collaboration

**🎯 Configuration des paramètres de collaboration externe :**
- **External collaboration settings** : Contrôlent qui peut inviter des utilisateurs externes
- **Domain restrictions** : Autoriser/bloquer des domaines spécifiques
- **Guest user visibility** : Contrôler ce que voient les invités dans l'annuaire
- **Conditional Access** : Renforcer l'authentification et bloquer l'accès depuis des emplacements inconnus
- **Cross-tenant access** : Configuration de collaboration avec des organisations Microsoft Entra spécifiques

**🎯 Format UPN des utilisateurs invités :**
- **Guest users** : `bsmith_contoso.com#EXT#@fabrikam.com`
- **Regular users** : `user@fabrikam.com`
- **Access reviews** : Non utilisées pour contrôler les invitations d'invités

**🎯 Prérequis pour assignation de licences :**
- **Usage location** : Obligatoire avant assignation de licence
- **Not all Microsoft 365 services** disponibles dans tous les emplacements
- **First name, Last name, Other email, User type** : Non obligatoires pour assignation de licence

### 1.2 Role-Based Access Control (RBAC)

#### Rôles Built-in Essentiels
- **Owner** : Accès complet + gestion des accès
- **Contributor** : Accès complet sauf gestion des accès
- **Reader** : Lecture seule
- **User Access Administrator** : Gestion des accès uniquement

#### Rôles Spécialisés
- **Virtual Machine Contributor** : Gestion des VMs
- **Storage Account Contributor** : Gestion des comptes de stockage
- **Network Contributor** : Gestion des ressources réseau

**🎯 Différenciation des rôles essentiels :**

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

#### Scopes d'assignation
1. **Management Group** : Niveau le plus élevé
2. **Subscription** : Toutes les ressources de la souscription
3. **Resource Group** : Toutes les ressources du groupe
4. **Resource** : Ressource spécifique

**🎯 Erreur identifiée :** Root Management Group
- **Aucun accès par défaut** au root management group
- Seuls les **Global Administrators** peuvent s'élever
- Process : Global Admin → "Access management for Azure resources" → Assign roles

### 1.3 Azure Policy

#### Concepts Clés
- **Policy Definition** : Règle de conformité
- **Policy Assignment** : Application d'une policy à un scope
- **Initiative** : Collection de policies
- **Compliance** : État de conformité des ressources

#### Effects Principaux
- **Deny** : Bloque la création/modification
- **Audit** : Marque comme non-conforme (pas de blocage)
- **Append** : Ajoute des propriétés
- **DeployIfNotExists** : Déploie des ressources si conditions

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

**🎯 Types de verrous et limitations :**

**Delete Locks**
- **Protection** : Bloque la suppression de ressources
- **Ressources supportées** : Virtual machines, subscriptions, resource groups
- **Ressources non supportées** : Management groups, storage account data
- **Usage** : Protection contre suppression accidentelle

**🎯 Points clés identifiés :**
- **Delete locks** : Empêchent la suppression mais pas la modification
- **Management groups** : Ne peuvent pas être verrouillés
- **Storage account data** : Données non protégées par les locks
- **Scope** : Applicable aux VMs, subscriptions, et resource groups uniquement

---

## 2. Implement and Manage Storage (15-20%)

### 2.1 Storage Accounts

#### Types de Storage Accounts

**General Purpose v2 (GPv2) - Standard**
- **Services** : Blobs, Files, Queues, Tables
- **Performance** : Standard (HDD)
- **Réplication** : Toutes les options (LRS, ZRS, GRS, GZRS, RA-GRS, RA-GZRS)
- **Usage** : Polyvalent, recommandé pour la plupart des cas

**Premium Block Blobs**
- **Services** : Blobs uniquement
- **Performance** : Premium (SSD)
- **Réplication** : LRS, ZRS uniquement
- **Usage** : Applications haute performance

**Premium File Shares**
- **Services** : Files uniquement
- **Performance** : Premium (SSD)
- **Réplication** : LRS, ZRS uniquement
- **Usage** : Partages de fichiers haute performance

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

#### Niveaux d'accès (Blob Storage)
- **Hot** : Accès fréquent, coût stockage élevé, coût accès faible
- **Cool** : Accès occasionnel (30 jours minimum), coût moyen
- **Archive** : Accès rare (180 jours minimum), coût très faible, latence haute

#### Réplication et Durabilité

**🎯 Points d'attention identifiés :**

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

### 2.2 Azure Files

#### Protocoles Supportés
- **SMB 3.0/3.1** : Windows, Linux, macOS
- **NFS 4.1** : Linux, Premium uniquement
- **REST API** : Accès programmatique

**🎯 Point clé identifié :** Port SMB
- **Port 445 TCP** obligatoire pour accès SMB
- Doit être ouvert sur les firewalls clients
- Nécessaire pour mapper des lecteurs réseau

#### Types de File Shares
- **Standard** : StorageV2 accounts, performance modérée
- **Premium** : FileStorage accounts, haute performance (SSD)

#### Capacités et Limites
- **Standard** : Maximum 5 TB par share
- **Premium** : Maximum 100 TB par share
- **Azure Import/Export** : Support Blob Storage et Azure Files

### 2.3 Blob Storage

#### Types de Blobs

** Comprendre les 3 types de blobs Azure - Points critiques pour l'examen :**
                                         
**1. Block Blobs**
- **Usage principal** : Stockage de fichiers standard (documents, images, vidéos, archives)
- **Structure** : Composés de blocs individuels (jusqu'à 50,000 blocs par blob)
- **Taille maximale** : 190.7 TB (4.75 TB × 50,000 blocs)
- **Taille de bloc** : Jusqu'à 4000 MB par bloc
- **Optimisation** : Idéal pour streaming et accès aléatoire
- **Cas d'usage typiques** :
  - Sites web statiques (HTML, CSS, JS, images)
  - Stockage de documents et médias
  - Sauvegardes et archives
  - Distribution de contenu (CDN)

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

### 2.4 Data Transfer Solutions

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

#### Other Transfer Options
- **AzCopy** : Outil ligne de commande
- **Azure Storage Explorer** : Interface graphique
- **Data Box** : Appliances physiques (TB vers PB)

#### Storage Account Roles et Permissions

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

#### High Availability

**Availability Sets**
- **Fault Domains** : Protection pannes matérielles (max 3)
- **Update Domains** : Protection maintenances (max 20)
- **SLA** : 99.95%
- **Scope** : Même datacenter

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

#### Service Plans
- **Shared** : F1 (Free), D1 (Shared) - Limitations CPU
- **Basic** : B1, B2, B3 - Applications simples
- **Standard** : S1, S2, S3 - Applications production
- **Premium** : P1, P2, P3 - Applications critiques
- **Isolated** : I1, I2, I3 - Environnements dédiés

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
- **Address Space** : Plage CIDR privée (10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16)
- **Subnets** : Subdivision du VNet
- **Network Security Groups** : Firewalls au niveau subnet/NIC

#### VNet Peering

**Types de Peering :**
- **Regional** : VNets dans la même région
- **Global** : VNets dans différentes régions
- **Traffic** : Privé, pas d'Internet, faible latence
- **Billing** : Facturation du trafic cross-region

**Tips critiques identifiés :**

**1. Règle d'Or : Plages d'adresses non-chevauchantes**
- **Principe** : Deux VNets ne peuvent être peerés que si leurs plages d'adresses ne se chevauchent pas
- **Exemple critique** : VNet1 (192.168.0.0/24) ne peut PAS être peeré avec VNet3 (192.168.0.0/16)
- **Raison** : /24 est inclus dans /16 → chevauchement détecté
- **Solution** : Utiliser des plages complètement différentes (ex: 10.0.0.0/16 vs 172.16.0.0/16)

**2. Performance et Latence**
- **Avantage clé** : Communication avec la même latence et bande passante que si les ressources étaient sur le même VNet
- **Trafic privé** : Pas de transit par Internet, sécurité renforcée
- **Optimisation** : Idéal pour architectures distribuées (prod/dev, multi-régions)

**3. Configuration dans le Portail Azure**
- **Navigation** : VNet → Peerings → Add peering
- **Bidirectionnel** : Créer le peering dans les deux sens
- **Validation** : Azure vérifie automatiquement la compatibilité des plages

#### DNS Resolution
** Point identifié :** DNS interne Azure
- **Format automatique** : `vm-name.internal.cloudapp.net`
- **Usage** : Résolution entre VMs dans VNet
- **Custom DNS** : Possibilité d'utiliser ses propres serveurs

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

#### Azure Load Balancer (Layer 4)
- **Internal** : Trafic interne au VNet
- **Public** : Trafic depuis Internet
- **Features** : Health probes, NAT rules, HA ports

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

#### Application Gateway (Layer 7)
- **WAF** : Web Application Firewall
- **SSL termination** : Gestion certificates
- **URL routing** : Routage basé sur l'URL
- **Multi-site hosting** : Plusieurs sites web

#### Traffic Manager (DNS-based)
- **Global** : Répartition géographique
- **Methods** : Performance, Geographic, Weighted, Priority
- **Health monitoring** : Surveillance des endpoints

### 4.4 Network Watcher

** Points identifiés pour l'examen :**

#### Connection Monitor
- **Usage** : Mesurer RTT entre VMs
- **Granularity** : Métriques par minute
- **Targets** : VM, FQDN, URI, IPv4
- **Protocols** : TCP direct

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

---

## 5. Monitor and Backup Azure Resources (10-15%)

### 5.1 Azure Monitor

#### Architecture
- **Data Collection** : Métriques, logs, traces
- **Storage** : Metrics store, Log Analytics workspace
- **Analysis** : KQL queries, workbooks, dashboards
- **Actions** : Alerts, autoscale, automation

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

---

## Checklist Final d'Examen

### Identities and Governance
- [ ] Dynamic group rules syntax : `(user.property -eq "value")`
- [ ] Custom domain DNS records : TXT ou MX
- [ ] Root MG access : Global Admin + elevation
- [ ] RBAC scopes : MG → Subscription → RG → Resource

### Storage
- [ ] FileStorage pour Premium files uniquement
- [ ] Import/Export destinations : Blob + Files (5TB max)
- [ ] Replication types par account type
- [ ] Port 445 pour Azure Files SMB

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

### Monitoring & Backup
- [ ] Log Analytics Workspace comme target pour VM alerts
- [ ] Backup policies : 100 VMs max par policy
- [ ] Recovery Services Vault : même région
- [ ] Stop backup avant delete vault
- [ ] KQL syntax : `Table | search "term"`

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
